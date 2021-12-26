---
title: "Generate a locally trusted certificate"
date: 2021-12-25T17:42:53+01:00
draft: false
---

### Introduction

Using HTTPS with a certificate is a security best practice. I had set up a certificate for my NAS with let's encrypt for a long time, but there was that little thing that bug me: the certificate was not valid when I was using the local LAN IP address instead of the DNS.

Browsing `https://my.nas.domain.name.com:PORT` was working fine but using its private IP address `https://192.168.1.10:PORT` was raising a security warning on my browser.

There's a fantastic utility named [mkcert](https://github.com/FiloSottile/mkcert) that can help to solve this problem: generate a certificate that matches a registered domain but also a local IP address, neat!

### Generate a certificate with `mkcert`

First, we need to install mkcert. I use macOS, so I will fire `brew`:

```
brew install mkcert
brew install nss # if you use Firefox
```

Next, let's generate a certificate that will match my registered domain and my local private IPs:

```sh
mkcert -install -key-file key.pem -cert-file cert.pem  *.hocquet.fr localhost 127.0.0.1 192.168.1.10 192.168.1.1
```

As you can see here, I've also included my localhost and my router gateway address. I don't want to use different certificates for all my different local devices.

I can now import the `cert.pem` and `key.pem` files on my Synology NAS and my UniFi Dream Machine Pro.

### Install the certificate on a Synology

Installation is pretty straightforward on Synology. Just go to `Control Panel --> Security --> Certificate` and import the two generated files :

[![synology screenshot](/img/2021-12-25-generate-certificate/synology.jpeg)](/img/2021-12-25-generate-certificate/synology.jpeg)

### Install the certificate on a UniFi Dream Machine Pro

You need to have configured SSH access as a prerequisite. In the next examples, I have transferred my SSH private key on the UDM and in my `~/.ssh/config` I have:

```text
Host udm
    HostName 192.168.1.1
    User root
```

Next I need to `scp` the two generated files to the right place:

```sh
scp cert.pem key.pem udm:/root
ssh udm
cp cert.pem  /mnt/data/unifi-os/unifi-core/config/unifi-core.crt
cp key.pem  /mnt/data/unifi-os/unifi-core/config/unifi-core.key
```

Finally, we need to restart the OS to take into account the modifications:

```sh
unifi-os restart
```

### Conclusion

If everything went well, we should now get a valid certificate even though we connect directly to the LAN IP address:

[![screenshot](/img/2021-12-25-generate-certificate/cert-ok.jpeg)](/img/2021-12-25-generate-certificate/cert-ok.jpeg)

The only downside is that the certificate won't be recognized with a valid issuer on other machines than the one where `mkcert` installed it. Internally, `mkcert` installs some sort of a Certificate Authority on the OS (and on firefox) so your browser can recognize the certificate issuer as valid. But I almost never go to my router / NAS from other machines than mines, so I'm ok with that.
