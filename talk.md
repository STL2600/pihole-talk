% Project Nephology: PiHole
% https://github.com/STL2600/pihole-talk.git

# What is it?

## A DNS Server

::: notes

PiHole provides a DNS server by wrapping DNSMasq
It isn't a full featured resolver like bind
It does cache local results for faster DNS lookups
It forwards queries to your configured DNS server

:::

## A DHCP Server

::: notes

PiHole can also optionally replace your router's DHCP server
This gives you the advantage of having built in local hostname resolution for your lan

:::

## An Ad Blocker

::: notes

This is why most people run PiHole, it will block ads using DNS resolution
By intercepting and redirecting the ads to its own HTTP server, most ad-blocker-blockers are fooled.

It doesn't currently block cname ads, though they're working on that

It also won't block first party ads like in facebook or most other social networksx

:::

# Setup - RaspberryPi

::: notes

Probably the most common way to run PiHole, as you might expect from the name

:::

## Download and install Raspbian
```
wget -O raspbian.zip \
  https://downloads.raspberrypi.org/raspbian_lite_latest
unzip raspbian.zip
dd \
  bs=4M  status=progress conv=fsync \
  if=2019-09-26-raspbian-buster-lite.img \
  of=/dev/sdX
```

##  Enable SSH
```
mkdir /tmp/pi
mount /dev/sdX2 /tmp/pi
touch /tmp/pi/ssh
```

## Set a Static IP
```
echo <<EOF > /etc/dhcpcd.conf
interface eth0
static ip_address=192.168.1.2/24
static routers=192.168.1.1
static domain_name_servers=8.8.8.8
EOF
```

## Install PiHole
```
wget -O basic-install.sh \
  https://install.pi-hole.net
sudo bash basic-install.sh
```

# Setup - Standalone / Virtual Machine

## Install your base operating system

## Install PiHole
```
wget -O basic-install.sh \
  https://install.pi-hole.net
sudo bash basic-install.sh
```
Or install via your system's package manager

# Setup - Docker

## Create docker-compose.yml
```
---
version: "2"
services:
pihole:
    container_name: pihole
    image: pihole/pihole:latest
    ports:
        - "53:53/tcp"
        - "53:53/udp"
        - "67:67/udp"
        - "9080:80/tcp"
        - "9443:443/tcp"
    environment:
        TZ: 'America/Chicago'
        WEBPASSWORD: 'MyVeryGoodPassword'
    volumes:
        - /path/to/pihole/etc-pihole/:/etc/pihole/
        - /path/to/pihole/etc-dnsmasq.d/:/etc/dnsmasq.d/
    dns:
        - 127.0.0.1
        - 1.1.1.1
    cap_add:
        - NET_ADMIN
    restart: unless-stopped
```

## Deploy docker container
```
docker-compose up -d
```

# Configuration

## Ensure your machine has a static IP

::: notes

 - Since the pihole will be your DNS server, and optionally DHCP as well, it will need a static IP
 - We covered this for the pi, but for docker and a standalone machine you'll need to do it yourself.

:::

## Open your firewall

 - `53 tcp / udp`
 - `67 udp`
 - `80 tcp`
 - `443 tcp`

::: notes

 - Port 53 is for DNS
 - Port 67 is for DHCP
 - Ports 80 and 443 are for the web server

:::

## Not-Using DHCP - Configure your router

 - Update your router to provide your PiHole's IP as DNS
 - Optionally add a second DNS as a backup

::: notes

If you're not using DHCP with the PiHole, just configure it to be the DNS server for your network.

:::

## Using DHCP - Disable your router's DHCP server

::: notes

If you want to use pihole for DHCP, first disable your router's DHCP system.
Reccomend setting static IPs for at least a couple systems to ensure you can access the router and pihole.

:::

## Using DHCP - Enable DHCP on the pi hole

 - Go to `http://<pihole IP>/admin`
 - Click on the DHCP tab
 - Enable the DHCP server
 - Configure the IP address ranges
 - COnfigure the router address.

::: notes

This is more difficult, but using DHCP from the pihole will allow you to have automatic local lan name resolution.

:::

## Local DNS - Set your machine's DNS

::: notes

If you don't need or want to set your whole network to use the pi hole, you can configure it per-machine.

:::

## Optional - Setup a reverse proxy

::: notes

For access to the web interface, if you want a valid cert on the SSL interface.

The pihole must be the default server in order to work.

:::

# Additional Notes

## Caching

::: notes

The DNS caching is a bit uneven, especially for non existant DNS entries, it'll hold them for a long time.

:::

## Local Overrides

```
echo <<EOF > /etc/dnsmasq.d/99-home-network.conf
address=/server.rtward.com/192.168.1.50
address=/h.rtward.com/192.168.1.50
EOF
```

::: notes

If you have local services running that you want to resolve differently inside your home network, this is really handy.

:::

# Questions?

https://github.com/STL2600/pihole-talk.git
