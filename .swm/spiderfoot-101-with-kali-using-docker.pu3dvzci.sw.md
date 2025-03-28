---
id: pu3dvzci
title: Spiderfoot 101 with Kali using Docker
file_version: 1.1.3
app_version: 1.12.1
---

This lab walks through some simple steps required to get the OSINT tool Spiderfoot up and running on a Kali Linux using Docker.

Spiderfoot is an application that enables you as a pentester/red teamer to collect intelligence about a given subject - email address, username, domain or IP address that may help you in planning and advancing your attacks against them.

## Download Spiderfoot

Download the Spiderfoot linux package from [https://www.spiderfoot.net/download/](https://www.spiderfoot.net/download/) and extract it to a location of your choice on your file system. I extracted it to `/root/Downloads/spiderfoot-2.12.0-src/spiderfoot-2.12`

and made it my working directory:

```
cd /root/Downloads/spiderfoot-2.12.0-src/spiderfoot-2.12
```

## Upgrade PIP

You may need to upgrade the pip before it starts giving you trouble:

```
pip install --upgrade pip
```

## Build Docker Image

Build the spiderfoot docker image :

```
docker build -t spiderfoot .
```

<br/>

<div align="center"><img src="https://firebasestorage.googleapis.com/v0/b/swimm-dev-content/o/repositories%2FZ2l0aHViJTNBJTNBUmVkLVRlYW0tSW5mcmFzdHJ1Y3R1cmUtV2lraSUzQSUzQXVzZXJ0ZXN0aW5nLXN3aW1t%2F7000f015-af2e-4a85-919e-2f9d0cd2a54f.png?alt=media&token=2fdc7349-e016-4192-901c-588f864b6581" style="width:'50%'"/></div>

<br/>

This file was generated by Swimm. [Click here to view it in the app](https://swimm-web-app.web.app/repos/Z2l0aHViJTNBJTNBUmVkLVRlYW0tSW5mcmFzdHJ1Y3R1cmUtV2lraSUzQSUzQXVzZXJ0ZXN0aW5nLXN3aW1t/docs/pu3dvzci).
