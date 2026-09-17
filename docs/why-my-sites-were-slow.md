# Why my sites were slow (and what fixed it)

## The symptom

For the past few days, clicking a link to blog.jonminton.net, stats.jonminton.net, stats-board.jonminton.net, or the hub at jonminton.net, in Chrome on my MacBook at home, could take 10 to 30 or more seconds to even start loading. Once a page did start, it appeared almost instantly, as normal. The delay was not consistent, which made the problem feel random and hard to pin down.

## Key terms

Before explaining what was happening, here are the pieces involved.

**IPv4 and IPv6** are two different addressing systems the internet uses to identify computers and servers. IPv4 addresses look like `185.199.108.153`; IPv6 is a newer, larger address space, with addresses that look like `2606:50c0::153`. Most modern devices support both, and when a website offers both, the browser chooses which to use.

**DNS** (Domain Name System) is the internet's phone book: it translates a name like `blog.jonminton.net` into the numeric address a browser needs to connect. An **A record** maps a name to an IPv4 address, and a **AAAA record** maps a name to an IPv6 address. A **CNAME record** is different: instead of giving an address directly, it says "this name is really just another name, go look up that one instead," so it inherits whatever A and AAAA records the target name has.

**GitHub Pages** and **Netlify** are the two free hosting services I use. Netlify hosts jonminton.net itself; GitHub Pages hosts the blog, stats, stats-board, food, and games subdomains, and publishes both IPv4 addresses (185.199.108.153 to 185.199.111.153) and IPv6 addresses (in the 2606:50c0:: range).

A **TCP connection** is the underlying pipe a browser opens to a server before it can ask for anything, and it takes a small amount of time to set up. **Keep-alive**, or connection reuse, means the browser keeps that pipe open after a page loads, in case it is needed again soon, rather than reopening it for every request. **HTTP/2 connection pooling** is Chrome's habit of keeping a small stock of these open connections to recently visited sites, ready to reuse: it is why clicking around a site you have just visited feels instant.

**Chrome's hung-connection timer** is a safety mechanism: if Chrome tries to reuse a pooled connection and gets no response, it waits 10 seconds, then gives up, discards the connection, and starts over with a fresh DNS lookup and connection.

Finally, a **route** through the internet is the chain of networks data actually travels through, rather than a straight line. IPv4 and IPv6 traffic can take different routes to the same destination, since each address system is routed separately and providers may have different arrangements with other networks for each.

## What was actually going wrong, step by step

Here is the story of one slow click. I open blog.jonminton.net for the first time in a while. Chrome has a pooled connection left over from an earlier visit, and my home internet, provided by BT, has given my laptop both an IPv4 and an IPv6 address. Chrome prefers IPv6 when both are available, so that pooled connection was made over IPv6.

The problem is that IPv6 connections to GitHub Pages, specifically, were being silently killed by something on the network path about 20 seconds after they opened, whether or not they were being used. Chrome had no way to know the connection was dead, so when I click the link, it tries to reuse the connection, sends a request into the pipe, and hears nothing back.

Chrome waits, because that is what the hung-connection timer is for. After 10 seconds it gives up, discards the dead connection, does a fresh DNS lookup, and opens a brand new connection. That fresh connection works fine, the actual page request over it took only about 130 milliseconds, so the page then loads normally. The visible result was a 10 second stall before anything appeared to happen, sometimes longer if several pooled connections were dead and had to be discarded in sequence, which is how the delay stretched to 30 seconds or more.

The problem looked random because GitHub Pages tells browsers to cache its pages for 10 minutes. A page visited in the last 10 minutes was served straight from Chrome's local cache, no network involved, so it felt instant. Only pages not recently cached triggered the stall.

## How we ruled out the other suspects

A batch of tests eliminated the more obvious culprits. Fetching each site from the command line, bypassing Chrome, took under 1.1 seconds every time, so the servers were fast. A background request over an already-warm connection took under 250 milliseconds, so a warm connection was never the issue. Site content was cleared too: no script is shared across all four affected sites, and their fonts and CDNs tested fine. The home router's DNS answered instantly, there was no system proxy or active VPN, and Chrome extensions and Safe Browsing were cleared because an unrelated site loaded instantly in the same browser. Even Chrome's own cache was tested and cleared.

A more targeted test made a secure connection to GitHub Pages, sent a request, waited, then sent a second request on the same connection. Over IPv4 this worked reliably even after waiting two minutes. Over IPv6 it worked after short waits, but reliably hung for 18 seconds and died after waits of 20 seconds or more, even when the connection had been sending a request every 10 seconds throughout. That rules out plain idleness: it is the connection's age, not inactivity, that kills it. The same test against Google, Netlify, Cloudflare, jsDelivr, cdnjs, and PyPI all worked fine over IPv6, so this was specific to GitHub Pages. Plain pings over IPv6 to GitHub Pages showed no packet loss and a steady 42 millisecond response, so basic reachability was fine. The difference is the route: IPv6 traffic travels from BT via a transit carrier called Cogent, while IPv4 traffic stays inside BT's own network at around 20 milliseconds. Something stateful on that IPv6 route is quietly killing established connections at around the 20 second mark.

## The two fixes

**Fix 1, laptop only:** turn off IPv6 on the Mac's Wi-Fi with `sudo networksetup -setv6off Wi-Fi` (reversible with `sudo networksetup -setv6automatic Wi-Fi`). With no IPv6 address available, Chrome has no choice but to connect over IPv4, and IPv4 connections to GitHub Pages do not die. The trade-off: this only fixes my own laptop on my own network, anyone else on a similar broken route is still affected, and the laptop loses IPv6 for everything, though that costs nothing in practice today.

**Fix 2, site-wide:** in Netlify's DNS settings, replace the five CNAME records that point blog, stats, stats-board, food, and games at jonminton.github.io with explicit A records pointing at the four GitHub Pages IPv4 addresses, and publish no AAAA record. A CNAME inherits whatever addresses its target publishes, IPv6 included, but an A-only record set declares these sites IPv4-only to every browser, so nobody's browser can pick the broken IPv6 path. GitHub Pages identifies sites by hostname rather than IP address, so existing certificates keep working unchanged. The trade-off: these sites lose IPv6 for every visitor, harmless since everyone also has IPv4; if GitHub ever changes its published addresses the records need manual updating, where a CNAME would follow automatically; and since it is a live change to public DNS, it is worth applying deliberately with the old records noted for rollback.

Neither fix changes any site content. The fault sits on the IPv6 route between BT and GitHub and may clear up on its own, but both fixes make the sites immune to it regardless.

## What to do now

For a quick, personal fix, turn off IPv6 on the laptop's Wi-Fi with the command above. For a permanent, visitor-facing fix, switch the five GitHub Pages subdomains from CNAME to A records in Netlify DNS, and keep a note of the old CNAME target in case a rollback is ever needed.
