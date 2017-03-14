# ILPKit on Google Cloud Platform (Single Instance GCE) 

This guide details how to intall [ILP Kit](https://github.com/interledgerjs/ilp-kit) and all of its dependencies into a single VM running in the Google Cloud Platform (GCP) using Compute Engine.  It is an adaptation of this [authoritative guide](https://github.com/interledgerjs/ilp-kit/blob/master/docs/SETUP.md), and is estimated to take a couple of hours to complete.

This guide will utilize the following technologies:

* [GCP Compute Engine](https://cloud.google.com/compute/)
* [GCP Stackdriver Logging](https://cloud.google.com/logging/)
* Google-provided and maintained Ubuntu 16.04 image
* [nginx](https://www.nginx.com/)
* [PostgreSQL](https://www.postgresql.org/)
* [ILP Kit](https://github.com/interledgerjs/ilp-kit)
* [Mailgun](https://www.mailgun.com) (optional)
* [Github Auth](https://www.github.com) (optional)
 
> **NOTE**: This tutorial does _not_ utilize [Google Appengine](https://cloud.google.com/appengine/docs/flexible/nodejs/).

### Table of Contents

- [Google Cloud Platform (GCP) Setup](#google-cloud-platform-gcp-setup)
- [Configure Your VM](#configure-your-vm)
- [ILP Kit configuration](#ilp-kit-configuration)
- [Domain Setup](#domain-setup)
- [ILP Kit configuration](#ilp-kit-configuration)
- [Backup Your VM Disk](#backup-your-vm-disk)
- [Appendix: Advanced Options](#appendix-advanced-options)

# Google Cloud Platform (GCP) Setup

## Create a GCP Account
First, you'll need to sign-in to GCP using your Google account.  See here for more details: [https://console.cloud.google.com](https://console.cloud.google.com).

## Create a New GCP Project
In order to start a new VM, you'll need to create a project in Google Cloud.  This is as easy as selecting the "Projects" drop-down list found in the center of the header of the GCP console.

## Configure and Start a VM
Once you have a new project created, you'll want to navigate to the Google Compute Engine (GCE) section to create a new VM.  

1. Click on `Compute Engine` in the GCP menu.  
1. Click on `VM Instances` in the left-hand menu.
1. Click the `Create Instance` button.
1. Give your instance a `name`, and then, under `machine type`, select `small (1 shared vCPU)`.  This will run you about $14/month, and is the smallest image you should select.  ILP-Kit in this guide requires about 720mb of memory, which means the `micro` instance will not be powerful enough.

> Pro-Tip: Choose a high-cpu "Machine type" like `n1-highcpu-2` as a temporary solution so that package installation is speedy.  Then, once you've completed this guide, create a `Snapshot` of your disk, and spin-up a `small (1 shared vCPU)` VM instance to save money during the month.  If you do this, be sure to shut-down the expensive VM you used for this guide, or you'll end up paying for that over the month, too.

1. Next, in the `Boot Disk` section:
    1. Accept the default `10gb standard persistent disk`.  10gb is the smallest disk you can use in Compute Engine, and "standard persistent" is the cheapest.  You can upgrade to SSD if you prefer -- it may slightly speedup the installation process, but ILP-Kit is generally not disk intensive, so for your monthly use, prefer the standard disk.
    1. Click the `Change` button to select a new image type.
    1. Select the `Ubuntu 16.04 LTS` image.
1. Do nothing to accept the defaults in the `Identity and API Access` section.
1. In the `Firewall` section:
    1. Check the `Allow HTTP traffic` select box.
    1. Check the `Allow HTTPS traffic` select box.
1. Do nothing to accept the defaults in the `Management, disk, networking, SSH keys` sections.
1. Finally, click the `Create` button, and your VM will startup.

## SSH Access

### From your local machine
1. GCP includes a nice CLI that allows you to interact with the Google Cloud Platform from your local machine.  Download a version that works for your platform here: [https://cloud.google.com/sdk/downloads](https://cloud.google.com/sdk/downloads).
1. Issue the following command to configure SSH access to your VMs:

``` sh 
$ gcloud compute config-ssh
```

Once this is complete, you should receive a message with info about how to access your VM via SSH.  

Alternatively, in the GCP console, you can click the down-arrow your instance's `SSH` drop-down, and select `view gcloud command` to get the actual command.

> The first time you access the Google Cloud Platform from your local machine via the CLI, you'll need to authenticate your local machine to GCP.  The CLI makes this pretty painless though, and should auto-prompt you when needed.

### From the browser
If you don't want to connect to your VM from your local machine (and instead are fine using an SSH emulator from your browser), then you can navigate to your running GCE instances and click the `SSH` button.

# Configure Your VM
Once you have a running VM, this portion of the guide will help you prepare it for ILP-Kit installation.

## Enable Swap (optional)
By default, the GCE VM has no swap file, which may cause the `npm install` command to run out of memory and fail.  If this happens, you can enable a memory swap file ([source](http://stackoverflow.com/questions/26193654/node-js-catch-enomem-error-thrown-after-spawn).  To do that, issue the following commands:

``` sh 
me$ sudo fallocate -l 4G /swapfile #Create a 4 gigabyte swapfile
me$ sudo chmod 600 /swapfile #Secure the swapfile by restricting access to root
me$ sudo mkswap /swapfile #Mark the file as a swap space
me$ sudo swapon /swapfile #Enable the swap
```

Verify your swap by issuing the following command:

```sh 
  me$ free -m
```

It should display something like this: 

```sh 
             total        used        free      shared  buff/cache   available
Mem:            588         475          48          13          64          20
Swap:          4095         282        3813
```

## Update Packages
By default, you will be signed-in to your VM as your Google account.  However, you'll need to execute the following commands as `root`, which you can do via the following:

``` sh 
me$ sudo su
root$ apt-get update && apt-get upgrade
```

> Note: The official Debian/Ubuntu images automatically ``apt-get clean`` after each ``apt-get``

## ILPKit Database
To run ILPKit, you'll need an SQL database.  At present, ILPKit support PostgreSQL 9.6 (or higher), so you have two options: Operate a local database instance in the same VM as your ILPKit, or operate a distinct database using Google's [Cloud SQL](https://cloud.google.com/sql/docs/postgres/) product.  Each option has its pro's and con's.

### Option1: Run PostgreSQL Locally
When compared to option 2, this option is simpler to setup and cheaper to operate on an hourly basis.  For most ILPKit scenarios, this will be the preferred choice, simply due to cost.  However, it _does_ mean you'll be responsible for managing the database layer of ILPKit, including backups, patches, maintenance, etc.  But, for development or exploration purposes, this is probably the way to go.

To use this option, [follow the instructions here](./PostgresLocal.md)

### Option2: Run PostgreSQL in Google's CloudSQL
This option offers significant advantages over Option 1, but will cost more money per month.  The primary reason to select this option is so that Google will manage your database software and the VM running your database, including automated backups.  This "management" includes OS patches, as well as PostgreSQL patches and updates.  For full details, including additional reasons you would want Google run your database, [read more here](https://cloud.google.com/sql/).

To use this option, [follow the instructions here](./PostgresGCP.md)

## Install Node and NPM
In order to run ILP-Kit, you'll need [Node](https://nodejs.org/) and NPM installed in your VM.  Execute the following commands to install these libraries:

``` sh
me$ curl -sL https://deb.nodesource.com/setup_6.x | sudo -E bash -
me$ sudo apt-get install nodejs  # NOTE: running as root
```

## ILP Kit Installation

> NOTE: Make sure you're not executing these commands as root.  You should be using the default Google Account you signed-in with via SSH.

``` sh
me$ cd ~ # start in your home folder
me$ git clone https://github.com/interledgerjs/ilp-kit
me$ cd ilp-kit
me$ npm install
```

> Installation of ILP-Kit may take several minutes, especially, if you have selected a slower VM in the configuration section above.

## Install And Confgiure Nginx
Install Nginx and letsencrypt via the following command:

``` sh
 me$ sudo apt-get install nginx letsencrypt
```
We can check with systemd to make sure the service is running by typing:

``` sh
 me$ systemctl status nginx
```

It should emit something like:

``` sh 
● nginx.service - A high performance web server and a reverse proxy server
   Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset: enabled)
   Active: active (running) since Mon 2017-03-06 23:50:30 UTC; 3min 8s ago
 Main PID: 16784 (nginx)
   CGroup: /system.slice/nginx.service
           ├─16784 nginx: master process /usr/sbin/nginx -g daemon on; master_process on
           └─16785 nginx: worker process    
```

> NOTE: Before you can proceed with the rest of this installation, you'll need to setup a domain in the DNS.

# Domain setup
The following instructions detail how to setup your domain so that your ILP-Kit server can be accessible via the Interledger network, as well as so that Letsencrypt can work properly to allow your instance to operate using HTTPS.

## Subdomain setup
- Go to your name server provider of choice.
- Go to the DNS settings, and add an A record.
- I want my ilp kit on `ilpkit-gce.fluid.money`, so I'll create an A record pointing `ilpkit-gce` to my GCP machine's public IP, `104.196.252.215`

## Set up proxy for ILP Kit
Initiate an editing session of your `/etc/nginx/sites-enabled/default`.  For example:

```sh 
me$ sudo vi /etc/nginx/sites-enabled/default
```

Once in your editor, remove the `location /` block, and add these two blocks in its place:

```
# point the root to the ILP Kit UI
location / {
  proxy_pass http://localhost:3010;

  proxy_http_version 1.1;
  proxy_set_header Upgrade $http_upgrade;
  proxy_set_header Connection "upgrade";
}

# allow webfinger and letsencrypt to coexist
location ~ /.well-known/webfinger {
  proxy_pass http://localhost:3010;

  proxy_http_version 1.1;
  proxy_set_header Upgrade $http_upgrade;
  proxy_set_header Connection "upgrade";
}
```

In order to make sure you didn't mess anything up, run:

``` sh 
$ sudo nginx -t
```

If that ran OK, then restart your nginx daemon with:

``` sh
$ sudo systemctl restart nginx
```

After following the above instructions, you should be able to resolve your webserver over http from your browser (https will not work properly until the steps below are completed).  For now, the default nginx page will appear on the site until you're running your ILP Kit.

> To find your VM's external IP, just look in the GCP console under `VM Instances`.

## Setup SSL via letsencrypt
Before you can configure SSL, you'll want to have an actual webserver running. Because we're on Ubuntu, letsencrypt provides a package that does most of the set up for us on nginx.

Now follow DigitalOcean's letsencrypt instructions [here](https://www.digitalocean.com/community/tutorials/how-to-secure-nginx-with-let-s-encrypt-on-ubuntu-16-04) in order to get SSL set up.

> Note: You can skip step #4 in the DigitalOcean letsencrypt instructions because the firewall for Google VMs is configured outside of any particular VM.

This is all the domain setup that you have to do. 

# ILP Kit configuration

## Configure environment

Start the configuration tool by running:

```sh
me$ cd ~/ilp-kit
me$ npm run configure
```

The CLI provides example values, but I'll also put the configuration I'm using.

- Posgres DB URI: `postgres://ilpkit:ilpkit@localhost:ilpkit` (postgres://USER:ROLE@localhost:DB)
- Hostname: `ilpkit-gce.fluid.money`
- Ledger name: `ilpkit-gce`
- Currency code: `USD`
- Country code: `US`
- Configure GitHub: `n` (we don't need that just yet; you can always go back and change it)
- Configure Mailgun: `n` (same deal)
- Username: `admin`
- Password: (use the randomly generated default, and note it down)

**That's all you need for a functioning ILP Kit!** To start your ILP Kit, run:

```sh
me$ cd ~/ilp-kit
me$ npm start
```

Your connector's account will be automatically created and given $1000, so you can
open up your domain and log into it.

## Connecting your ledger to the ILP Network

Of course, it's not very useful to have a ledger that's not connected to any others. You can create a peering relationship with another ILP Kit by following the instructions in the original ILP Kit installation guide found [here](https://github.com/interledgerjs/ilp-kit/blob/master/docs/SETUP.md#connecting-your-ledger).  Once you're done, come back to this guide and follow the instructions below for some final setup.  

## Launch ILP Kit

You can [create a systemd](#systemd-setup) config for ILP Kit, but I'm just going to use [`nohup`](https://en.wikipedia.org/wiki/Nohup) to
run ILP Kit independent of our SSH session.

Create a file called `start.sh`, and put the following lines into it:

```sh
#!/bin/bash

killall -9 node
nohup npm start &
tail -f nohup.out
```

To make this script executable, run:

```sh
me$ chmod +x ./start.sh
```

Now when you want to modify or restart your ILP Kit, you can just run:

```
$ ./start.sh
# ... ILP Kit Logs
```

You'll see your connector connect successfully.

# Backup Your VM Disk
The Google Cloud Platform allows you to create a `Snapshot` image of any disk on any VM in your account (running or otherwise).  Once created, you can initialize a new VM using this image.  Once you have everything setup an running properly, you should consider creating a `Snapshot` of your disk so that if the VM is terminated for any reason, you won't lose your work.

Additionally:

* Consider using some of the more advanced features of Compute Engine to configure a VM from this snapshot for automatic reconfiguration in the event of a system failure.  
* Be aware that the storage engine for the PostgreSQL resides in your VM's disk or snapshot, so if you create a new VM, be sure you have a very recent snapshot or you may lose data that may have been created after your snapshot was created.
    * Once CloudSQL supports PostgreSQL, this will be less of an issue because the storage engine and its backups will be managed by Google).

# Appendix: Additional Options
The original [ILP-Kit setup guide](https://github.com/interledgerjs/ilp-kit/blob/master/docs/SETUP.md#appendix-additional-options) contains some additional pointers for advanced usage.  These include how to issue money, enable Github auth, setup Mailgun so your ILP-Kit can use email for account-related activities, and more.  Please reference that guide for more details.