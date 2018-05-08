# vagrantfile-wp

Simple Vagrantfile for local WordPress development

## Notice

The goal of this Vagrantfile is to get a basic WordPress install running locally with minimal setup.  However, as some default credentials are being used, it's not advisable to have the machine publicly accessible on the web.

## Features

- Custom hostname (requires [vagrant-hostsupdater](https://github.com/cogitatio/vagrant-hostsupdater))
- Easy syncing of host and guest directories
- Permissions compatible with both Vagrant and WordPress
- Easily reference uploads from another WordPress install

## What's in the Box?

- Ubuntu Server 16.04
- NGINX
- MySQL
- PHP7.0-FPM
- phpMyAdmin
- WP-CLI
- WordPress

## Options

There are several options at the top of the Vagrantfile.  The defaults are set for developing a theme at `wp.test`.

`hostname`

Local URL where the install can be accessed.  Use the same value in a Browsersync proxy to enable live reloading.

`target`

The directory containing the Vagrantfile will be synced to this location on the guest machine.  WordPress will be installed at `/var/www/html`.

`uploads`

URL to the uploads directory from another install. Used to view files during development without needing to have them stored locally.

## Setup

1. Ensure all dependencies are installed.
  - [VirtualBox](https://www.virtualbox.org/)
  - [Vagrant](https://www.vagrantup.com/)
  - [vagrant-hostsupdater](https://github.com/cogitatio/vagrant-hostsupdater)
  - [vagrant-auto_network](https://github.com/oscar-stack/vagrant-auto_network)
2. Copy the Vagrantfile to your project directory and set the options.
3. Run the `vagrant up` command.
  - This can take a little while if it's the first time the Ubuntu Server 16.04 box is being used on the machine.
4. Visit the `hostname` you chose in your browser and WordPress should be running.
  - Default login credentials for WordPress are admin/admin.
  - phpMyAdmin can be accessed at `hostname`/phpmyadmin with credentials root/root or username/password.
