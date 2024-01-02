# Wires #4: Production Server Setup

Let's configure the production environment.

<!--more-->

## Getting a domain

{{< image src="godaddy.png" position="center" style="border-radius: 8px;" caption="Acquiring a domain from GoDaddy" >}}

I have bought this domain: [ayehia0.info](https://ayehia0.info) from godaddy and configured the DNS to point to github's IP. Now going to my domain opens up my personal website which is hosted on github pages.

## Getting a server

I have also bought a server on digitalocean and created a droplet.
To get started with your droplet you just created, you have to configure a user as root user is the default one (Imagine the nasty things you can do with that!). For me I have created a user called `none` and added my SSH public key to be able to connect in a secure way.

{{< image src="digitalocean.png" position="center" style="border-radius: 8px;" caption="Digital Ocean Droplet Settings!" >}}

{{< admonition tip "Create a domain for your api" >}}
if you don't want to write your IP each time you connect to your server you need to create an `A` record in the DNS section which directs to your IP.
{{< /admonition >}}

### Connecting to the Server through SSH
Once you get everything configured, it's time to connect to the server and install all the packages and dependencies.

{{< image src="ssh_connect.png" position="center" style="border-radius: 8px;" caption="Connect to the server!" >}}

### Installing dependencies
I installed dotnet, go and docker even through I can use docker directly.
- [Installing docker](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-22-04)

### Configuring HTTPs
It's time to configure `HTTPs`, you don't want to host your website on HTTP for security reasons ofc.
Fortunately, it's easy to achieve this using [certbot](https://certbot.eff.org/pages/about) which use's [Let's Encrypt](https://letsencrypt.org/).

Following the instructions on their page (which is straight forward btw), you get HTTPs!

## All together
For now everything is manual, so if we want to update any functionality to the server, we will (which isn't ideal):
- SSH to the Server
- Make changes
- Run server

In the upcoming blogs you will create some CI/CD.

