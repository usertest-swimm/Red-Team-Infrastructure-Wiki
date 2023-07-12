---
id: 55tpg864
title: Automating Red Team Infrastructure with Terraform
file_version: 1.1.3
app_version: 1.12.1
---

## Context

The purpose of this lab was to get my hands dirty while building a simple, resilient and easily disposable red team infrastructure. Additionally, I wanted to play around with the the concept of `Infrastructure as a Code`, so I chose to tinker with a tool I have been hearing about for some time now - Terraform**.**

## **Credits**

Automated red teaming infrastructure is not a new concept - quite the opposite - I drew my inspiration from the great work of [\_RastaMouse](https://twitter.com/_RastaMouse) [where](https://rastamouse.me/2017/08/automated-red-team-infrastructure-deployment-with-terraform-part-1/) he explained his process of building an automated red team environment. He based it off of the great [wiki](https://github.com/bluscreenofjeff/Red-Team-Infrastructure-Wiki) by [Steve Borosh](https://twitter.com/424f424f) and [Jeff Dimmock](https://twitter.com/bluscreenofjeff)

<br/>

## **Infrastructure Overview**

Below is a high level diagram showing the infrastructure that I built for this lab - it can be and usually is much more built out, but the principle remains the same - redirectors are placed in front of each server to make the infrastructure more resilient to discovery that enables operators to quickly replace the burned servers with new ones:

<br/>

<div align="center"><img src="https://firebasestorage.googleapis.com/v0/b/swimm-dev-content/o/repositories%2FZ2l0aHViJTNBJTNBUmVkLVRlYW0tSW5mcmFzdHJ1Y3R1cmUtV2lraSUzQSUzQXVzZXJ0ZXN0aW5nLXN3aW1t%2F6036a1a6-2b3c-4d04-9f01-6c91fc48c6e2.png?alt=media&token=66e9caa6-9d1e-48fa-b551-fdc5e67b81e0" style="width:'50%'"/></div>

<br/>

*   There are 6 servers in total

*   3 servers (phishing, payload and c2) are considered the long term servers - we do not want our friendly blue teams to discover those

*   3 redirectors (smtp relay, payload redirector and c2 redirector) - these are the servers that sit in front of our long term servers and act as proxies. It is assumed that these servers will be detected and burned during an engagement. This is where the automation piece of Terraform comes in - since the our environment's state is defined in Terraform configuration files, we can rebuild those burned servers in almost no time and the operation can continue without bigger interruptions.

Configuring Infrastructure

### Service Providers

My test red team infrasture is built by leveraging the following services and providers:

*   DigitalOcean Droplets for all the servers and redirectors

*   DigitalOcean DNS management for the smtp relay (phishing redirector) - mostly because we need the ability to set a `PTR` DNS record for our smtp relay in order to reduce chances of our phishing email being classified as spam by target users' mail gateways

*   CloudFlare DNS management for controlling DNS records for any other domains that point to our long-term servers

Note however, you could build your servers using Amazon AWS or other popular VPS provider as long as it is supported by [Terraform](https://www.terraform.io/docs/providers/). Same applies to the DNS management piece. I used DigitalOcean and CloudFlare because I already had accounts and I like them ¯\\\_(ツ)\_/¯

### File Structure

My red team infrastructure is defined by terraform state configuration files that are currently organized in the following way:

<br/>

<div align="center"><img src="https://2603957456-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LXALNUKvZ6lpsYbF3CX%2F-LXAMYEoEsbDblCXMpFu%2FScreenshot%20from%202019-01-26%2018-13-38.png?alt=media&token=8eb3ba53-2c4f-456e-8a0b-149eaaa0a5bc" style="width:'100%%'"/></div>

<br/>

I think the file names are self explanatory, but below gives additional info on some of the config files:

*   `Configs` folder - all the config files that were too big or inconvenient to modify during Droplet creation with Terraform's provisioners. It includes configs for payload redirector (apache: `.htaccess`, `apache2.conf`), smtp redirector (postfix: `header_checks` - for stripping out email headers of the originating smtp server, `master.cf` - general postfix config for TLS and opendkim, `opendkim.conf` - configuring DKIM integration with postfix)

*   providers - required to build the infrastructure such as DigitalOcean and CloudFlare in my case

*   variables - stores API keys and similar data used across different terraform state files

*   sshkeys - stores ssh keys that our servers and redirectors will accept logons from

*   dns - defines DNS records and specify how our servers and redirectors can be accessed

*   firewalls - define access rules - who can access which server

*   outputs - a file that prints out key IP addresses and domain names of the built infrastructure

Other key points on a couple of the files are outlined below.

<br/>

### Variables

[Variables.tf](http://Variables.tf) stores things like API tokens, domain names for redirectors and c2s, operator IPs that are used in firewall rules (i.e only allow incoming connections to team server or GoPhish from an operator owned IP):

<br/>

<div align="center"><img src="https://firebasestorage.googleapis.com/v0/b/swimm-dev-content/o/repositories%2FZ2l0aHViJTNBJTNBUmVkLVRlYW0tSW5mcmFzdHJ1Y3R1cmUtV2lraSUzQSUzQXVzZXJ0ZXN0aW5nLXN3aW1t%2F29715f1b-88dc-4fd8-8b8a-d32e84025a2d.png?alt=media&token=929563f9-1b1a-475a-90e3-a2d9ba2336a3" style="width:'50%'"/></div>

<br/>

Additionally, `variables.tf` contains link to a password protected Cobalt Strike zip archive and the password itself:

<br/>

<div align="center"><img src="https://firebasestorage.googleapis.com/v0/b/swimm-dev-content/o/repositories%2FZ2l0aHViJTNBJTNBUmVkLVRlYW0tSW5mcmFzdHJ1Y3R1cmUtV2lraSUzQSUzQXVzZXJ0ZXN0aW5nLXN3aW1t%2Ffa576d17-3e12-4094-927b-fb0444905570.png?alt=media&token=a3c73505-d051-4918-b01d-20cd64abc6ed" style="width:'50%'"/></div>

<br/>

<div align="center"><img src="https://firebasestorage.googleapis.com/v0/b/swimm-dev-content/o/repositories%2FZ2l0aHViJTNBJTNBUmVkLVRlYW0tSW5mcmFzdHJ1Y3R1cmUtV2lraSUzQSUzQXVzZXJ0ZXN0aW5nLXN3aW1t%2F9f3f3535-5703-4b7e-ab2a-6f88385729f7.png?alt=media&token=eb651027-66ee-44ce-9731-2bf8ca7c861c" style="width:'50%'"/></div>

<br/>

### C2

For this lab, I chose Cobalt Strike as my C2 server.

Below is the `remote-exec` Terraform provisioner for C2 server that downloads CS zip, unzips it with a given CS password and creates a cron job to make sure the C2 server is started once the server boots up:

<br/>

This file was generated by Swimm. [Click here to view it in the app](https://swimm-web-app.web.app/repos/Z2l0aHViJTNBJTNBUmVkLVRlYW0tSW5mcmFzdHJ1Y3R1cmUtV2lraSUzQSUzQXVzZXJ0ZXN0aW5nLXN3aW1t/docs/55tpg864).
