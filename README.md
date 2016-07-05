# vagrantfile-wp

Simple Vagrantfile for local WordPress development

## Notice

The goal of this Vagrantfile is to get a basic WordPress install running locally with minimal setup.  However, as some default credentials are being used, it's not advisable to have the machine publicly accessible on the web.

## Features

- Custom hostname (requires [vagrant-hostsupdater](https://github.com/cogitatio/vagrant-hostsupdater))
- Easy syncing of host and guest folders
- Permissions compatible with both Vagrant and WordPress
- Basic optimizations using gzip
- Credentials match WordPress defaults for a quick setup

## What's in the Box?

- Ubuntu Server 14.04
- NGINX
- MySQL
- PHP5-FPM
- WordPress

## Options

There are several options at the top of the Vagrantfile.  The defaults are set for developing a theme at `wp.dev`.

`hostname`

Local URL where the install can be accessed.  Use the same value in a Browsersync proxy to enable live reloading.

`synced_host`

The location that will sync on the host machine.  This will commonly be the folder for a theme or plugin in development.

`synced_guest`

The location that will sync on the guest machine.  This value is relative to `/usr/share/nginx/html/` - the root location of the WordPress install.

## Setup

1. Ensure all dependencies are installed.
  - [VirtualBox](https://www.virtualbox.org/)
  - [Vagrant](https://www.vagrantup.com/)
  - [vagrant-hostsupdater](https://github.com/cogitatio/vagrant-hostsupdater)
2. Copy the Vagrantfile to your project folder and set the options.
3. Run the `vagrant up` command.
  - This can take a little while if it's the first time the Ubuntu Server 14.04 box is being used on the machine.
4. Visit the `hostname` you chose in your browser and proceed with the WordPress setup.
  - All credentials match the defaults that WordPress fills in, so there's no need to change anything.
