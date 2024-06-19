## Setting up the server

### Step 1: install [Docker's Convience Script](https://docs.docker.com/engine/install/debian/)


```shell
curl https://get.docker.com | sh \
  && sudo systemctl --now enable docker   
```

### Step 2: Add user to docker commands

```shell
sudo usermod -aG docker $USER
```

### Step 3: Install Portainer
> Create a volume that the portainer server will use to store its database
```shell
docker volume create portainer_data
```

### Step 4: Download and Install Portainer Server Container
> ortainer generates and uses a self-signed SSL certificate to secure port 9443
> portainer available at: https:[PI-address]:9443
> Create User/Password to login
```shell 
docker run -d -p 8000:8000 -p 9443:9443 --name portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce:latest
```

### Step 5: UniFi Controller setup directories

```shell
cd ~
mkdir -p unifi/data
mkdir -p unifi/log
```


### Step 5: UniFi Controller compose file in portainer
> Pre-configured setup at [jacobalberty/unifi-docker](https://github.com/jacobalberty/unifi-docker)
> Open Portainer web interface, navigate to Stacks in local environment. Add stack, name it `unifi-controller`, build as web editor. Include the config file.

- Add environmet variable and local time-zone
  - name: TZ
  - value: America/Denver

```shell
version: "3.8"

services:
  unifi:
    user: unifi
    image: ghcr.io/jacobalberty/unifi-docker
    container_name: unifi-controller
    restart: unless-stopped
    ports:
      - "8080:8080"
      - "8443:8443"
      - "8880:8880"
      - "3478:3478/udp"
      - "10001:10001/udp"
    environment:
      TZ: ${TZ}
    volumes:
      - ./data:/unifi

```

### Step 6: Access controller web interface

- https://[IPaddress]:8443

### Step 7: Inital UniFi setup 

- Server name: `boopbeep`
> advanced setup -> set local access credentials
- username: homeadmin

### Step 8: Adopt Ubiquiti endpoints
> When you adopt Ubiquiti endpoints, they call back to the Network Application to report their status, so it’s important to make sure the latter is reachable. But since the app is running inside a Docker container, all it has to provide is a Docker-internal IP address that’s not routable from outside.
> To fix this you need to explicitly override the address in settings otherwise you’ll not be able to adopt any endpoints

- enable: setting -> system -> advanced -> inform host
- put the host IP of the raspberry pi (set via asus router)

#### Manually adopt the PoE-Lite-Switch 
> Adopting my access point worked, however the switch was failing. The solution was to manually adopt by sshing into the switch by:

```shell
ssh ubnt@[switchIP]
set-inform http://[RaspberryPiIP]:8080/inform
```

#### Update firmware 
- [Search device and copy link download](https://www.ui.com/download/software/u6-lite)
- ssh USERNAME@IP_ADDRESS
- upgrade [pastelink]
  
<!-- ### Step 5: Installing [Homepage](https://gethomepage.dev/main/installation/docker/#running-as-non-root)
> local -> stacks -> add stack -> add stack -> gib name -> add docker-compose file
> ports -> host:container
> bind the /app/config container to a /path/on/host
> mkdir -p /home/pi-admin/docker/homepage/config
```shell 
version: "3.3"
services:
  homepage:
    image: ghcr.io/gethomepage/homepage:latest
    container_name: homepage
    ports:
      - 3027:3000
    volumes:
      - /home/pi-admin/docker/homepage/config:/app/config # Make sure your local config directory exists
    environment:
      PUID: 1000
      PGID: 1000
``` -->