---
layout: page
title: Authoring a Chef Cookbook as an application developer
tagline: Supporting tagline
group: guides
---
{% include JB/setup %}

This guide will walk you through preparing an OSX machine for developing a new Chef Cookbook for a java web application that you've been developing called "myface".

# System Setup

Follow this section to configure your machine with the necessary software to get started.

## Install Homebrew

Homebrew is a package manager for OSX. Follow the instructions on the [Homebrew Installation Page](https://github.com/mxcl/homebrew/wiki/installation) to install Homebrew.

This guide will assume that you have Homebrew installed but if you prefer MacPorts or to build your software manually, that is ok too.

## Install Git

Git is a distributed version control system that is used heavily by the Chef, and other open source communities.

    $ brew install git
    $ git --version
    git version 1.7.11.2
    hub version 1.8.1

> Note: Git is required even if you do not plan to store your cookbook or application in Git.

## Install rbenv and ruby-build

This will cover installing Ruby with a simple, but powerful, Ruby version manager `rbenv` and `ruby-build`.

    $ brew install rbenv
    $ if which rbenv > /dev/null; then eval "$(rbenv init -)"; fi
    $ brew install ruby-build

## Install Ruby

    $ rbenv install 1.9.3-p194

Set Ruby 1.9.3-p194 as your default Ruby version

    $ rbenv global 1.9.3-p194

And install bundler for dependency resolution

    $ gem install bundler

## Install Gecode

Gecode is a requirement of Berkshelf as of version (0.4.0.rc1).

    $ brew install gecode

## Install Berkshelf

Like Bundler (or Maven) - Berkshelf is a dependency resolver and retriever for Chef Cookbooks.

    $ gem install berkshelf

## Install VirtualBox

VirtualBox is a virtualization solution for creating virtual machines on your local computer. We will be using it to test your software and cookbooks inside a controlled environment.

Download Virtualbox from the [Virtualbox Downloads Page](https://www.virtualbox.org/wiki/Downloads) and then install it. We will be using version 4.1.18 in this guide.

## Install Vagrant

Vagrant is used in conjunction with VirtualBox and Chef to provide you with a command line tool to quickly manipulate virtual machines and rapidly iterate on your cookbook.

Install it with the gem commmand

    $ gem install vagrant

# Creating the Cookbook

Let's begin by generating a new cookbook for our application. We'll call it "myface" to match the name of our web application.

    $ berks cookbook myface --vagrant --git --foodcritic
          create  myface/files/default
          create  myface/templates/default
          create  myface/attributes
          create  myface/definitions
          create  myface/libraries
          create  myface/providers
          create  myface/recipes
          create  myface/resources
          create  myface/recipes/default.rb
          create  myface/README.md
          create  myface/metadata.rb
          create  myface/Berksfile
          create  myface/chefignore
          create  myface/.gitignore
          create  myface/Thorfile
          create  myface/Gemfile
          create  myface/Vagrantfile
    Using myface (0.0.1) at path: '/Users/reset/code/berkshelf/myface'
    Shims written to: '/Users/reset/code/berkshelf/myface/cookbooks'

This will create a skeleton for a new cookbook named 'myface' in the directory `myface` in your current working directory. The skeleton will contain some additional files to get you started iterating quickly with Berkshelf.

Passing the additional `--vagrant`, `--git`, and `--foodcritic` options will generate additional boiler plate files for your cookbook if you intend on working with Vagrant, Git, and lint testing with Foodcritic. We will be going over these topics in this guide so make sure you pass those options!

# Prepare your virtual environment

Switch into the directory of the newly created cookbook and install the Gem dependencies with bundler

    $ cd myface
    $ bundle install

This will install Vagrant, Berkshelf, and thor-foodcritic. We will be using Vagrant for building our virtual environment, Berkshelf for managing our cookbook dependencies, and thor-foodcritic to lint test our work.

A `Vagrantfile` was generated for you that, by default, provides you with a CentOS 6.3 Vagrant Box and is configured with to provision with `chef-solo`. For purposes of this guide I recommend sticking with these defaults.

The default box to use can be changed by opening the `Vagrantfile` inside your cookbook with your [favorite editor](http://www.sublimetext.com/2) and change the `config.vm.box` and `config.vm.box_url` attributes to point to the Vagrant Box of your choice.

    Vagrant::Config.run do |config|
      ...

      config.vm.box = "Berkshelf-CentOS-6.3-x86_64-minimal"
      config.vm.box_url = "https://dl.dropbox.com/u/31081437/Berkshelf-CentOS-6.3-x86_64-minimal.box"

      ...
    end

Start up your virtual machine

    $ bundle exec vagrant up
    [default] Importing base box 'centos-6.0-x86_64'...
    [default] Matching MAC address for NAT networking...
    [default] Clearing any previously set forwarded ports...
    [default] Forwarding ports...
    [default] -- 22 => 2222 (adapter 1)
    [default] Creating shared folders metadata...
    [default] Clearing any previously set network interfaces...
    [default] Booting VM...
    [default] Waiting for VM to boot. This can take a few minutes.
    [default] VM booted and ready for use!
    [default] Mounting shared folders...
    [default] -- v-root: /vagrant
    [default] -- v-csc-1: /tmp/vagrant-chef-1/chef-solo-1/cookbooks
    [default] Running provisioner: Vagrant::Provisioners::ChefSolo...
    [default] Generating chef JSON and uploading...
    [default] Running chef-solo...
    [Wed, 18 Jul 2012 07:54:58 +0200] INFO: *** Chef 10.12.0 ***
    [Wed, 18 Jul 2012 07:54:58 +0200] INFO: Setting the run_list to ["recipe[myface::default]"] from JSON
    [Wed, 18 Jul 2012 07:54:58 +0200] INFO: Run List is [recipe[myface::default]]
    [Wed, 18 Jul 2012 07:54:58 +0200] INFO: Run List expands to [myface::default]
    [Wed, 18 Jul 2012 07:54:58 +0200] INFO: Starting Chef Run for localhost
    [Wed, 18 Jul 2012 07:54:58 +0200] INFO: Chef Run complete in 0.022233916 seconds
    [Wed, 18 Jul 2012 07:54:58 +0200] INFO: Running report handlers
    [Wed, 18 Jul 2012 07:54:58 +0200] INFO: Report handlers complete

Destroy the virtual machine

    $ bundle exec vagrant destroy

# Running lint tests

    $ bundle exec thor foodcritic:lint
