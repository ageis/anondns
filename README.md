# AnonDNS: encrypting and anonymizing all your DNS requests with Tor (WORK IN PROGRESS)

This project's goal is to anonymize and encrypt DNS requests. It currently consists of automation code to set up dnsmasq (either locally or public-facing) in combination with Tor which will anonymize all of your outgoing DNS requests. **This is not an official Tor project.**

In recent years, the world wide web has been making significant and impressive strides in HTTPS adoption. As part of this, Google's Chrome web browser will begin marking plain HTTP sites "insecure" in the user interface later this year. Likewise, Mozilla plans to require secure contexts for most features. The statistics since the advent of Let's Encrypt have been impressive.

![Let's Encrypt statistics][2]
<sub><sup>
Source: [Let's Encrypt][13]
</sub></sup>

Still, one place where privacy has been significantly lacking is in DNS. Each time one makes a connection to a website, the client must first translate the hostname to an IPv4/6 address. It's an old protocol, first standardized by the IETF in 1983 and [BIND][16] came out a year later. SSL would not happen for another decade, which is to say that the Domain Name System was not not designed with security in mind. There's no encryption of DNS requests, although the results can be signed with [DNSSEC][18] to block tampering or poisoning. This subject has been explored previously, as in IETF papers titled [Pretty Bad Privacy: Pitfalls of DNS Encryption][14] and [Encrypted DNS: An opportunistic encryption protocol extension for DNS][15]. Google, who operates the most popular public open resolvers, provides information about [DNS-over-HTTPS][17]. There's the recent [RFC 8094][3], a Specification for DNS over Datagram Transport Layer Security (DTLS), as well as RFCs [7626][19] and [7858][20].

The goal of the project is to anonymize and encrypt one's DNS requests on their machine by leveraging [Tor](https://www.torproject.org) and its [DNSPort](https://www.torproject.org/docs/tor-manual.html.en#DNSPort) feature, absolutely prohibiting leaks of requests to your ISP or other network adversaries with [iptables](http://www.netfilter.org/projects/iptables/index.html), plus a locally-running DNS server such as [dnsmasq](http://www.thekelleys.org.uk/dnsmasq/doc.html) to provide caching (avoiding slowness) and DNSSEC validation.

It's presently geared to users of Debian GNU/Linux 9 (stretch). Currently this is just a set of Ansible roles and playbooks, but the long-term plan is a cross-platform program that runs as service and has an accompanying graphical configuration tool.

In this alpha release, it can be configured to run locally or as a public Tor-based DNS server. Every time you make a DNS request, you could be getting your results from a random Tor exit node anywhere. Poisoning or spoofing is a risk; so the default configuration includes DNSSEC validation, and also prevents rebinding attacks and addresses from private IP space to be returned. Any process or user who is not dnsmasq will be redirected through Tor.

Please consult [NOTES_AND_TODO.md](NOTES_AND_TODO.md) for what needs to be imeplemented and explored next.

Here's an overview of the playbooks:

* local.yml: Run ansible-playbook --connection=local --limit localhost in order to activate AnonDNS on your machine.
* security.yml: Provides OS and SSH hardening of the target machine. Requires you to run `ansible-galaxy install --force --ignore-errors -r requirements.yml` in order to obtain the security-related dependencies from [dev-sec.io](http://dev-sec.io)
* tor-dns-server.yml: Sets up a full-fleged public Tor-based DNS server, with monitoring (explained below) and extra configuration provided by the "common" role.
* monitoring.yml: Provides a monitoring framework for both Tor and dnsmasq, based on [Prometheus](https://prometheus.io) and [Grafana](https://grafana.com), with which you can collect metrics and build visualizations. You should have previously setup a valid DNS name or A record for the server in `hosts`. A [Let's Encrypt](https://letsencrypt.org) certificate will be generated.

## Variables

```yaml
# If set to true this will open port 53 to the public internet via ufw, and bind your ethernet interface instead of loopback..
public_dns_server: false

# If set to true, dnsmasq will fall back in strict order to the listed nameservers, when Tor fails. This is recommended.
nameserver_fallbacks: true

# A list in relation to the above setting.
fallback_nameservers:
  - "9.9.9.9" # Quad9
  - "8.8.8.8" # Google
  - "208.67.222.222" # OpenDNS
dnssec_enabled: true
dnssec_check_unsigned: false

# Replace your system-wide Tor configuration with our own. Otherwise, we'll only set the options needed to make AnonDNS work.
replace_torrc: true

# The name of your regular unprivileged system user.
anondns_user: ''
```

## Getting started

The configuration management is done with [Ansible](https://www.ansible.com/), which may be obtained via Python's [pip](https://bootstrap.pypa.io/get-pip.py).

We use Debian GNU/Linux (currently stretch or 9.x) for everything. Don't try running this code on another distribution and expect it to work.

All commands documented here are expected to be executed from root of this repository. Here's what one needs to have installed:

```bash
sudo pip install -U ansible
```

We recommend Ansible 2.4.3 or later.

In order to update the security-related dependencies for hardening base operating system and SSH daemon:

Edit the hosts inventory at `hosts` and set the IP address for your server. It's ideal to have some DNS subdomains pointing to that same server.

Then you may execute the playbook at `tor-dns-server.yml`.

# HTTPS adoption succeeds while most of the Domain Name System remain plaintext

![Chrome HTTPS statistics][4]
<sub><sup>
Source: [https://research.google.com/pubs/pub46197.html][5]
</sub></sup>

![HTTPS statistics][6]
<sub><sup>
Source: [https://research.google.com/pubs/pub46197.html][5]
</sub></sup>

![Alexa statistics][7]
<sub><sup>
Source: [https://research.google.com/pubs/pub46197.html][5]
</sub></sup>

![HTTPS support][8]
<sub><sup>
Source: [https://research.google.com/pubs/pub46197.html][5]
</sub></sup>

![HTTPS support][9]
<sub><sup>
Source: [https://research.google.com/pubs/pub46197.html][5]
</sub></sup>

![Chrome statistics][4]
<sub><sup>
Source: [https://research.google.com/pubs/pub46197.html][5]

![Firefox HTTPS page loads][10]
<sub><sup>
Source: [https://research.google.com/pubs/pub46197.html][5]
</sub></sup>

![HTTPS adoption in Alexa's top million websites][11]
<sub><sup>
Source: [https://scotthelme.co.uk/alexa-top-1-million-analysis-aug-2017/][12]
</sub></sup>

[1]: https://img.cointel.pro/firefox_telemetry.png
[2]: https://img.cointel.pro/letsencrypt_stats.png
[3]: https://tools.ietf.org/html/rfc8094
[4]: https://img.cointel.pro/chrome_stats.png
[5]: https://research.google.com/pubs/pub46197.html
[6]: https://img.cointel.pro/https_stats.png
[7]: https://img.cointel.pro/alexa_stats.png
[8]: https://img.cointel.pro/https_support.png
[9]: https://img.cointel.pro/https_support_2.png
[10]: https://img.cointel.pro/firefox_page_loads.png
[11]: https://img.cointel.pro/https_adoption.png
[12]: https://scotthelme.co.uk/alexa-top-1-million-analysis-aug-2017/
[13]: https://letsencrypt.org/stats/
[14]: https://www.ietf.org/mail-archive/web/dns-privacy/current/pdfWqAIUmEl47.pdf
[15]: https://www.ietf.org/mail-archive/web/dns-privacy/current/pdfWjIXeAM9so.pdf
[16]: https://www.isc.org/downloads/bind/
[17]: https://developers.google.com/speed/public-dns/docs/dns-over-https
[18]: http://www.dnssec.net
[19]: https://tools.ietf.org/html/rfc7626
[20]: https://tools.ietf.org/html/rfc7858