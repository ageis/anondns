Notes and todo
---

Make the UI and choose UI framework.

Options configurable for public or local DNS server, extra hosts files to include, and fallback servers.

The UI might just be a wrapper around Ansible. I'd also probably prefer not to develop my own caching/DNSSEC-validating DNS server in Python, although DNSPython and PyPi packages like minidns and dnslib exist.

Build solution for OSX and brew, either native firewall or Little Snitch.

Build solution for Windows, using [Acrylic](http://mayakron.altervista.org/wikibase/show.php?id=AcrylicHome) or cross-compiling dnsmasq with mingw32/64. Work with the registry and Windows Firewall or PowerShell to alter DNS nameservers.

Clone OnionShare's usage of nss_installer and embedded Tor service, register it as a service.

Examine potential of [Unbound](https://www.unbound.net) instead... or [Knot](https://github.com/CZ-NIC/knot-resolver)

Look for more interfering services that could change nameserver away from 127.0.0.1. Check resolvonf for the symlink check, disable that (done).

Look into [DNSCrypt](https://dnscrypt.info) plus [dnscrypt-proxy](https://github.com/jedisct1/dnscrypt-proxy) to see if it's at all relevant.

Decide whether wueries to upstream fallback servers should still be proxied through Tor.

Other random links:

- http://mayakron.altervista.org/wikibase/show.php?id=AcrylicFAQ
- https://thesprawl.org/projects/dnschef/
- https://wiki.archlinux.org/index.php/Tor#Using_TorDNS_for_all_DNS_queries 
- https://tor.stackexchange.com/questions/7212/how-do-i-resolve-dns-using-tor/14594#14594
- https://thesprawl.org/projects/dnschef/
