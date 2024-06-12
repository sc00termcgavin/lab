## Set up Pi Hole

- Install Pi Hole with curl
- Follow the entire Pi Hole setup process (it will ask you to confirm settings at a few different points).
>  Note the admin dashboard password at the end of the install process; you can visit the dashboard afterwards and see DNS statistics.


```shell
wget -O basic-install.sh https://install.pi-hole.net
sudo bash basic-install.sh
```

#### Testing with dig

```shell
dig @127.0.0.1 youtube.com
```

### Adding blocklists to admin dashboard

- For Blocklists: https://firebog.net/
- URL-only and CSV versions are found https://v.firebog.net/hosts/lists.php
- Ticked Lists: https://v.firebog.net/hosts/lists.php?type=tick
- run `pihole -g` to update the database internal to pi-hole
- to update adlist: settings -> restart DNS resolver


### Chrome Bypassing pi-hole

- Chrome uses DNS over HTTPS, A more secure than basic DNS but has the downside that pi-hole cant intercept it.
- When chrome does this, it depends on how DNS is configured. If only DNS server given to your machine is pi-hole: chrome will not use DNS over HTTPS and honor pi-hole.
- If you have set up multiple DNS servers, where one can support DNS over HTTPS, chrome will use that most likely and adds may show up.

### Pi-hole as a recursive DNS server solution

Instead of disabling the security feature, you can replace it with network-wide secure DNS using `unbound`. unbound runs as a recursive DNS resolver which cuts out the third party DNS service completely and gives full control of the DNS resolver.

[Setting up Pi-hole as a recursive DNS server solution](https://docs.pi-hole.net/guides/dns/unbound/)
[Configure Pi Hole for DNS Over TLS](https://bartonbytes.com/posts/configure-pi-hole-for-dns-over-tls/)


### install the recursive DNS resolver:

```shell
$ sudo apt install -y unbound dnsutils
```

### Correct config file

```shell
$ sudo wget https://bartonbytes.com/pihole.txt -O /etc/unbound/unbound.conf.d/pihole.conf
```

### Make sure Unbound is running

```shell
$ sudo systemctl restart unbound && sudo systemctl enable unbound
```

### check status

```shell
$ sudo systemctl status unbound
```

### Use dig to check that unbound can resolve DNS names

```shell
$ dig @127.0.0.1 google.com -p 5533
```

- Tell pihole to use this unbound server for all of its outbound DNS
- go to pihole admin console -> Settings -> DNS -> Disable what you selected from installation wizzard
- Upstream DNS Server custom 1: add `127.0.0.1#5533`

### Check that pihole can resolve dns names

```shell
dig @127.0.0.1 youtube.com
```

- pi hole is doing onward dns requersts using a secure protocol rather than the insecure traditional dns.


### Tell pihole to use this unbound server for all of its outbound dns

Configure Pi-hole

Finally, configure Pi-hole to use your recursive DNS server by specifying 127.0.0.1#5335 as the Custom DNS (IPv4):