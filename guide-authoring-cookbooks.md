---
layout: page
title: Authoring a Chef Cookbook as an application developer
tagline: Supporting tagline
group: guides
---
{% include JB/setup %}

This is an advanced Chef guide for application developers that want to use Chef to configure and deploy their software. This guide will walk you through setting up an OSX machine with a suitable development environment and then writing a new Chef Cookbook for a java web application that you've been working on called _myface_.

You will be introduced to Vagrant and VirtualBox which will be used in conjunction with Chef for provisioning virtual machines on your local computer to quickly iterate and test your application's cookbook. I will also be introducing you to Berkshelf which you will use for resolving cookbook dependencies and retrieving them. Newcomers to Ruby may be unfamiliar with Ruby, rbenv, and Bundler but I'll walk you through a simple setup here as well.

# System Setup

Follow this section to configure your machine with the necessary software to get started.

## Install Homebrew

Homebrew is a package manager for OSX. Follow the instructions on the [Homebrew Installation Page](https://github.com/mxcl/homebrew/wiki/installation) to install Homebrew.

This guide will assume that you have Homebrew installed but if you prefer MacPorts or to build your software manually, that is ok too.

## Install Git

Git is a distributed version control system that is used heavily by the Chef, and other open source communities.

    $ brew install git
    $ git --version
    git version 1.8.3.2

> Note: Git is required even if you do not plan to store your cookbook or application in Git.

## Install rbenv and ruby-build

This will cover installing Ruby with a simple, but powerful, Ruby version manager `rbenv` and `ruby-build`.

    $ brew install rbenv
    $ if which rbenv > /dev/null; then eval "$(rbenv init -)"; fi
    $ brew install ruby-build

## Install Ruby

Ruby 1.9 is a requirement of Berkshelf. We will be using `1.9.3-p448` which is the latest patch level of 1.9.3.

    $ rbenv install 1.9.3-p448

Set Ruby 1.9.3-p448 as your default Ruby version

    $ rbenv global 1.9.3-p448

And install bundler for dependency resolution

    $ gem install bundler

## Install Berkshelf

Like Bundler (or Maven) - Berkshelf is a dependency resolver and retriever for Chef Cookbooks.

    $ gem install berkshelf

__note__: This guide requires Berkshelf (2.0.7) or later.

## Install Foodcritic

[Foodcritic](http://acrmp.github.com/foodcritic/) is linting tool for Chef Cookbooks that helps you find problems and improve your code.

    $ gem install foodcritic

## Install VirtualBox

VirtualBox is a virtualization solution for creating virtual machines on your local computer. We will be using it to test your software and cookbooks inside a controlled environment.

Download Virtualbox from the [Virtualbox Downloads Page](https://www.virtualbox.org/wiki/Downloads) and then install it. We will be using version 4.2.6 in this guide.

## Install Vagrant

[Vagrant](http://www.vagrantup.com/) provides easy to configure, reproducible, and portable work environments built on top of VirtualBox, VMWare, AWS, or any other providers. We will be using VirtualBox but you can easily switch one of the others.

Vagrant can be installed by [downloading](http://downloads.vagrantup.com/) the installer for your operating system and using standard procedures to install that package. We will be using version 1.2.4.

You also need to install Berkshelf into Vagrant (skip if you installed Vagrant using Rubygems)

    $ vagrant plugin install vagrant-berkshelf
    Installing the 'vagrant-berkshelf' plugin. This can take a few minutes...
    Installed the plugin 'vagrant-berkshelf (1.3.3)'!

It's also a great idea to install the `vagrant-omnibus` plugin which lets you specify the version of Chef you want to use.

    $ vagrant plugin install vagrant-omnibus
    Installing the 'vagrant-omnibus' plugin. This can take a few minutes...
    Installed the plugin 'vagrant-omnibus (1.1.0)'!

# Creating the Cookbook

Let's begin by generating a new cookbook for our application. We'll call it "myface" to match the name of our web application.

    $ berks cookbook myface --foodcritic
          create  myface/files/default
          create  myface/templates/default
          create  myface/attributes
          create  myface/definitions
          create  myface/libraries
          create  myface/providers
          create  myface/recipes
          create  myface/resources
          create  myface/recipes/default.rb
          create  myface/metadata.rb
          create  myface/LICENSE
          create  myface/README.md
          create  myface/Berksfile
          create  myface/Thorfile
          create  myface/chefignore
          create  myface/.gitignore
             run  git init from "./myface"
          create  myface/Gemfile
          create  myface/Vagrantfile

This will create a skeleton for a new cookbook named 'myface' in the directory `myface` in your current working directory. The skeleton will contain some additional files to get you started iterating quickly with Berkshelf.

Passing the additional `--foodcritic` option will generate additional boilerplate files for your cookbook if you intend on lint testing with Foodcritic. We will be going over this topic in this guide so make sure you pass this option!

# Prepare your virtual environment

Switch into the directory of the newly created cookbook and install the Gem dependencies with bundler

    $ cd myface
    $ bundle install

Bundler will install all of the dependent RubyGems and guarantee that you have the right versions.

## Starting your virtual machine

A `Vagrantfile` was generated for you with a boilerplate configuration that should be suitable for our needs. The default Vagrantfile is configured to download and boot a CentOS 6.3 Vagrant Box and provision it with `chef-solo`.

Let's update the Vagrant file to pull down a more up to date box and configure a more recent version of chef.

    Vagrant.configure("2") do |config|

      # All Vagrant configuration is done here. The most common configuration
      # options are documented and commented below. For a complete reference,
      # please see the online documentation at vagrantup.com.

      config.vm.hostname = "myface-berkshelf"

      config.vm.box = "misheska-centos6.4"
      config.vm.box_url = "https://www.dropbox.com/s/y733o4ifkowc1w0/misheska-centos64.box"

      config.omnibus.chef_version = "11.6.0"

      # Assign this VM to a host-only network IP, allowing you to access it
      # via the IP. Host-only networks can talk to the host machine as well as
      # any other machines on the same network, but cannot be accessed (through this
      # network interface) by any external networks.
      config.vm.network :private_network, ip: "33.33.33.10"
      ...
    end

Check the full [Vagrant Documentation](http://vagrantup.com/v1/docs/index.html) for future reference.


Start up your virtual machine

    $ bundle exec vagrant up
    Bringing machine 'default' up with 'virtualbox' provider...
    [default] Importing base box 'misheska-centos6.4'...
    [default] Matching MAC address for NAT networking...
    [default] Setting the name of the VM...
    [default] Clearing any previously set forwarded ports...
    [Berkshelf] Using myface (0.1.0)
    [default] Fixed port collision for 22 => 2222. Now on port 2200.
    [default] Creating shared folders metadata...
    [default] Clearing any previously set network interfaces...
    [default] Preparing network interfaces based on configuration...
    [default] Forwarding ports...
    [default] -- 22 => 2200 (adapter 1)
    [default] Booting VM...
    [default] Waiting for VM to boot. This can take a few minutes.
    [default] VM booted and ready for use!
    [default] Setting hostname...
    [default] Configuring and enabling network interfaces...
    [default] Mounting shared folders...
    [default] -- /vagrant
    [default] -- /tmp/vagrant-chef-1/chef-solo-1/cookbooks
    [default] Installing Chef 11.6.0 Omnibus package...
    [default] Running provisioner: chef_solo...
    Generating chef JSON and uploading...
    Running chef-solo...
    [2013-07-24T12:12:24-07:00] INFO: Forking chef instance to converge...
    [2013-07-24T12:12:24-07:00] INFO: *** Chef 11.6.0 ***
    [2013-07-24T12:12:24-07:00] INFO: Setting the run_list to ["recipe[myface::default]"] from JSON
    [2013-07-24T12:12:24-07:00] INFO: Run List is [recipe[myface::default]]
    [2013-07-24T12:12:24-07:00] INFO: Run List expands to [myface::default]
    [2013-07-24T12:12:24-07:00] INFO: Starting Chef Run for myface-berkshelf
    [2013-07-24T12:12:24-07:00] INFO: Running start handlers
    [2013-07-24T12:12:24-07:00] INFO: Start handlers complete.
    [2013-07-24T12:12:24-07:00] INFO: Chef Run complete in 0.01797156 seconds
    [2013-07-24T12:12:24-07:00] INFO: Running report handlers
    [2013-07-24T12:12:24-07:00] INFO: Report handlers complete


If at anytime your virtual machine becomes unstable or if you'd like to start over you can destroy your virtual machine with one command

    $ bundle exec vagrant destroy
    [default] Forcing shutdown of VM...
    [default] Destroying VM and associated drives...
    [Berkshelf] cleaning Vagrant's shelf

__note__: If you destroy VM make sure to recreate it with `bundle exec vagrant up` before moving to the next section

# Deploying with Artifact Deploy

Now that we've got a barebones cookbook for our hot new social networking application, myface, let's make our first cookbook change and deploy our application with Artifact Deploy. Artifact Deploy is a Light-weight Resource and Provider (LWRP) that comes bundled with the [Artifact Cookbook](https://github.com/riotgames/artifact-cookbook).

In Chef, a resource represents a piece of system state and a provider is the underlying implementation which brings the resource into the desired state. Chef comes with a number of Resources and Providers for you to use out of the box but you can create your own by generating a cookbook that contains an LWRP and including it into the metadata of another cookbook.

Open the default recipe for editing at `myface/recipes/default.rb` and add following code block.

    artifact_deploy "myface" do
      version "1.0.0"
      artifact_location "http://dl.dropbox.com/u/31081437/myface-1.0.0.tar.gz"
      deploy_to "/srv/myface"
      owner "myface"
      group "myface"
      action :deploy
    end

Save your work.

Next we will re-provision your virtual machine by running Vagrant's __provision__ command. Provision will re-run the Chef-Solo provisioner. This is the same provisioner that ran earlier when we started our virtual machine.

Through the magic of the Berkshelf Vagrant plugin - Vagrant will automatically make any changes you make to your cookbook and all of your cookbook's dependencies available to the virtual machine.

    $ bundle exec vagrant provision

You should have experienced a failure in the provisioning

    [Berkshelf] Using myface (0.1.0)
    [default] Chef 11.6.0 Omnibus package is already installed.
    [default] Running provisioner: chef_solo...
    Generating chef JSON and uploading...
    Running chef-solo...
    [2013-07-24T12:18:22-07:00] INFO: Forking chef instance to converge...
    [2013-07-24T12:18:22-07:00] INFO: *** Chef 11.6.0 ***
    [2013-07-24T12:18:22-07:00] INFO: Setting the run_list to ["recipe[myface::default]"] from JSON
    [2013-07-24T12:18:22-07:00] INFO: Run List is [recipe[myface::default]]
    [2013-07-24T12:18:22-07:00] INFO: Run List expands to [myface::default]
    [2013-07-24T12:18:22-07:00] INFO: Starting Chef Run for myface-berkshelf
    [2013-07-24T12:18:22-07:00] INFO: Running start handlers
    [2013-07-24T12:18:22-07:00] INFO: Start handlers complete.


    ================================================================================

    Recipe Compile Error in /tmp/vagrant-chef-1/chef-solo-1/cookbooks/myface/recipes/default.rb

    ================================================================================

    NameError

    ---------

    Cannot find a resource for artifact_deploy on centos version 6.4

    Cookbook Trace:

    ---------------

      /tmp/vagrant-chef-1/chef-solo-1/cookbooks/myface/recipes/default.rb:1:in `from_file'

    Relevant File Content:

    ----------------------

    /tmp/vagrant-chef-1/chef-solo-1/cookbooks/myface/recipes/default.rb:

      1>> artifact_deploy "myface" do
      2:    version "1.0.0"
      3:    artifact_location "http://dl.dropbox.com/u/31081437/myface-1.0.0.tar.gz"
      4:    deploy_to "/srv/myface"
      5:    owner "myface"
      6:    group "myface"
      7:    action :deploy
      8:  end
      9:


    [2013-07-24T12:18:22-07:00] ERROR: Running exception handlers
    [2013-07-24T12:18:22-07:00] ERROR: Exception handlers complete
    [2013-07-24T12:18:22-07:00] FATAL: Stacktrace dumped to /var/chef/cache/chef-stacktrace.out
    [2013-07-24T12:18:22-07:00] FATAL: Chef::Exceptions::ChildConvergeError: Chef run process exited unsuccessfully (exit code 1)
    Chef never successfully completed! Any errors should be visible in the
    output above. Please fix your recipes so that they properly complete.

The important bit is the NameError. It appears that we do not have the resource `artifact_deploy` available on CentOS 6.4.

You can also see what file the error occurred in and on what line by looking at the Cookbook Trace section. In this case it's on line 1 in the default recipe of the myface cookbook.

    /tmp/vagrant-chef-1/chef-solo-1/cookbooks/myface/recipes/default.rb:1:in `from_file'

We actually don't have the `artifact_deploy` resource at all; it's not part of Chef or the only cookbook available to our Chef Client. This is because we haven't told our cookbook about the Artifact cookbook which contains the Light-weight Resource and Provider (LWRP) that provides `artifact_deploy` to our recipes.

## Working with cookbook metadata

To tell our myface cookbook about the Artifact cookbook we need to modify the `metadata.rb` file at the root of our cookbook's directory. This is an often overlooked file to new Chef developers but it is one of the most important.

The metadata file is a lot like a RubyGems `gemspec`, it tells your Chef Server some important things about your cookbook such as:

* the name of a cookbook set by the `name` attribute
* the version of a cookbook set by the `version` attribute
* a list of dependent cookbooks and optionally their versions set by `depends` definitions
* a list of conflicting cookbooks and optionally their versions set by `conflicts` definitions
* the maintainer of the cookbook set by the `maintainer` attribute
* the email address of the maintainer of the cookbook set by the `maintainer_email` attribute
* license information set by the `license` attribute
* a description of the cookbook set by the `description` and `long_description` attributes

It is important to note that not all of these attributes are required. Surprisingly, the name attribute is optional. It is dangerous to leave this attribute blank because the name of the cookbook will then be inferred by the directory containing the contents of the cookbook when it is loaded. Remember to __always set the name attribute for your cookbook__ and save operators or fellow cookbook authors a headache.

For more information see the complete documentation for [Cookbook Metadata](http://wiki.opscode.com/display/chef/Metadata).

Open up the `metadata.rb` file in our cookbook and add the following line of code to the bottom

    depends "artifact", "~> 0.10.7"

This tells the Chef server and clients that the myface cookbook depends on the artifact cookbook. We've also provided a version constraint to the dependency which ensures that other cookbook authors or operators are using a version of the artifact cookbook that we approve works with our cookbook. You should __always set reasonable version constraints for your dependencies__ to save your operators and fellow cookbook authors from wanting to [light you on fire](http://24.media.tumblr.com/tumblr_m7fpxfkHM81rzupqxo1_500.png).

Cookbooks follow the [SemVer](http://semver.org) versioning scheme and accept constraints containing anyone of the approved constraint operators. In this case we've used the optimistic operator `~>` to tell our cookbook that we allow any version of artifact that is greater than 0.10.7 but _not_ 0.11.0 or higher. This means we accept 0.10.7, 0.10.8, or 0.10.30002, etc.

Now you should have a `metadata.rb` file that looks like this

    name             "myface"
    maintainer       "YOUR_NAME"
    maintainer_email "YOUR_EMAIL"
    license          "All rights reserved"
    description      "Installs/Configures myface"
    long_description IO.read(File.join(File.dirname(__FILE__), 'README.md'))
    version          "0.0.1"

    depends "artifact", "~> 0.10.7"

Now re-run the vagrant provisioner and see what we get

    $ bundle exec vagrant provision
    [Berkshelf] Using myface (0.1.0)
    [Berkshelf] Using artifact (0.10.10)
    [Berkshelf] Using nexus (2.2.0)
    [Berkshelf] Using java (1.9.6)
    [Berkshelf] Using windows (1.10.0)
    [Berkshelf] Using chef_handler (1.1.4)
    [Berkshelf] Using nginx (1.6.0)
    [Berkshelf] Using build-essential (1.4.0)
    [Berkshelf] Using yum (2.3.0)
    [Berkshelf] Using apt (2.0.0)
    [Berkshelf] Using runit (1.1.6)
    [Berkshelf] Using ohai (1.1.10)
    [default] Chef 11.6.0 Omnibus package is already installed.
    [default] Running provisioner: chef_solo...
    Generating chef JSON and uploading...
    Running chef-solo...
    ...
    ================================================================================

    Error executing action `create` on resource 'remote_file[/var/chef/cache/artifact_deploys/myface/1.0.0/myface-1.0.0.tar.gz]'

    ================================================================================
    ...

    Chef::Exceptions::UserIDNotFound

    --------------------------------

    cannot determine user id for 'myface', does the user exist on this system?
    ...

Well we don't have the NameError exception anymore but we have a new problem. The user that we're attempting to deploy the artifact with is not found on the system. It's a good idea to avoid running your applications as root and to create a user to run them. Let's create the 'myface' user.

## Creating an application user

Open the default recipe for editing at `myface/recipes/default.rb` and define a new [Group Resource](http://wiki.opscode.com/display/chef/Resources#Resources-Group) and [User Resource](http://wiki.opscode.com/display/chef/Resources#Resources-User) for the myface group and user above your artifact_deploy resource.

    group "myface"

    user "myface" do
      group "myface"
      system true
      shell "/bin/bash"
    end

Save your work and re-run the Vagrant provisioner

    $ bundle exec vagrant provision
    [2012-09-20T20:57:48+00:00] INFO: Chef Run complete in 27.94327067 seconds
    [2012-09-20T20:57:48+00:00] INFO: Running report handlers
    [2012-09-20T20:57:48+00:00] INFO: Report handlers complete

You should now have your application code deployed to `/srv/myface/current`

    $ bundle exec vagrant ssh -c "ls -lah /srv/myface/current"
    lrwxrwxrwx 1 root root 26 Jul 23 22:14 /srv/myface/current -> /srv/myface/releases/1.0.0

## Using Berkshelf to gather dependencies

Let's take a quick pause to answer a question you may be wondering, "How Berkshelf was able to figure out I needed the artifact cookbook and where to get it?".

When we generated the cookbook earlier on it created a file called `Berksfile` at the root of the myface cookbook. This file is read by Berkshelf's CLI and Vagrant plugin. Much like the Gemfile in Bundler, it describes the dependencies of your project.

If you open the Berksfile that was generated for you, you will see just one line with one word

    metadata

If you are familiar with RubyGems and Bundler this is similar to the their `gemspec` keyword. Berkshelf will inspect the `metadata.rb` file of the cookbook and recursively download the dependencies of your cookbook and their dependencies (and so on). Berkshelf will search the Opscode community site for a cookbook matching the version constraint if you do not explicitly provide a source for the artifact cookbook in your Berksfile.

If the version of the artifact cookbook wasn't available on the Opscode community site or you just want to host things locally an explicit source can be provided for where the cookbook can be found

    cookbook 'artifact', '~> 0.10.1', chef_api: :config

This entry in your Berksfile would tell Berkshelf to look at at Chef API using your Berkshelf configuration for authorization to download the artifact cookbook. Let's leave things as is for now for the purposes of this guide.

## The deployed application structure

The value passed to the `deploy_to` attribute on artifact_deploy resource is where we want to deploy our code to. You'll notice, though, that the contents of the archive weren't extracted directly into `/srv/myface`. Instead the code was placed in a directory of the same name as the value given for the `version` attribute in the releases directory sub directory. This is done so we can have multiple different versions of our code on the same node.

The `deploy_to` directory contains three sub directories

    lrwxrwxrwx  1 root   root     26 Jul 23 22:14 current -> /srv/myface/releases/1.0.0
    drwxr-xr-x  3 root   root   4.0K Jul 23 22:12 releases
    drwxr-xr-x  5 myface myface 4.0K Jul 23 22:12 shared

* releases directory that contains directories whose names are SemVer versions and contain the contents of the artifact of the same version
* current directory which points to the release directory of the version of our software that we deem 'active'
* shared directory that contains common files that we want to persist between artifact versions. Log files are a good example of a shared file

This deploy structure was made popular by [Capistrano](https://github.com/capistrano/capistrano). If you are familiar with Capistrano's than this should be very familiar to you.

# Refactoring into attributes

If you've got a good eye for good development practices you might have noticed that we repeated ourself quite a bit in our default recipe, specifically when referencing the group and user. We used the string "myface" in our first pass to identify the group and user name and also needed to give that to the artifact_deploy resource.

It's very common to need to provide repetitive data to resources in a Chef recipe and a good idea to abstract these into an a primitive that Chef calls an [Attribute](http://wiki.opscode.com/display/chef/Attributes). An attribute holds configurable and searchable node data.

This refactor is valuable for a few reasons

* Ensures that a cookbook author does not typo the group or user in a change
* Allows an operator to customize the group and user name
* Other cookbook authors can query the node about this information and use it in recipes of their own

You should __always abstract your tunables and constants into attributes__. Let's [DRY](http://en.wikipedia.org/wiki/Don't_repeat_yourself) up our code and replace these strings with attributes.

Open the default recipe for editing at `myface/recipes/default.rb` and replace the "myface" string for user with `node[:myface][:user]` and the "myface" string for group with `node[:myface][:group]`. Now your default recipe should look like this

    group node[:myface][:group]

    user node[:myface][:user] do
      group node[:myface][:group]
      system true
      shell "/bin/bash"
    end

    artifact_deploy "myface" do
      version "1.0.0"
      artifact_location "http://dl.dropbox.com/u/31081437/myface-1.0.0.tar.gz"
      deploy_to "/srv/myface"
      owner node[:myface][:user]
      group node[:myface][:group]
      action :deploy
    end

Within a recipe attributes are accessed on the [Node Object](http://wiki.opscode.com/display/chef/Recipes#Recipes-NodeObject). Attributes are accessed in a similar fashion to how you would interact with a Hash in Ruby. You can use a symbol or a string for keys in your attributes and they are interchangeable, Chef will not complain and tell you that your attribute is not defined if you use a string when you initialized it with a symbol. Although you can use strings, it is __strongly recommended that you [use symbols for keys in Ruby](http://www.robertsosinski.com/2009/01/11/the-difference-between-ruby-symbols-and-strings/)__.

So let's re-provision with Vagrant and see how our refactor went

    $ bundle exec vagrant provision
    [Mon, 23 Jul 2012 23:37:07 +0000] INFO: *** Chef 11.6.0 ***
    [Mon, 23 Jul 2012 23:37:07 +0000] INFO: Setting the run_list to ["recipe[myface::default]"] from JSON
    [Mon, 23 Jul 2012 23:37:07 +0000] INFO: Run List is [recipe[myface::default]]
    [Mon, 23 Jul 2012 23:37:07 +0000] INFO: Run List expands to [myface::default]
    [Mon, 23 Jul 2012 23:37:07 +0000] INFO: Starting Chef Run for localhost
    [Mon, 23 Jul 2012 23:37:07 +0000] INFO: Running start handlers
    [Mon, 23 Jul 2012 23:37:07 +0000] INFO: Start handlers complete.
    [Mon, 23 Jul 2012 23:37:07 +0000] ERROR: Running exception handlers
    [Mon, 23 Jul 2012 23:37:07 +0000] ERROR: Exception handlers complete
    [Mon, 23 Jul 2012 23:37:07 +0000] FATAL: Stacktrace dumped to /tmp/vagrant-chef-1/chef-stacktrace.out
    [Mon, 23 Jul 2012 23:37:07 +0000] FATAL: NoMethodError: undefined method `[]' for nil:NilClass
    Chef never successfully completed! Any errors should be visible in the
    output above. Please fix your recipes so that they properly complete.

Not so well... it seems that we have an undefined method somewhere!?

    [Mon, 23 Jul 2012 23:37:07 +0000] FATAL: Stacktrace dumped to /tmp/vagrant-chef-1/chef-stacktrace.out
    [Mon, 23 Jul 2012 23:37:07 +0000] FATAL: NoMethodError: undefined method `[]' for nil:NilClass

Well, yes and no. This cryptic error message is a very common and people new to Ruby are often confused by it at first so let's break it down. Everything in Ruby is an object - even Nil. Ruby also allows the characters `[` and `]` in function definitions. We're attempting to send the message `[]` to an instance of nil because we are accessing the nested attribute `node[:myface][:user]` but we haven't set set a value for the attribute `node[:myface]` it by default evaluates to nil. Asking for the attribute `:user` from `nil` results in the no method error that we see above. We can easily remedy this by initializing attributes before we attempt to access them. You should __always initialize attributes before using them__.

## Initialize your attributes

Attributes can be set and accessed from a [Node Object](http://wiki.opscode.com/display/chef/Recipes#Recipes-NodeObject) and also within attribute files.

Create a new file that sets values for the missing attributes and save it as `myface/attributes/default.rb`.

    default[:myface][:user] = "myface"
    default[:myface][:group] = "myface"

The syntax for accessing an attribute in an attribute file is different than we've seen in recipes. You do not access them from a Node Object, instead you access them by precedence. In this case we are using the default precedence which, when set in an attribute file, has a precedence level of 1. Attribute precedence is commonly a source for bugs or confusion so it's a good idea to read up the full documentation on [Attribute Precedence](http://wiki.opscode.com/display/chef/Attributes#Attributes-AttributeTypeandPrecedence) before moving forward. Currently there are 11 precedence levels in total where 1 is lowest and 11 is highest.

re-run Vagrant provision and our attributes should replace our hardcoded strings

    $ bundle exec vagrant provision
    [default] Running provisioner: Vagrant::Provisioners::ChefSolo...
    [default] Generating chef JSON and uploading...
    [default] Running chef-solo...
    [Tue, 24 Jul 2012 01:30:30 +0000] INFO: *** Chef 11.6.0 ***
    [Tue, 24 Jul 2012 01:30:30 +0000] INFO: Setting the run_list to ["recipe[myface::default]"] from JSON
    [Tue, 24 Jul 2012 01:30:30 +0000] INFO: Run List is [recipe[myface::default]]
    [Tue, 24 Jul 2012 01:30:30 +0000] INFO: Run List expands to [myface::default]
    [Tue, 24 Jul 2012 01:30:30 +0000] INFO: Starting Chef Run for localhost
    [Tue, 24 Jul 2012 01:30:30 +0000] INFO: Running start handlers
    [Tue, 24 Jul 2012 01:30:30 +0000] INFO: Start handlers complete.
    [Tue, 24 Jul 2012 01:30:31 +0000] INFO: Processing group[myface] action create (myface::default line 10)
    [Tue, 24 Jul 2012 01:30:31 +0000] INFO: Processing user[myface] action create (myface::default line 12)
    [Tue, 24 Jul 2012 01:30:31 +0000] INFO: Processing artifact_deploy[myface] action deploy (myface::default line 18)
    [Tue, 24 Jul 2012 01:30:31 +0000] INFO: Chef Run complete in 0.09489162 seconds
    [Tue, 24 Jul 2012 01:30:31 +0000] INFO: Running report handlers
    [Tue, 24 Jul 2012 01:30:31 +0000] INFO: Report handlers complete

Success! You'll notice that your Chef run went a lot faster than the first time we successfully ran and deployed our application. That is because we're using well defined resources which are completely [idempotent](http://en.wikipedia.org/wiki/Idempotence).

# Idempotent recipes

You should __always write idempotent recipes__ that execute cleanly on their very first run and perform no work if no work needs to be done. You can do this by selecting the right resource for the job or creating your own by writing a Light-weight Resource Provider (LWRP). If you've been working with Chef and reading cookbooks authored by people new to Chef then you've probably seen something like this

    bash "download-and-extract" do
      code <<-EOH
        cd /tmp
        wget http://dl.dropbox.com/u/31081437/myface-1.0.0.tar.gz
        mkdir -p /srv/myface/releases/1.0.0
        tar xzvf myface-1.0.0.tar.gz -C /srv/myface/releases/1.0.0
        rm -Rdf /tmp/myface-1.0.0.tar.gz
      EOH
    end

This snippet of code is actually very similar to what we did with the artifact_deploy resource

1. Downloads the artifact
2. Creates the release directory
3. Extracts the artifact into the release directory
4. Cleans up after itself

The problem is that this resource definition is not idempotent. This resource will be executed every time Chef is run regardless if version 1.0.0 of myface has been installed to /srv/myface/releases/1.0.0. This could be prevented with by using a [Conditional Execution](http://wiki.opscode.com/display/chef/Resources#Resources-ConditionalExecution) check

    bash "download-and-extract" do
      code <<-EOH
        cd /tmp
        wget http://dl.dropbox.com/u/31081437/myface-1.0.0.tar.gz
        mkdir -p /srv/myface/releases/1.0.0
        tar xzvf myface-1.0.0.tar.gz -C /srv/myface/releases/1.0.0
        rm -Rdf /tmp/myface-1.0.0.tar.gz
      EOH

      not_if { File.exists?("/srv/myface/releases/1.0.0") }
    end

But as the logic for deploying an artifact grows more complex, the conditional execution block also grows in complexity.

_A Chef recipe is not a collection of procedurally executing bash scripts_

Always use the right resource for the job and avoid using the [Script resource](http://wiki.opscode.com/display/chef/Resources#Resources-Script) where possible. In this case, we can (and did) use the artifact deploy Light-weight Resource Provider (LWRP) to ensure our deployment steps are idempotent.

# Configuring the application server

Open up the `metadata.rb` file in our cookbook and add a dependency for Tomcat below the artifact dependency (note: order does not matter)

    depends "tomcat", "~> 0.11.0"

Now you should have a `metadata.rb` file that looks like this

    name             "myface"
    maintainer       "YOUR_NAME"
    maintainer_email "YOUR_EMAIL"
    license          "All rights reserved"
    description      "Installs/Configures myface"
    long_description IO.read(File.join(File.dirname(__FILE__), 'README.md'))
    version          "0.0.1"

    depends "artifact", "~> 0.10.1"
    depends "tomcat", "~> 0.11.0"

Next we will tell our default recipe to install Tomcat and configure it before deploying our application. Open the default recipe `myface/recipes/default.rb` for editing and add an `include_recipe` call before the resource definitions and save your work. You should have

    #
    # Cookbook Name:: myface
    # Recipe:: default
    #
    # Copyright (C) 2012 YOUR_NAME
    #
    # All rights reserved - Do Not Redistribute
    #

    include_recipe "tomcat"

    group node[:myface][:group]
    ...

Re-run the Vagrant provisioner to install to your virtual machine

    $ bundle exec vagrant provision
    [Tue, 24 Jul 2012 23:30:25 +0000] INFO: *** Chef 10.14.2 ***
    [Tue, 24 Jul 2012 23:30:26 +0000] INFO: Setting the run_list to ["recipe[myface::default]"] from JSON
    [Tue, 24 Jul 2012 23:30:26 +0000] INFO: Run List is [recipe[myface::default]]
    [Tue, 24 Jul 2012 23:30:26 +0000] INFO: Run List expands to [myface::default]
    ...
    [Tue, 24 Jul 2012 23:36:45 +0000] INFO: Processing service[tomcat] action restart (tomcat::default line 37)
    [Tue, 24 Jul 2012 23:26:45 +0000] INFO: service[tomcat] restarted
    [Tue, 24 Jul 2012 23:26:45 +0000] INFO: Chef Run complete in 380.647822346 seconds
    [Tue, 24 Jul 2012 23:26:45 +0000] INFO: Running report handlers
    [Tue, 24 Jul 2012 23:26:45 +0000] INFO: Report handlers complete

After a little bit of time Tomcat and all of it's requirements, including Java 1.6, will be installed and configured within your virtual machine.

Writing idempotent recipes is so important that we should give our Vagrant provisioner another run just to make sure that no work is being performed after we've already done it

    $ bundle exec vagrant provision
    [default] Running provisioner: Vagrant::Provisioners::ChefSolo...
    [default] Generating chef JSON and uploading...
    [default] Running chef-solo...
    [Wed, 25 Jul 2012 00:49:05 +0000] INFO: *** Chef 11.6.0 ***
    [Wed, 25 Jul 2012 00:49:06 +0000] INFO: Setting the run_list to ["recipe[myface::default]"] from JSON
    [Wed, 25 Jul 2012 00:49:06 +0000] INFO: Run List is [recipe[myface::default]]
    [Wed, 25 Jul 2012 00:49:06 +0000] INFO: Run List expands to [myface::default]
    [Wed, 25 Jul 2012 00:49:06 +0000] INFO: Starting Chef Run for localhost
    [Wed, 25 Jul 2012 00:49:06 +0000] INFO: Running start handlers
    [Wed, 25 Jul 2012 00:49:06 +0000] INFO: Start handlers complete.
    [Wed, 25 Jul 2012 00:49:06 +0000] INFO: Processing ruby_block[set-env-java-home] action create (java::openjdk line 36)
    [Wed, 25 Jul 2012 00:49:06 +0000] INFO: ruby_block[set-env-java-home] called
    [Wed, 25 Jul 2012 00:49:06 +0000] INFO: Processing ruby_block[update-java-alternatives] action nothing (java::openjdk line 43)
    [Wed, 25 Jul 2012 00:49:06 +0000] INFO: Processing package[java-1.6.0-openjdk] action install (java::openjdk line 80)
    [Wed, 25 Jul 2012 00:49:07 +0000] INFO: Processing package[java-1.6.0-openjdk-devel] action install (java::openjdk line 80)
    [Wed, 25 Jul 2012 00:49:07 +0000] INFO: Processing package[sun-java6-jdk] action purge (java::default line 25)
    [Wed, 25 Jul 2012 00:49:07 +0000] INFO: Processing package[sun-java6-bin] action purge (java::default line 25)
    [Wed, 25 Jul 2012 00:49:07 +0000] INFO: Processing package[sun-java6-jre] action purge (java::default line 25)
    [Wed, 25 Jul 2012 00:49:07 +0000] INFO: Processing package[tomcat6] action install (tomcat::default line 32)
    [Wed, 25 Jul 2012 00:49:07 +0000] INFO: Processing package[tomcat6-admin-webapps] action install (tomcat::default line 32)
    [Wed, 25 Jul 2012 00:49:07 +0000] INFO: Processing service[tomcat] action enable (tomcat::default line 37)
    [Wed, 25 Jul 2012 00:49:07 +0000] INFO: Processing service[tomcat] action start (tomcat::default line 37)
    [Wed, 25 Jul 2012 00:49:07 +0000] INFO: Processing template[/etc/sysconfig/tomcat6] action create (tomcat::default line 50)
    [Wed, 25 Jul 2012 00:49:07 +0000] INFO: Processing template[/etc/tomcat6/server.xml] action create (tomcat::default line 67)
    [Wed, 25 Jul 2012 00:49:07 +0000] INFO: Processing group[myface] action create (myface::default line 12)
    [Wed, 25 Jul 2012 00:49:07 +0000] INFO: Processing user[myface] action create (myface::default line 14)
    [Wed, 25 Jul 2012 00:49:07 +0000] INFO: Processing artifact_deploy[myface] action deploy (myface::default line 20)
    [Wed, 25 Jul 2012 00:49:07 +0000] INFO: Chef Run complete in 1.574612849 seconds
    [Wed, 25 Jul 2012 00:49:07 +0000] INFO: Running report handlers
    [Wed, 25 Jul 2012 00:49:07 +0000] INFO: Report handlers complete

Each successive Chef run is now taking 1.5 seconds to inspect the node to ensure that no work needs to be performed. You might be wondering why Chef is doing so much "Processing" if no work is getting done. Chef output is verbose and may be hard to grok for new a user so let's quickly dissect one of the INFO logs to better understand.

    [Wed, 25 Jul 2012 00:49:07 +0000] INFO: Processing artifact_deploy[myface] action deploy (myface::default line 20)

* `[Wed, 25 Jul 2012 00:49:07 +0000]` this is the time the log resource was "processed" or "inspected" at
* `INFO` this is the log level of the log. INFO is good. WARN maybe an issue. FATAL and ERROR are bad.
* `Processing artifact_deploy[myface]` we are processing the a resource called "myface" of type artifact_deploy. You can think of processing as inspecting for change.
* `action deploy` the action `:deploy` is being considered for execution. If the resource is not in a state which would match that if we had performed the action of `:deploy` than the action would execute and the resource would change state.
* `(myface::default line 20)` this is where the resource is defined. In this case the resource is defined in the cookbook "myface" in the "default" recipe at line 20.

You might have noticed instead of creating a role and populating the run_list with "tomcat" and "myface" we actually used `include_recipe` to magically build and properly order our run_list. You've probably been told to use a role to describe what your application servers look like. You've seriously been mislead.

## include_recipe versus a Role with a run_list

The great thing about Chef is that it provides you with a collection of primitives to accomplish any configuration task. But like Uncle Ben told Peter Parker, "With great power comes great responsibility". As a cookbook author you are able to combine these primitives in various ways to accomplish the same task. In Chef there is no wrong way to do something, only better ways.

In the previous section we used `include_recipe` to install Tomcat prior to deploying our application. We could have achieved this by creating a role that builds a run_list with these recipes

    name "myface_appserver"
    description "Configures a node to be a Myface application server"
    run_list(
      "recipe[tomcat]",
      "recipe[myface]"
    )

This would accomplish the same task of configuring a node with Tomcat and deploying Myface to it, right? Except a role is not packaged with a cookbook. You could easily update your README and include a strongly worded note instructing the operator of your cookbook to create a role and place it into their Chef Repository, but why make them go through the trouble?

What if the operator already has a role named "myface_appserver" or more likely we named the role "appserver" and the operator had one of those already. Well you could just tell the operator to name it whatever he wants until you have logic in your recipe that requires the role to be named what you expect. Take this search for example where we dynamically find all of our application servers

    search(:node, 'role:myface_appserver')

This would cause our recipe to entirely break and should be avoided. A role is domain logic used by operators to configure nodes to their liking and should not be used by cookbook authors to configure the dependencies of their cookbooks.

You could also suggest to a user to include the Tomcat recipe in their nodes run_list along with the "myface" recipe. What if the operator was to put the "myface" recipe first before "tomcat"? Take for example this node JSON

    {
      "name": "proving-ground",
      "chef_environment": "reset-development",
      "run_list": [
        "recipe[myface]",
        "recipe[tomcat]"
      ]
    }

Well... nothing actually. The order of execution doesn't matter for Myface just yet. Tomcat can be installed at any point and things will be just fine. In the next section we will wire up our application into Tomcat and this will change. After every deployment Tomcat will restart to pick up newly deployed artifacts of Myface. This will seemingly work fine since we've been building our virtual machine up as we go. However, if we were to destroy and rebuild our virtual machine we would get a FATAL error telling us that the Tomcat service could not be restarted. This is because Tomcat wouldn't have been installed to the system yet.

Cookbooks should have entry points exposed to operators and these entry points should be well documented. The default recipe is a good place to start and should be your default entry point. These entry points need to be self sufficient and responsible for doing the job they advertise and nothing more. It is unacceptable to require an operator to build a role or manually construct a run_list for their nodes.

# Wiring up Tomcat

Since we're developing a web app and running it in Tomcat we're going to need to know the URL to put into our browser to check out Myface. Tomcat doesn't typically run on port 80 so we're going to need to figure out what port Tomcat is running on. It's a good practice to make port or memory settings for your applications into tunable attributes, and lucky for us, the fine folks over at Opscode who developed this Tomcat cookbook follow such best practices.

If we check the [Tomcat cookbook's documentation](https://github.com/opscode-cookbooks/tomcat/blob/master/README.md) we see that there is an _Attributes_ section that lists some of the important attributes and what they do. This information is also readily available by reading the [attributes file](https://github.com/opscode-cookbooks/tomcat/blob/master/attributes/default.rb) of the Tomcat cookbook. By reading the documentation we see that Tomcat will be configured to run on port `8080` by default.

Now we need to know what ipaddress or hostname to put into our browser to go along with that port. Since Tomcat is running within a virtual machine we need a way to address the VM on port 8080. This can be achieved by setting up port forwarding in Vagrant.

## Forwarding ports to Vagrant

To access Tomcat on port `8080` we'll need to open our Vagrantfile for editing and add a forwarded port option for the virtual machine

    Vagrant::Config.run do |config|
      ...

      # Forward a port from the guest to the host, which allows for outside
      # computers to access the VM, whereas host only networking does not.
      config.vm.network :forwarded_port, guest: 8080, host: 9090

      ...
      end
    end

And then we need to reload our virtual machine. note: A provision will not suffice, the virtual machine must be stopped and started or reloaded.

    $ bundle exec vagrant reload
    [default] Attempting graceful shutdown of VM...
    [default] Clearing any previously set forwarded ports...
    [default] Forwarding ports...
    [default] -- 22 => 2222 (adapter 1)
    [default] -- 8080 => 9090 (adapter 1)
    [default] Creating shared folders metadata...
    [default] Clearing any previously set network interfaces...
    [default] Preparing network interfaces based on configuration...
    [default] Booting VM...
    [default] Waiting for VM to boot. This can take a few minutes.
    [default] VM booted and ready for use!
    [default] Configuring and enabling network interfaces...
    [default] Mounting shared folders...
    [default] -- v-root: /vagrant
    [default] -- v-csc-1: /tmp/vagrant-chef-1/chef-solo-1/cookbooks
    [default] Running provisioner: Vagrant::Provisioners::ChefSolo...
    [default] Generating chef JSON and uploading...
    [default] Running chef-solo...
    ...
    [Thu, 26 Jul 2012 02:03:28 +0000] INFO: Chef Run complete in 2.449577932 seconds
    [Thu, 26 Jul 2012 02:03:28 +0000] INFO: Running report handlers
    [Thu, 26 Jul 2012 02:03:28 +0000] INFO: Report handlers complete

If you check the output from Vagrant you'll see the forwarding ports section now has an entry for 9090 on your local machine to 8080 on the guest machine.

    [default] Forwarding ports...
    [default] -- 22 => 2222 (adapter 1)
    [default] -- 8080 => 9090 (adapter 1)

Now you should be able to access the [Tomcat manager](http://localhost:9090/manager/html/) but you'll be greeted with an authorization box instead of some Tomcat managing goodness. In the next section we'll cover creating a Tomcat user and role to access the Tomcat manager using the users recipe provided by the Tomcat cookbook.

_note: You could forward 8080 on your host machine to 8080 on the virtual machine but there is a good chance if you're a Java developer that you are starting up Tomcats on 8080 already on your host machine._

_note: If you are not using the virtual box provided by this tutorial you may have iptables, selinux, or another software firewall running that may block connections coming into your virtual machine._

## Configuring Tomcat users

In order to access the Tomcat manager you'll need to configure a user with the "manager" role. The Tomcat cookbook provides the `tomcat::users` recipe and some documentation on how to manage Tomcat's users and roles. If using Chef Solo the `tomcat::users` recipe requires you to create a data bag called `tomcat_users` and populate it with data bag items for each user you wish to create. If using Chef Client with Chef Server you'll need to go an additional step and make sure that all of the data bag items are encrypted.

Since we're using Chef Solo let's start by creating the `tomcat_users` data bag. This is accomplished by creating a directory of the same name within a `data_bags` directory within our cookbook

    $ mkdir -p data_bags/tomcat_users

This directory will hold the data bag items representing our Tomcat users. Let's create a data bag item for the user "tomcat"

    $ touch data_bags/tomcat_users/tomcat.json

Open this file in your favorite editor and add the following JSON for the new user

    {
      "id": "tomcat",
      "password": "tomcat",
      "roles": [
        "manager"
      ]
    }

Now that we have our data bag and data bag item created we need to tell Chef to create the Tomcat users. This is done by including the `tomcat::users` recipe into your recipe. Earlier we told Chef to install Tomcat by including the default Tomcat recipe in Myface's default recipe.

Open the default recipe for editing at `myface/recipes/default.rb` and include the `tomcat::users` recipe

    #
    # Cookbook Name:: myface
    # Recipe:: default
    #
    # Copyright (C) 2012 YOUR_NAME
    #
    # All rights reserved - Do Not Redistribute
    #

    include_recipe "tomcat"
    include_recipe "tomcat::users"
    ...

Now re-provision with Vagrant

    $ bundle exec vagrant provision
    [default] Running provisioner: Vagrant::Provisioners::ChefSolo...
    [default] Generating chef JSON and uploading...
    [default] Running chef-solo...
    [Fri, 27 Jul 2012 21:11:28 +0000] INFO: *** Chef 11.6.0 ***
    [Fri, 27 Jul 2012 21:11:29 +0000] INFO: Setting the run_list to ["recipe[myface::default]"] from JSON
    [Fri, 27 Jul 2012 21:11:29 +0000] INFO: Run List is [recipe[myface::default]]
    [Fri, 27 Jul 2012 21:11:29 +0000] INFO: Run List expands to [myface::default]
    [Fri, 27 Jul 2012 21:11:29 +0000] INFO: Starting Chef Run for myface-cookbook-development
    [Fri, 27 Jul 2012 21:11:29 +0000] INFO: Running start handlers
    [Fri, 27 Jul 2012 21:11:29 +0000] INFO: Start handlers complete.
    [Fri, 27 Jul 2012 21:11:29 +0000] ERROR: Running exception handlers
    [Fri, 27 Jul 2012 21:11:29 +0000] ERROR: Exception handlers complete
    [Fri, 27 Jul 2012 21:11:29 +0000] FATAL: Stacktrace dumped to /tmp/vagrant-chef-1/chef-stacktrace.out
    [Fri, 27 Jul 2012 21:11:29 +0000] FATAL: Chef::Exceptions::InvalidDataBagPath: Data bag path '/var/chef/data_bags' is invalid
    Chef never successfully completed! Any errors should be visible in the
    output above. Please fix your recipes so that they properly complete.

Oops! This cryptic error message is telling us that we don't have a valid data bag path in our virtual machine at '/var/chef/data_bags'. This is because a bit earlier we created our data bag and data bag items on our host machine but never told our virtual machine about them. We can solve this problem by telling Vagrant where to find our data bags.

Open the cookbook's Vagrantfile and tell it where to find the data bags

    config.vm.provision :chef_solo do |chef|
      chef.data_bags_path = "data_bags"

      ...
    end

Now reload your virtual machine to have it pick up the changes

    $ bundle exec vagrant reload
    [default] Attempting graceful shutdown of VM...
    [default] Clearing any previously set forwarded ports...
    [default] Forwarding ports...
    [default] -- 22 => 2222 (adapter 1)
    [default] -- 8080 => 9090 (adapter 1)
    [default] Creating shared folders metadata...
    [default] Clearing any previously set network interfaces...
    [default] Preparing network interfaces based on configuration...
    [default] Booting VM...
    [default] Waiting for VM to boot. This can take a few minutes.
    [default] VM booted and ready for use!
    [default] Configuring and enabling network interfaces...
    [default] Setting host name...
    [default] Mounting shared folders...
    [default] -- v-root: /vagrant
    [default] -- v-csc-1: /tmp/vagrant-chef-1/chef-solo-1/cookbooks
    [default] -- v-csdb-2: /tmp/vagrant-chef-1/chef-solo-2/data_bags
    [default] Running provisioner: Vagrant::Provisioners::ChefSolo...
    [default] Generating chef JSON and uploading...
    [default] Running chef-solo...
    [Fri, 27 Jul 2012 21:15:33 +0000] INFO: *** Chef 11.6.0 ***
    [Fri, 27 Jul 2012 21:15:33 +0000] INFO: Setting the run_list to ["recipe[myface::default]"] from JSON
    [Fri, 27 Jul 2012 21:15:33 +0000] INFO: Run List is [recipe[myface::default]]
    [Fri, 27 Jul 2012 21:15:33 +0000] INFO: Run List expands to [myface::default]
    [Fri, 27 Jul 2012 21:15:33 +0000] INFO: Starting Chef Run for myface-cookbook-development
    [Fri, 27 Jul 2012 21:15:33 +0000] INFO: Running start handlers
    [Fri, 27 Jul 2012 21:15:33 +0000] INFO: Start handlers complete.
    [Fri, 27 Jul 2012 21:15:33 +0000] INFO: Processing ruby_block[set-env-java-home] action create (java::openjdk line 36)
    [Fri, 27 Jul 2012 21:15:33 +0000] INFO: ruby_block[set-env-java-home] called
    [Fri, 27 Jul 2012 21:15:33 +0000] INFO: Processing ruby_block[update-java-alternatives] action nothing (java::openjdk line 43)
    [Fri, 27 Jul 2012 21:15:33 +0000] INFO: Processing package[java-1.6.0-openjdk] action install (java::openjdk line 80)
    [Fri, 27 Jul 2012 21:15:36 +0000] INFO: Processing package[java-1.6.0-openjdk-devel] action install (java::openjdk line 80)
    [Fri, 27 Jul 2012 21:15:36 +0000] INFO: Processing package[sun-java6-jdk] action purge (java::default line 25)
    [Fri, 27 Jul 2012 21:15:36 +0000] INFO: Processing package[sun-java6-bin] action purge (java::default line 25)
    [Fri, 27 Jul 2012 21:15:36 +0000] INFO: Processing package[sun-java6-jre] action purge (java::default line 25)
    [Fri, 27 Jul 2012 21:15:36 +0000] INFO: Processing package[tomcat6] action install (tomcat::default line 32)
    [Fri, 27 Jul 2012 21:15:36 +0000] INFO: Processing package[tomcat6-admin-webapps] action install (tomcat::default line 32)
    [Fri, 27 Jul 2012 21:15:36 +0000] INFO: Processing service[tomcat] action enable (tomcat::default line 37)
    [Fri, 27 Jul 2012 21:15:36 +0000] INFO: Processing service[tomcat] action start (tomcat::default line 37)
    [Fri, 27 Jul 2012 21:15:36 +0000] INFO: Processing template[/etc/sysconfig/tomcat6] action create (tomcat::default line 50)
    [Fri, 27 Jul 2012 21:15:36 +0000] INFO: Processing template[/etc/tomcat6/server.xml] action create (tomcat::default line 67)
    [Fri, 27 Jul 2012 21:15:36 +0000] INFO: Processing template[/etc/tomcat6/tomcat-users.xml] action create (tomcat::users line 22)
    [Fri, 27 Jul 2012 21:15:36 +0000] INFO: template[/etc/tomcat6/tomcat-users.xml] backed up to /var/chef/backup/etc/tomcat6/tomcat-users.xml.chef-20120727211536
    [Fri, 27 Jul 2012 21:15:36 +0000] INFO: template[/etc/tomcat6/tomcat-users.xml] mode changed to 644
    [Fri, 27 Jul 2012 21:15:36 +0000] INFO: template[/etc/tomcat6/tomcat-users.xml] updated content
    [Fri, 27 Jul 2012 21:15:36 +0000] INFO: Processing group[myface] action create (myface::default line 13)
    [Fri, 27 Jul 2012 21:15:36 +0000] INFO: Processing user[myface] action create (myface::default line 15)
    [Fri, 27 Jul 2012 21:15:36 +0000] INFO: Processing artifact_deploy[myface] action deploy (myface::default line 21)
    [Fri, 27 Jul 2012 21:15:36 +0000] INFO: template[/etc/tomcat6/tomcat-users.xml] sending restart action to service[tomcat] (delayed)
    [Fri, 27 Jul 2012 21:15:36 +0000] INFO: Processing service[tomcat] action restart (tomcat::default line 37)
    [Fri, 27 Jul 2012 21:15:38 +0000] INFO: service[tomcat] restarted
    [Fri, 27 Jul 2012 21:15:38 +0000] INFO: Chef Run complete in 5.159478213 seconds
    [Fri, 27 Jul 2012 21:15:38 +0000] INFO: Running report handlers
    [Fri, 27 Jul 2012 21:15:38 +0000] INFO: Report handlers complete

If you inspect the output from Vagrant you'll see that the data bags path that you specified is now mounted within the virtual machine

    ...
    [default] Mounting shared folders...
    [default] -- v-root: /vagrant
    [default] -- v-csc-1: /tmp/vagrant-chef-1/chef-solo-1/cookbooks
    [default] -- v-csdb-2: /tmp/vagrant-chef-1/chef-solo-2/data_bags
    ...

And if we check the `tomcat-users.xml` file it should have the entries for the user and role we wanted

    $ bundle exec vagrant ssh -c "cat /etc/tomcat6/tomcat-users.xml"
    <tomcat-users>
    <role rolename="manager" />
    <user username="tomcat" password="tomcat" roles="manager" />
    </tomcat-users>

Tomcat was also notified to restart when the `tomcat-users.xml` was changed so now when we go to the [Tomcat manager](http://localhost:9090/manager/html/) and enter our user "tomcat" with the password "tomcat" we should be granted access and see two running applications for the Tomcat Manager Application. Success!

# Hooking the application into Tomcat

With Tomcat running and access to the Tomcat Manager we can now hook our application into Tomcat. This is going to be pretty easy since we've already got our application deployed to the virtual machine and Tomcat is installed and running. All we have to do is tell our recipe to link our application into the webapps directory of Tomcat.

Open the default recipe `myface/recipes/default.rb` and add a [link resource](http://wiki.opscode.com/display/chef/Resources#Resources-Link) below the artifact_deploy

    ...
    artifact_deploy "myface" do
      version "1.0.0"
      artifact_location "http://dl.dropbox.com/u/31081437/myface-1.0.0.tar.gz"
      deploy_to "/srv/myface"
      owner node[:myface][:user]
      group node[:myface][:group]
      action :deploy
    end

    link "#{node[:tomcat][:home]}/webapps/myface.war" do
      to "/srv/myface/current/myface.war"
    end

_note: In the name attribute of the link resource you'll notice that we actually used an attribute to build part of it. The attribute used was `node[:tomcat][:home]`. If you check the Tomcat documentation you'll see that attribute evaluates to the path on disk for Tomcat's home which contains the webapp directory that we wanted to link our application into. This is a shining example of the power of attributes._

Tomcat will automatically pick up our application and load it after the symlink is written. Let's re-provision our node and check the Tomcat Manager to make sure it all worked

    $ bundle exec vagrant provision
    [default] Running provisioner: Vagrant::Provisioners::ChefSolo...
    [default] Generating chef JSON and uploading...
    [default] Running chef-solo...
    ...
    [Fri, 27 Jul 2012 21:41:39 +0000] INFO: link[/usr/share/tomcat6/webapps/myface.war] created
    [Fri, 27 Jul 2012 21:41:39 +0000] INFO: Chef Run complete in 1.662062569 seconds
    [Fri, 27 Jul 2012 21:41:39 +0000] INFO: Running report handlers
    [Fri, 27 Jul 2012 21:41:39 +0000] INFO: Report handlers complete

And now when we visit the [Tomcat Manager](http://localhost:9090/manager/html) there should be three applications in the applications list. The new application should be mounted at path /myface.

We can now visit our new (and unimpressive) application [in our browser](http://localhost:9090/myface)

# A bit more refactoring

Some duplication crept into Myface's default recipe while we were working on things. Did you notice it?

If you look back at the 'Refactoring into attributes' section you'll recall that when we have common values that are being passed to multiple resources it is a good case to refactor those into attributes. In this case we've been repeating ourself a bit on where our application code should be deployed to. Let's create a new attribute called `myface[:deploy_to]` and put it in our default attributes file.

Open the default attributes file for editing `myface/attributes/default.rb` and add a default attribute for deploy_to

    #
    # Cookbook Name:: myface
    # Attribute:: default
    #
    # Copyright (C) 2012 Jamie Winsor
    #
    # All rights reserved - Do Not Redistribute
    #

    default[:myface][:user] = "myface"
    default[:myface][:group] = "myface"
    default[:myface][:deploy_to] = "/srv/myface"

Now in our default recipe replace the references to the deploy to location with this attribute

    ...
    artifact_deploy "myface" do
      version "1.0.0"
      artifact_location "http://dl.dropbox.com/u/31081437/myface-1.0.0.tar.gz"
      deploy_to node[:myface][:deploy_to]
      owner node[:myface][:user]
      group node[:myface][:group]
      action :deploy
    end

    link "#{node[:tomcat][:home]}/webapps/myface.war" do
      to "#{node[:myface][:deploy_to]}/current/myface.war"
    end

Now let's run the Vagrant provisioner and ensure nothing was changed (It shouldn't have changed since we're writing idempotent recipes, right?).

    $ bundle exec vagrant provision
    [default] Running provisioner: Vagrant::Provisioners::ChefSolo...
    [default] Generating chef JSON and uploading...
    [default] Running chef-solo...
    ...
    [Fri, 27 Jul 2012 21:59:59 +0000] INFO: Processing artifact_deploy[myface] action deploy (myface::default line 21)
    [Fri, 27 Jul 2012 21:59:59 +0000] INFO: Processing link[/usr/share/tomcat6/webapps/myface.war] action create (myface::default line 30)

It's good to review the entire output of Vagrant to ensure no additional work was done, but since we only changed the artifact_deploy and link resource, they are the important bits to check on here. You should only see a Processing log for each resource with no actions taken.

## Configurable artifact version and URL

Since we're already in the spirit of refactoring let's hit the last remaining problem. What if the location of our artifact was to change or we wanted to deploy a different version? Well right now you'd need to make a change to your cookbook. While this doesn't sound like much it is actually a big problem - you'd need to increment the version of your cookbook and re-publish it to the community site or your Chef Server.

You wouldn't hardcode the location of your database into your application code, would you?

This brings up the topic of _Configurable Attributes_ (sometimes referred to as Tunable Attributes or Tunables). Attributes can be overridden on multiple levels to allow an operator to change the behavior of your cookbook without actually making any code changes.

Let's make a configurable attribute for the artifact URL and the artifact version. We will document these in the README to explain to an operator how they can deploy different versions of our application or from different locations without making any cookbook changes.

Start by creating two new default attributes in the default attributes file (`myface/attributes/default.rb`)

    #
    # Cookbook Name:: myface
    # Attribute:: default
    #
    # Copyright (C) 2012 Jamie Winsor
    #
    # All rights reserved - Do Not Redistribute
    #

    ...
    default[:myface][:artifact_url] = "http://dl.dropbox.com/u/31081437/myface-1.0.0.tar.gz"
    default[:myface][:artifact_version] = "1.0.0"

And just like we did for the deploy_to attribute, replace the values in the default recipe with these new attributes

    #
    # Cookbook Name:: myface
    # Recipe:: default
    #
    # Copyright (C) 2012 YOUR_NAME
    #
    # All rights reserved - Do Not Redistribute
    #

    ...
    artifact_deploy "myface" do
      version node[:myface][:artifact_version]
      artifact_location node[:myface][:artifact_url]
      deploy_to node[:myface][:deploy_to]
      owner node[:myface][:user]
      group node[:myface][:group]
      action :deploy
    end

And if we run our Vagrant provisioner we should see no changes

    $ bundle exec vagrant provision
    [default] Running provisioner: Vagrant::Provisioners::ChefSolo...
    [default] Generating chef JSON and uploading...
    [default] Running chef-solo...
    ...
    [Fri, 27 Jul 2012 22:23:19 +0000] INFO: Processing artifact_deploy[myface] action deploy (myface::default line 21)
    ...
    [Fri, 27 Jul 2012 22:23:19 +0000] INFO: Chef Run complete in 1.610049321 seconds
    [Fri, 27 Jul 2012 22:23:19 +0000] INFO: Running report handlers
    [Fri, 27 Jul 2012 22:23:19 +0000] INFO: Report handlers complete

# Configuring the database server

Version 1.0.0 of Myface is just a web application that is serving up a static page - it doesn't do much. Well version 2.0.0 of Myface is ready and it requires a persistent database to hold account information and the hordes of cats that your users are going to be posting. We're going to cover installing and configuring MySQL in this section but the steps are similar enough if your application requires another persistent datastore like PostgreSQL.

## Creating the database recipe

We're going to create a new recipe and call it 'database'. It's a solid best practice to separate your applications components into their own recipes for a few reasons.

* It's easier to develop and test components when they are separate. This is a good time to bring up the [Single Responsibility Principle (SRP)](http://en.wikipedia.org/wiki/Single_responsibility_principle). Each recipe should do the job of installing or configuring a single component - nothing more, nothing less.
* Enables you to distribute components across different nodes without including unnecessary components.

Start out by creating a new recipe called 'database'

    $ touch recipes/database.rb

And include the MySQL cookbooks's server recipe.

    #
    # Cookbook Name:: myface
    # Recipe:: database
    #
    # Copyright (C) 2012 YOUR_NAME
    #
    # All rights reserved - Do Not Redistribute
    #

    include_recipe "mysql::server"

To test our work out we'll want to add this new recipe to the `run_list` of our virtual machine. This is done by editing the Vagrantfile and adding the `myface::database` recipe to the array of recipes in the run_list. You should place the database as the first recipe in the run_list since it's common for the application server to require a database to connect to before it would successfully start.

Open the Vagrantfile and add the `myface::database` recipe to index 0 of the run_list

    config.vm.provision :chef_solo do |chef|
      ...

      chef.run_list = [
        "recipe[myface::database]",
        "recipe[myface::default]"
      ]
    end

While we're in the Vagrantfile we should also add the root password for MySQL so it isn't automatically generated for us and then lost. At the time of writing Berkshelf automatically populates these attributes for you as a nice example for how to override attributes. Re-open the Vagrantfile and add the attributes

    config.vm.provision :chef_solo do |chef|
      ...

      chef.json = {
        :mysql => {
          :server_root_password => 'rootpass',
          :server_debian_password => 'debpass',
          :server_repl_password => 'replpass'
        }
      }

      ...
    end

These attributes are documented in the [README for the MySQL cookbook](https://github.com/RiotGames/mysql-cookbook/blob/master/README.md).

We are nearly there but something is missing. Can you find it? Re-run the Vagrant provisioner and it would immediately become clear

    $ bundle exec vagrant provision
    ...
    [Sat, 28 Jul 2012 02:15:50 +0000] FATAL: Stacktrace dumped to /tmp/vagrant-chef-1/chef-stacktrace.out
    [Sat, 28 Jul 2012 02:15:50 +0000] FATAL: ArgumentError: Cannot find a recipe matching database in cookbook myface

There is still one more problem. Let's re-run the Vagrant provisioner and see what's left

    $ bundle exec vagrant provision
    ...
    [Sat, 28 Jul 2012 02:19:11 +0000] FATAL: Stacktrace dumped to /tmp/vagrant-chef-1/chef-stacktrace.out
    [Sat, 28 Jul 2012 02:19:11 +0000] FATAL: Chef::Exceptions::CookbookNotFound: Cookbook mysql not found. If you're loading mysql from another cookbook, make sure you configure the dependency in your metadata

The output from Vagrant includes a well written error message telling you that if you're loading MySQL from another cookbook you should also configure the dependency in your metadata.

Open up the `metadata.rb` file for editing and add "mysql" as a dependency

    name             "myface"
    maintainer       "YOUR_NAME"
    maintainer_email "YOUR_EMAIL"
    license          "All rights reserved"
    description      "Installs/Configures myface"
    long_description IO.read(File.join(File.dirname(__FILE__), 'README.md'))
    version          "0.0.1"

    depends "artifact", "~> 0.10.1"
    depends "tomcat", "~> 0.11.0"
    depends "mysql", "~> 1.3.0"

And if we re-run the Vagrant provisioner MySQL will now successfully be installed

    $ bundle exec vagrant provision
    [Sat, 28 Jul 2012 02:40:22 +0000] INFO: *** Chef 10.14.2 ***
    [Sat, 28 Jul 2012 02:40:23 +0000] INFO: Setting the run_list to ["recipe[myface::database]", "recipe[myface::default]"] from JSON
    [Sat, 28 Jul 2012 02:40:23 +0000] INFO: Run List is [recipe[myface::database], recipe[myface::default]]
    [Sat, 28 Jul 2012 02:40:23 +0000] INFO: Run List expands to [myface::database, myface::default]
    ...
    [Sat, 28 Jul 2012 02:40:24 +0000] INFO: Processing chef_gem[mysql] action install (mysql::client line 59)
    [Sat, 28 Jul 2012 02:40:28 +0000] INFO: chef_gem[mysql] installed version 2.8.1
    [Sat, 28 Jul 2012 02:40:28 +0000] INFO: ruby_block[install the mysql chef_gem at run time] called
    [Sat, 28 Jul 2012 02:40:28 +0000] INFO: Processing package[mysql-server] action install (mysql::server line 77)
    [Sat, 28 Jul 2012 02:40:28 +0000] INFO: package[mysql-server] installing mysql-server-5.1.61-4.el6 from base repository
    [Sat, 28 Jul 2012 02:40:33 +0000] INFO: package[mysql-server] installed version 5.1.61-4.el6
    [Sat, 28 Jul 2012 02:40:33 +0000] INFO: Processing directory[/etc/mysql/conf.d] action create (mysql::server line 84)
    [Sat, 28 Jul 2012 02:40:33 +0000] INFO: directory[/etc/mysql/conf.d] created directory /etc/mysql/conf.d
    [Sat, 28 Jul 2012 02:40:33 +0000] INFO: directory[/etc/mysql/conf.d] owner changed to 27
    [Sat, 28 Jul 2012 02:40:33 +0000] INFO: directory[/etc/mysql/conf.d] group changed to 27
    [Sat, 28 Jul 2012 02:40:33 +0000] INFO: Processing service[mysql] action nothing (mysql::server line 104)
    [Sat, 28 Jul 2012 02:40:33 +0000] INFO: Processing template[/etc/my.cnf] action create (mysql::server line 124)
    [Sat, 28 Jul 2012 02:40:33 +0000] INFO: template[/etc/my.cnf] backed up to /var/chef/backup/etc/my.cnf.chef-20120728024033
    [Sat, 28 Jul 2012 02:40:33 +0000] INFO: template[/etc/my.cnf] mode changed to 644
    [Sat, 28 Jul 2012 02:40:33 +0000] INFO: template[/etc/my.cnf] updated content
    [Sat, 28 Jul 2012 02:40:33 +0000] INFO: template[/etc/my.cnf] sending restart action to service[mysql] (immediate)
    [Sat, 28 Jul 2012 02:40:33 +0000] INFO: Processing service[mysql] action restart (mysql::server line 104)
    [Sat, 28 Jul 2012 02:40:35 +0000] INFO: service[mysql] restarted
    [Sat, 28 Jul 2012 02:40:35 +0000] INFO: Processing execute[assign-root-password] action run (mysql::server line 147)
    [Sat, 28 Jul 2012 02:40:35 +0000] INFO: execute[assign-root-password] ran successfully
    [Sat, 28 Jul 2012 02:40:35 +0000] INFO: Processing template[/etc/mysql_grants.sql] action create (mysql::server line 171)
    [Sat, 28 Jul 2012 02:40:35 +0000] INFO: template[/etc/mysql_grants.sql] updated content
    [Sat, 28 Jul 2012 02:40:35 +0000] INFO: template[/etc/mysql_grants.sql] sending run action to execute[mysql-install-privileges] (immediate)
    [Sat, 28 Jul 2012 02:40:35 +0000] INFO: Processing execute[mysql-install-privileges] action run (mysql::server line 187)
    [Sat, 28 Jul 2012 02:40:35 +0000] INFO: execute[mysql-install-privileges] ran successfully
    [Sat, 28 Jul 2012 02:40:35 +0000] INFO: Processing execute[mysql-install-privileges] action nothing (mysql::server line 187)
    ...
    [Sat, 28 Jul 2012 02:40:35 +0000] INFO: Chef Run complete in 12.018921229 seconds
    [Sat, 28 Jul 2012 02:40:35 +0000] INFO: Running report handlers
    [Sat, 28 Jul 2012 02:40:35 +0000] INFO: Report handlers complete

Well we have MySQL installed but we don't have a database. In the next section we'll create a database and users to go with it that have proper permissions.

## Creating a database with the Database cookbook

Opscode provides an amazing utility cookbook that exposes a number of Light-weight Resource Providers (LWRP) for manipulating persistent databases like MySQL and PostgreSQL. To get access to these LWRPs all we need to do is include the 'database' cookbook in the metadata of our own.

Add the 'database' cookbook as a dependency in our `metadata.rb` file

  depends "database", "~> 1.3.4"

Because the database cookbook has it's own requirement for MySQL we can also go ahead and remove our dependency on the MySQL cookbook. We will inherit the dependency by the database cookbook since it has it's own constraint defined.

Your `metadata.rb` file should now look like

    name             "myface"
    maintainer       "YOUR_NAME"
    maintainer_email "YOUR_EMAIL"
    license          "All rights reserved"
    description      "Installs/Configures myface"
    long_description IO.read(File.join(File.dirname(__FILE__), 'README.md'))
    version          "0.0.1"

    depends "artifact", "~> 0.10.1"
    depends "tomcat", "~> 0.11.0"
    depends "mysql", "~> 1.3.0"
    depends "database", "~> 1.3.4"

Now in the database recipe (`recipes/database.rb`) let's describe a database to be created for our application

    mysql_database "myface_dev" do
      connection(
        :host => "localhost",
        :username => "root",
        :password => node[:mysql][:server_root_password]
      )
      action :create
    end

This resource will create a database in MySQL using the hash passed to the connection resource attribute. This hash is equivalent to what you might pass to mysqladmin on the command line. Remember that attribute that we explicitly set in the Vagrantfile for the MySQL root password? Here you see it again (`node[:mysql][:server_root_password]`) but this time we're reading it and passing it to MySQL for authorization.

Now re-run the Vagrant provisioner to create the database

    $ bundle exec vagrant provision
    ...
    [Thu, 16 Aug 2012 23:25:45 +0000] INFO: Processing mysql_database[myface_dev] action create (myface::database line 17)
    [Thu, 16 Aug 2012 23:25:45 +0000] ERROR: Running exception handlers
    [Thu, 16 Aug 2012 23:25:45 +0000] ERROR: Exception handlers complete
    [Thu, 16 Aug 2012 23:25:45 +0000] FATAL: Stacktrace dumped to /tmp/vagrant-chef-1/chef-stacktrace.out
    [Thu, 16 Aug 2012 23:25:45 +0000] FATAL: LoadError: no such file to load -- mysql

Well that's not good: another cryptic FATAL error. This message is actually coming from Ruby itself. If you are a Ruby developer coming to Chef you've probably seen this message before. "no such file to load -- {file}" is the error message from a LoadError. When you attempt to use `require` with a gem that is not located in your load path this error is thrown.

Here we are getting a load error for the MySQL gem because we didn't read the README for the database cookbook (or you can blame me since I didn't tell you about it). The README explains to use the MySQL LWRP's you need to include the `database::mysql` recipe in your recipe before any MySQL LWRPs are processed. Let's go ahead and include that recipe in our database recipe to get this error patched up.

    include_recipe "mysql::server"
    include_recipe "database::mysql"

    mysql_database "myface_dev" do
      connection(
        :host => "localhost",
        :username => "root",
        :password => node[:mysql][:server_root_password]
      )
      action :create
    end

Now if we re-run the Vagrant provisioner the database will be created

    $ bundle exec vagrant provision
    [default] Running provisioner: Vagrant::Provisioners::ChefSolo...
    [default] Generating chef JSON and uploading...
    [default] Running chef-solo...
    ...
    [Sat, 28 Jul 2012 03:33:45 +0000] INFO: Processing mysql_database[myface_dev] action create (myface::database line 12)
    ...
    [Sat, 28 Jul 2012 03:33:45 +0000] INFO: Chef Run complete in 3.253535476 seconds
    [Sat, 28 Jul 2012 03:33:45 +0000] INFO: Running report handlers
    [Sat, 28 Jul 2012 03:33:45 +0000] INFO: Report handlers complete

If you don't believe the database has been created, check for yourself!

    $ bundle exec vagrant ssh
    vagrant> mysql -u root -p -e "show databases;"
    +--------------------+
    | Database           |
    +--------------------+
    | information_schema |
    | myface_dev         |
    | mysql              |
    | test               |
    +--------------------+

See the [Database Cookbook's README](https://github.com/opscode-cookbooks/database/blob/master/README.md) for full documentation regarding creating databases.

## Creating a MySQL user with the Database cookbook

It's a good practice to create a user in MySQL with strict permissions for each one of your applications. Ideally this user will only have the ability to only manipulate the application's database and have no MySQL administrative privileges. Let's create a user for Myface and call it "myface_app".

Open the database recipe (`recipes/database.rb`) and describe the new MySQL database user

    mysql_database_user "myface_app" do
      connection(
        :host => "localhost",
        :username => "root",
        :password => node[:mysql][:server_root_password]
      )
      password "supersecret"
      database_name "myface_dev"
      host "localhost"
      action [:create, :grant]
    end

When we re-run the Vagrant provisioner the user will be created

    $ bundle exec vagrant provision
    [default] Running provisioner: Vagrant::Provisioners::ChefSolo...
    [default] Generating chef JSON and uploading...
    [default] Running chef-solo...
    ...
    [Sat, 28 Jul 2012 03:51:59 +0000] INFO: mysql_database_user[myface_app]: granting access with statement [GRANT all ON myface_dev.* TO 'myface_app'@'localhost' IDENTIFIED BY 'supersecret']
    ...
    [Sat, 28 Jul 2012 03:51:59 +0000] INFO: Chef Run complete in 3.258085474 seconds
    [Sat, 28 Jul 2012 03:51:59 +0000] INFO: Running report handlers
    [Sat, 28 Jul 2012 03:51:59 +0000] INFO: Report handlers complete

_note: It's worth noting that the password for your database was just output to the console and to a log on disk by Chef. As far as I am concerned this is a bug but it hasn't been addressed in the database cookbook yet. Please ensure in production that you destroy this log and clear your console anytime this resource changes state. Or maybe I'm just too paranoid... thanks [Pat](http://www.codeofhonor.com/blog/)._

## Reopening resource definitions: Making MySQL available on boot

You would get a pretty nasty message right now if you were to reload your virtual machine or something sudden happened where the machine was shut down and you powered it back up. It would look something like this

    [Tue, 31 Jul 2012 00:25:58 +0000] ERROR: Running exception handlers
    [Tue, 31 Jul 2012 00:25:58 +0000] ERROR: Exception handlers complete
    [Tue, 31 Jul 2012 00:25:58 +0000] FATAL: Stacktrace dumped to /tmp/vagrant-chef-1/chef-stacktrace.out
    [Tue, 31 Jul 2012 00:25:58 +0000] FATAL: Mysql::Error: mysql_database[myface_dev] (myface::database line 12) had an error: Mysql::Error: Can't connect to local MySQL server through socket '/var/lib/mysql/mysql.sock' (111)

The database couldn't be created (or it's existence verified) because MySQL wasn't started and listening on it's sock. This is because the MySQL cookbook does not explicitly enable MySQL to be started after each boot. Generally when we start a machine up we want it to come up in it's "best" state.

Open the `recipes/database.rb` recipe and re-open the [service resource](http://wiki.opscode.com/display/chef/Resources#Resources-Service) defining the MySQL service and make sure it's enabled

    include_recipe "mysql::server"

    service "mysql" do
      action :enable
    end

    ...

The placement of this resource definition is __very important__. It is placed after the inclusion of the mysql server recipe because mysql needs to be installed before the service is available to interact with. This is similar to how you open a class or module in Ruby for modifications.

Run the vagrant provisioner to pickup the changes

    $ bundle exec vagrant provision

# Configurable Attributes Strike Back

When we wrote the database recipe the database name, user, and the user's password were all hardcoded directly into the recipe. This is a bad pattern for a few reasons.

Security. Chef attributes are indexed and are publicly available to all API clients. This means that your database credentials are easy found if your Chef server is compromised. And, since the database is nicely marked with the recipe named "myface::database", the hostname and ipaddress are effortlessly found by a potential attacker.

Keeping your code DRY. It's easy to make mistakes as a cookbook author when things that could be extracted into configurable attributes are littered throughout multiple recipes. An author needs to make good use of his editor's find and replace function if he hopes to reconfigure his application properly.

In a previous section the artifact URL and version were refactored into attributes. This is an excellent way to store non-sensitive data and let's us keep our code nice and DRY, but it's a poor choice when storing passwords or other sensitive data. Chef provides you with the [Encrypted Data Bags](http://wiki.opscode.com/display/chef/Encrypted+Data+Bags) primitive for such storage needs.

## Extracting database connection info into an Encrypted Data Bag

## Abstracting In a previous section we abstracted

# Refactoring default recipe into application recipe

# Incrementing versions

# Running lint tests

    $ bundle exec thor foodcritic:lint

# Building your VM with Chef Client instead of Chef Solo
