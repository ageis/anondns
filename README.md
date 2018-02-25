# AnonDNS: encrypting and anonymizing all your DNS requests with Tor

This 
Automation code to set up dnsmasq (either locally or public-facing) which will anonymize all of your outgoing DNS requests.

In recent years, the world wide web has been making significant and impressive strides in HTTPS adoption. As part of this, Google's Chrome web browser will begin marking plain HTTP sites "insecure" in the user interface later this year. Likewise, Mozilla plans to require secure contexts for most features. The statistics since the advent of Let's Encrypt have been impressive.

![][1]

Source: Firefox Telemetry via [https://letsencrypt.org/stats/][2]

Still, one place where privacy has been significantly lacking is in DNS. Each time one makes a connection to a website, the client must first translate the hostname to an IPv4/6 address. Being an old protocol which was not designed with security in mind, there's no encryption of DNS requests, although the results can be signed with DNSSEC to block tampering or poisoning. In this article, I will describe an approach I use myself which leverages Tor (and its DNSPort feature) plus dnsmasq running locally in order to provide necessary caching.

It's geared to users of Debian GNU/Linux 9 (stretch). Here's how it works. First, download Tor and configure your /etc/tor/torrc with the following lines:
    
    
    VirtualAddrNetworkIPv4 10.192.0.0/10  
    AutomapHostsOnResolve 1  
    AutomapHostsSuffixes .onion  
    DNSPort 5353

Now run `systemctl enable tor && systemctl restart tor`

Next, install dnsmasq and edit /etc/dnsmasq.conf:
    
    
    bind-interfaces  
    bogus-priv  
    cache-size=10000  
    conf-file=/usr/share/dnsmasq-base/trust-anchors.conf  
    dns-forward-max=1024  
    dnssec  
    dnssec-check-unsigned  
    domain-needed  
    interface=lo  
    #listen-address=::1  
    listen-address=127.0.0.1  
    localmx  
    local-service  
    local-ttl=900  
    log-async=24  
    log-queries  
    max-cache-ttl=7200  
    min-cache-ttl=300  
    no-dhcp-interface=lo  
    no-negcache  
    no-ping  
    no-poll  
    no-resolv  
    port=53  
    proxy-dnssec  
    selfmx  
    stop-dns-rebind  
    strict-order  
    # Tor's DNSPort  
    server=127.0.0.1#5354  
    # Fallback servers: Quad9, Google, OpenDNS  
    #server=9.9.9.9  
    #server=208.67.222.222  
    #server=8.8.8.8

Also edit /etc/default/dnsmasq and uncomment the line that says `IGNORE_RESOLVCONF=yes`
    
    
    options timeout:3  
    options attempts:3  
    nameserver 127.0.0.1  
    nameserver ::1

Now run `systemctl enable dnsmasq && systemctl restart dnsmasq`

**Preventing changes to resolv.conf**

Many network-related services and daemons on your system have the potential to overwrite and clobber your nameservers. This must be avoided.

DHCP leases will often hand out a new list of DNS nameservers which affects the contents of /etc/resolv.conf. Here's how to make sure that doesn't occur.

This can be performed using an empty hook on the function make_resolv_conf() that prevents any modifications at all, but the simpler method is to edit /etc/dhcp/dhclient.conf and include the following line:
    
    
    supersede domain-name-servers 127.0.0.1 ::1;

Then there's NetworkManager, which is standard within GNOME-based environments. Edit /etc/NetworkManager/NetworkManager.conf and set `dns=none` under the [main] block. You could edit that value for each interface separately under system-connections, but this is the most straightforward.

Lastly, there's systemd — Ideally I would recommend disabling the systemd-resolved service. There's many  
Automatio ways all of these services can conflict and interfere with each other.

You might want to keep it around though, as it has the ability to manage a system-wide cache of DNS responses (which is already being provided by dnsmasq anyhow). Try the following options in /etc/systemd/resolved.conf:
    
    
    [Resolve]  
    DNS=127.0.0.1  
    DNSSEC=yes  
    Cache=yes  
    DNSStubListener=no

That last option in particular will prevent it from binding to address 127.0.0.53 on port 53. Otherwise it would be set to 'udp'. Then run `systemctl restart systemd-resolved` and `systemctl dameon-reexec`.

This service has its own embedded version of resolvconf, which gets confusing. Many packages depend on resolvconf itself, and it usually turns to NetworkManager in order to know what to do. With that now out of the way from before, insert your configuration to /etc/resolvconf/resolv.conf.d/base:
    
    
    lookup file bind  
    nameserver 127.0.0.1

Here's what you'd want in /etc/resolvconf.conf in any event:
    
    
    name_servers="127.0.0.1"

You can then run `resolvconf -u` and `resolvconf --disable-updates` .

**Avoiding leaks with network firewall and iptables**

Since our DNS server listens on the loopback interface, it won't be accessible to any addresses outside of your machine. However, we'd like any attempts to inadvertently reach other DNS servers to be captured and routed through Tor as well. We can do this with iptables.

For the commonly-used firewall and wrapper around iptables called UFW set DEFAULT_FORWARD_POLICY="ACCEPT" in /etc/default/ufw.

net.ipv4.conf.eth0.route_localnet = 1

net.ipv4.ip_forward = 1

There's RFC 7858, , a "Specification for DNS over Transport Layer Security (TLS)"

and a related one, RFC 8094:

[https://tools.ietf.org/html/rfc8094][3]

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

```
ansible-galaxy install --force --ignore-errors -r requirements.yml
```

Edit the hosts inventory at `hosts` and set the IP address for your server. It's ideal to have some DNS subdomains pointing to that same server.

Then you may execute the playbook at `tor-dns-server.yml`.