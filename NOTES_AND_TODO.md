Notes and todo
---

Make the UI, choose UI framework, options configurable for public or local DNS server, extra hosts files to include, and fallback servers.
The UI might be a wrapper around Ansible. I'd also probably prefer not to develop my own caching/DNSSEC-validating  DNS server in Pythohn.
Build solution for OSX and brew, either native firewall or Little Snitch.
Build solution for Windows, using Acrylic or building dnsmasq with mingw32/64.
Clone OnionShare's usage of nss_installer and embedded Tor service, register it as a service.

Examine potential of Unbound instead.
http://mayakron.altervista.org/wikibase/show.php?id=AcrylicFAQ
check resolvonf for symlink check
https://thesprawl.org/projects/dnschef/
https://wiki.archlinux.org/index.php/Tor#Using_TorDNS_for_all_DNS_queries 
https://tor.stackexchange.com/questions/7212/how-do-i-resolve-dns-using-tor/14594#14594
Look into [DNSCrypt](https://wiki.archlinux.org/index.php/DNSCrypt) 