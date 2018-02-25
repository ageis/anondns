# AnonDNS: encrypting and anonymizing all your DNS requests with Tor (WORK IN PROGRESS)

This project's goal is to anonymize and encrypt DNS requests.  Automation code to set up dnsmasq (either locally or public-facing) which will anonymize all of your outgoing DNS requests.

In recent years, the world wide web has been making significant and impressive strides in HTTPS adoption. As part of this, Google's Chrome web browser will begin marking plain HTTP sites "insecure" in the user interface later this year. Likewise, Mozilla plans to require secure contexts for most features. The statistics since the advent of Let's Encrypt have been impressive.

![][1 Source: Firefox Telemetry via [https://letsencrypt.org/stats/][2]

Still, one place where privacy has been significantly lacking is in DNS. Each time one makes a connection to a website, the client must first translate the hostname to an IPv4/6 address. It's an old protocol, first standardized by the IETF in 1983 and BIND came out a year later. SSL would not happen for another decade, which so to say that the Domain Name System was not not designed with security in mind. There's no encryption of DNS requests, although the results can be signed with DNSSEC to block tampering or poisoning. The goal of the project is to anonymize and encrypt one's DNS requests on their machine by leveraging [Tor](https://www.torproject.org] and its DNSPort feature, absolutely prohibiting leaks of requests to your ISP or other network adversaries, plus a locally-running DNS server such as dnsmasq to provide caching (avoiding slowness) and DNSSEC validation.

It's presently geared to users of Debian GNU/Linux 9 (stretch). Currently this is just a set of Ansible roles and playbooks, but the long-term plan is a cross-platform program that runs as service and has an accompanying graphical configuration tool.

In this alpha release, it can be configured to run locally or as a public Tor-based DNS server. Every time you make a DNS request, you could be getting your results from a random Tor exit node anywhere. Poisoning or spoofing is a risk; so the default configuration includes DNSSEC validation, and also prevents rebinding attacks and addresses from private IP space to be returned. Any process or user who is not dnsmasq will be redirected through Tor.

Here's an overview of the playbooks:

* local.yml: Run ansible-playbook --connection=local --limit localhost in order to activate AnonDNS on your machine.
* security.yml: Provides OS and SSH hardening of the target machine. Requires you to run `ansible-galaxy install --force --ignore-errors -r requirements.yml` in order to obtain the security-related dependencies from [dev-sec.io](http://dev-sec.io]
* tor-dns-server.yml: Sets up a full-fleged public Tor-based DNS server, with monitoring (explained below) and extra configuration provided by the "common" role.
* monitoring.yml: Provides a monitoring framework for both Tor and dnsmasq, based on Prometheus and Grafana, with which you can collect metrics and build visualizations. You should have previously setup a valid DNS name or A record for the server in `hosts`. A [Let's Encrypt](https://letsencrypt.org) certificate will be generated.

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



![][4]

Source: [https://research.google.com/pubs/pub46197.html][5]

![][6]

Source: [https://research.google.com/pubs/pub46197.html][5]

![][7]

Source: [https://research.google.com/pubs/pub46197.html][5]

![][8]

Source: [https://research.google.com/pubs/pub46197.html][5]

![][9]

Source: [https://research.google.com/pubs/pub46197.html][5]

![][4]

Source: [https://research.google.com/pubs/pub46197.html][5]

![][10]

Source: [https://research.google.com/pubs/pub46197.html][5]

![][11]

Source: [https://scotthelme.co.uk/alexa-top-1-million-analysis-aug-2017/][12]

[1]: https://cdn-images-1.medium.com/max/1600/1*4Ks0_NtnfjyY8N-FZyYZbg.png
[2]: https://medium.com/r/?url=https%3A%2F%2Fletsencrypt.org%2Fstats%2F
[3]: https://medium.com/r/?url=https%3A%2F%2Ftools.ietf.org%2Fhtml%2Frfc8094
[4]: https://cdn-images-1.medium.com/max/1600/1*DZCItCe-KvAU3N7m_aPLhw.png
[5]: https://medium.com/r/?url=https%3A%2F%2Fresearch.google.com%2Fpubs%2Fpub46197.html
[6]: https://cdn-images-1.medium.com/max/1600/1*fzPhekhQ4ZvnQMQJsiQrog.png
[7]: https://cdn-images-1.medium.com/max/1600/1*ai9k7A9oJhmWEhzYNqDJ1Q.png
[8]: https://cdn-images-1.medium.com/max/1600/1*CDkiZVQ2xef5XnWF5yB_Ow.png
[9]: https://cdn-images-1.medium.com/max/1600/1*CLqbuJkkC2K9XVNvnZElSg.png
[10]: https://cdn-images-1.medium.com/max/1600/1*cXgB3Rowq8lBw3snhqvhGQ.png
[11]: https://cdn-images-1.medium.com/max/1600/1*J9yrgW4CCh3iOT8Kh9pSZA.png
[12]: https://medium.com/r/?url=https%3A%2F%2Fscotthelme.co.uk%2Falexa-top-1-million-analysis-aug-2017%2F

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