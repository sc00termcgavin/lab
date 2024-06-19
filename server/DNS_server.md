# Recursive DNS server on Raspberry Pi with Portainer

## Pi-hole configured as a recursive DNS server utilizing Unbound, supporting [DNS-over-TLS](https://datatracker.ietf.org/doc/html/rfc7858) and [DNS-over-HTTPS](https://datatracker.ietf.org/doc/html/rfc8484)


### Step 1: Pi-hole and Unbound via portainer

Installing pi-hole with unbound, the following settings will need to be set:
- **ServerIP**: Set the Raspberry Pi IP address
- **TZ**: Timezone (America/Denver)
- **DNSSEC**: [defaulte enabled](https://en.wikipedia.org/wiki/Domain_Name_System_Security_Extensions)

Open Ports:
- 53: DNS
- 1010: (HTTP)
- 4443: (HTTPS)
  - Open for extra functions:
  - 22/TCP: SSH connection to docker
  - 67/UDP: DHCP server


### Step 2: After Deployment, disable `auto-refresh logs` -> look for `Assigning random password:XXXXXXXX` for admin pw
- access the container consle: container -> console -> type command to change pw:
- access web interface: `http://<Raspberry IP>:1010/admin`

```shell
pihole -a -p
```


### Step 3: Setting Mainstream DNS
> `Settings` -> `DNS`

- Upstream DNS servers will be set where unbound is hosted: `127.0.0.1#5335` and `127.0.0.1#5335`


### Step 4: Conditional Forwarding

In order for Pi-Hole to ask the DHCP server the host name of local IPS, we will `Use Conditional Forwarding`. We will Set the local network using CIDR notion and input the IP address of the DHCP server.

- Define the local network CIDR format
  - `192.168.x.x/24` -  that covers all possible subnets within 192.168.x.x range
- IP address of the DHCP server (router)
  - `192.168.x.x`
- Include a local domain name 


### Step 5: Point router to Pi-----------
> [ASUS DOCS](https://www.asus.com/support/faq/1046062/) give details on how to accomplish this 
> [Pi-Hole Docs](https://docs.pi-hole.net/routers/fritzbox/) describe how to distribute pi-hole as DNS server Via DHCP


### Step 6: Adding blocklists to admin dashboard
> `group management` -> `adlists`
- For Blocklists: https://firebog.net/
- URL-only and CSV versions are found https://v.firebog.net/hosts/lists.php
- Ticked Lists: https://v.firebog.net/hosts/lists.php?type=tick
- run `pihole -g` to update the database internal to pi-hole
- to update adlist: settings -> restart DNS resolver