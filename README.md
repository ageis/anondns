# AnonDNS: encrypting and anonymizing all your DNS requests with Tor (WORK IN PROGRESS)

This project's goal is to anonymize and encrypt DNS requests.  Automation code to set up dnsmasq (either locally or public-facing) which will anonymize all of your outgoing DNS requests.

In recent years, the world wide web has been making significant and impressive strides in HTTPS adoption. As part of this, Google's Chrome web browser will begin marking plain HTTP sites "insecure" in the user interface later this year. Likewise, Mozilla plans to require secure contexts for most features. The statistics since the advent of Let's Encrypt have been impressive.

<center>
![][2]

<sub><sup>Source: [Let's Encrypt][13]</sub></sup>
</center>

Still, one place where privacy has been significantly lacking is in DNS. Each time one makes a connection to a website, the client must first translate the hostname to an IPv4/6 address. It's an old protocol, first standardized by the IETF in 1983 and BIND came out a year later. SSL would not happen for another decade, which so to say that the Domain Name System was not not designed with security in mind. There's no encryption of DNS requests, although the results can be signed with DNSSEC to block tampering or poisoning. The goal of the project is to anonymize and encrypt one's DNS requests on their machine by leveraging [Tor](https://www.torproject.org) and its DNSPort feature, absolutely prohibiting leaks of requests to your ISP or other network adversaries, plus a locally-running DNS server such as dnsmasq to provide caching (avoiding slowness) and DNSSEC validation.

It's presently geared to users of Debian GNU/Linux 9 (stretch). Currently this is just a set of Ansible roles and playbooks, but the long-term plan is a cross-platform program that runs as service and has an accompanying graphical configuration tool.

In this alpha release, it can be configured to run locally or as a public Tor-based DNS server. Every time you make a DNS request, you could be getting your results from a random Tor exit node anywhere. Poisoning or spoofing is a risk; so the default configuration includes DNSSEC validation, and also prevents rebinding attacks and addresses from private IP space to be returned. Any process or user who is not dnsmasq will be redirected through Tor.

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
```

Getting started
---------------

The configuration management is done with [Ansible](https://www.ansible.com/), which may be obtained via Python's [pip](https://bootstrap.pypa.io/get-pip.py).

We use Debian GNU/Linux (currently stretch or 9.x) for everything. Don't try running this code on another distribution and expect it to work.

All commands documented here are expected to be executed from root of this repository. Here's what one needs to have installed:

```
sudo pip install -U ansible
```

We recommend Ansible 2.4.3 or later.

In order to update the security-related dependencies for hardening base operating system and SSH daemon:


Edit the hosts inventory at `hosts` and set the IP address for your server. It's ideal to have some DNS subdomains pointing to that same server.

Then you may execute the playbook at `tor-dns-server.yml`.

<center>
![][4]

Source: [https://research.google.com/pubs/pub46197.html][5]
</center>

<center>
![][6]

Source: [https://research.google.com/pubs/pub46197.html][5]
</center>

<center>
![][7]

Source: [https://research.google.com/pubs/pub46197.html][5]
</center>

<center>
![][8]

Source: [https://research.google.com/pubs/pub46197.html][5]
</center>

<center>
![][9]

Source: [https://research.google.com/pubs/pub46197.html][5]
</center>

<center>
![][4]

Source: [https://research.google.com/pubs/pub46197.html][5]
</center>

<center>
![][10]

Source: [https://research.google.com/pubs/pub46197.html][5]
</center>

<center>
![][11]

Source: [https://scotthelme.co.uk/alexa-top-1-million-analysis-aug-2017/][12]
</center>

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
