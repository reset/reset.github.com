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

Ruby 1.9 is a requirement of Berkshelf. We will be using `1.9.3-p194` which is the latest patch level of 1.9.3.

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
             run  git init from "./myface"
    Initialized empty Git repository in /Users/reset/code/berkshelf/myface/.git/
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

This will install Vagrant, Berkshelf, and thor-foodcritic. We will be using Vagrant for building our virtual environment, Berkshelf for managing our cookbook dependencies, and thor-foodcritic to lint test our cookbook.

A `Vagrantfile` was generated for you with a boilerplate configuration that should be suitable for our needs. The Vagrantfile is configured to download and boot a CentOS 6.3 Vagrant Box and provision it with `chef-solo`. I recommend sticking with these defaults while you are working through this guide.

Start up your virtual machine

    $ bundle exec vagrant up
    [default] Importing base box 'Berkshelf-CentOS-6.3-x86_64-minimal'...
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
    [Mon, 23 Jul 2012 20:16:30 +0000] INFO: *** Chef 10.12.0 ***
    [Mon, 23 Jul 2012 20:16:30 +0000] INFO: Setting the run_list to ["recipe[myface::default]"] from JSON
    [Mon, 23 Jul 2012 20:16:30 +0000] INFO: Run List is [recipe[myface::default]]
    [Mon, 23 Jul 2012 20:16:30 +0000] INFO: Run List expands to [myface::default]
    [Mon, 23 Jul 2012 20:16:30 +0000] INFO: Starting Chef Run for localhost
    [Mon, 23 Jul 2012 20:16:30 +0000] INFO: Running start handlers
    [Mon, 23 Jul 2012 20:16:30 +0000] INFO: Start handlers complete.
    [Mon, 23 Jul 2012 20:16:30 +0000] INFO: Chef Run complete in 0.020575978 seconds
    [Mon, 23 Jul 2012 20:16:30 +0000] INFO: Running report handlers
    [Mon, 23 Jul 2012 20:16:30 +0000] INFO: Report handlers complete

The default CentOS 6.3 Vagrant Box can be swapped with the OS of your choosing by opening the `Vagrantfile` inside your cookbook with your [favorite editor](http://www.sublimetext.com/2) and editting the values of the `config.vm.box` and `config.vm.box_url` attributes.

    Vagrant::Config.run do |config|
      ...

      config.vm.box = "Berkshelf-CentOS-6.3-x86_64-minimal"
      config.vm.box_url = "https://dl.dropbox.com/u/31081437/Berkshelf-CentOS-6.3-x86_64-minimal.box"

      ...
    end

Check the full [Vagrant Documentation](http://vagrantup.com/v1/docs/index.html) for future reference.

If at anytime your virtual machine becomes unstable or if you'd like to start over you can destroy your virtual machine with one command

    $ bundle exec vagrant destroy

# Deploying with Artifact Deploy

Now that we've got a barebones cookbook for our hot new social networking application, myface, let's make our first cookbook change and deploy our application with Artifact Deploy. Artifact Deploy is a Light-weight Resource and Provider (LWRP) that comes bundled with the [Artifact Cookbook](https://github.com/riotgames/artifact-cookbook).

In Chef, a resource represents a piece of system state and a provider is the underlying implementation which brings the resource into the desired state. Chef comes with a number of Resources and Providers for you to use out of the box but you can create your own by generating a cookbook that contains an LWRP and including it into the metadata of a another cookbook.

Open the default recipe for editing at `myface/recipes/default.rb` and add following code block.

    artifact_deploy "myface" do
      version "1.0.0"
      artifact_location "http://dl.dropbox.com/u/31081437/HelloWebApp.war"
      deploy_to "/srv/myface"
      owner "myface"
      group "myface"
      action :deploy
    end

Save your work.

Next we will re-provision your virtual machine by running Vagrant's __provision__command. Provision will re-run the Chef-Solo provisioner. This is the same provisioner that ran earlier when we started our virtual machine. Vagrant will automatically pick up the changes that you made in the default recipe and attempt to provision the node with these changes. Vagrant is able to automatically receive these updates because a path on your host machine is automatically mounted into the virtual machine when you run `vagrant up`. Berkshelf populates this path with shims pointing to the actual contents of the cookbooks managed by Berkshelf.

    $ bundle exec vagrant provision

You should have experienced a failure in the provisioning

    [default] Running provisioner: Vagrant::Provisioners::ChefSolo...
    [default] Generating chef JSON and uploading...
    [default] Running chef-solo...
    [Mon, 23 Jul 2012 20:48:53 +0000] INFO: *** Chef 10.12.0 ***
    [Mon, 23 Jul 2012 20:48:53 +0000] INFO: Setting the run_list to ["recipe[myface::default]"] from JSON
    [Mon, 23 Jul 2012 20:48:53 +0000] INFO: Run List is [recipe[myface::default]]
    [Mon, 23 Jul 2012 20:48:53 +0000] INFO: Run List expands to [myface::default]
    [Mon, 23 Jul 2012 20:48:53 +0000] INFO: Starting Chef Run for localhost
    [Mon, 23 Jul 2012 20:48:53 +0000] INFO: Running start handlers
    [Mon, 23 Jul 2012 20:48:53 +0000] INFO: Start handlers complete.
    [Mon, 23 Jul 2012 20:48:53 +0000] ERROR: Running exception handlers
    [Mon, 23 Jul 2012 20:48:53 +0000] ERROR: Exception handlers complete
    [Mon, 23 Jul 2012 20:48:53 +0000] FATAL: Stacktrace dumped to /tmp/vagrant-chef-1/chef-stacktrace.out
    [Mon, 23 Jul 2012 20:48:53 +0000] FATAL: NameError: Cannot find a resource for artifact_deploy on centos version 6.3
    Chef never successfully completed! Any errors should be visible in the
    output above. Please fix your recipes so that they properly complete.

The important bit here is in the lines prefaced with FATAL.

    [Mon, 23 Jul 2012 20:48:53 +0000] FATAL: Stacktrace dumped to /tmp/vagrant-chef-1/chef-stacktrace.out
    [Mon, 23 Jul 2012 20:48:53 +0000] FATAL: NameError: Cannot find a resource for artifact_deploy on centos version 6.3    

When inspecting a Chef stacktrace the source of your problem is generally marked by FATAL. In this case the resource artifact_deploy cannot be found. If we inspected the stacktrace that was dumped to `/tmp/vagrant-chef-1/chef-stacktrace.out` in our virtual machine we'll see exactly which line caused this fatal error.

    $ bundle exec vagrant ssh
    [vagrant@localhost ~]$ cat /tmp/vagrant-chef-1/chef-stacktrace.out
    Generated at 2012-07-23 20:48:53 +0000
    NameError: Cannot find a resource for artifact_deploy on centos version 6.3
    /opt/chef/embedded/lib/ruby/gems/1.9.1/gems/chef-10.12.0/lib/chef/resource_platform_map.rb:126:in `get'
    /opt/chef/embedded/lib/ruby/gems/1.9.1/gems/chef-10.12.0/lib/chef/resource.rb:667:in `resource_for_platform'
    /opt/chef/embedded/lib/ruby/gems/1.9.1/gems/chef-10.12.0/lib/chef/resource.rb:684:in `resource_for_node'
    /opt/chef/embedded/lib/ruby/gems/1.9.1/gems/chef-10.12.0/lib/chef/mixin/recipe_definition_dsl_core.rb:58:in `method_missing'
    /tmp/vagrant-chef-1/chef-solo-1/cookbooks/myface/recipes/default.rb:10:in `from_file'
    /opt/chef/embedded/lib/ruby/gems/1.9.1/gems/chef-10.12.0/lib/chef/mixin/from_file.rb:30:in `instance_eval'
    /opt/chef/embedded/lib/ruby/gems/1.9.1/gems/chef-10.12.0/lib/chef/mixin/from_file.rb:30:in `from_file'
    /opt/chef/embedded/lib/ruby/gems/1.9.1/gems/chef-10.12.0/lib/chef/cookbook_version.rb:578:in `load_recipe'
    /opt/chef/embedded/lib/ruby/gems/1.9.1/gems/chef-10.12.0/lib/chef/mixin/language_include_recipe.rb:46:in `load_recipe'
    /opt/chef/embedded/lib/ruby/gems/1.9.1/gems/chef-10.12.0/lib/chef/mixin/language_include_recipe.rb:33:in `block in include_recipe'
    /opt/chef/embedded/lib/ruby/gems/1.9.1/gems/chef-10.12.0/lib/chef/mixin/language_include_recipe.rb:27:in `each'
    /opt/chef/embedded/lib/ruby/gems/1.9.1/gems/chef-10.12.0/lib/chef/mixin/language_include_recipe.rb:27:in `include_recipe'
    /opt/chef/embedded/lib/ruby/gems/1.9.1/gems/chef-10.12.0/lib/chef/run_context.rb:72:in `block in load'
    /opt/chef/embedded/lib/ruby/gems/1.9.1/gems/chef-10.12.0/lib/chef/run_context.rb:69:in `each'
    /opt/chef/embedded/lib/ruby/gems/1.9.1/gems/chef-10.12.0/lib/chef/run_context.rb:69:in `load'
    /opt/chef/embedded/lib/ruby/gems/1.9.1/gems/chef-10.12.0/lib/chef/client.rb:199:in `setup_run_context'
    /opt/chef/embedded/lib/ruby/gems/1.9.1/gems/chef-10.12.0/lib/chef/client.rb:162:in `run'
    /opt/chef/embedded/lib/ruby/gems/1.9.1/gems/chef-10.12.0/lib/chef/application/solo.rb:207:in `block in run_application'
    /opt/chef/embedded/lib/ruby/gems/1.9.1/gems/chef-10.12.0/lib/chef/application/solo.rb:195:in `loop'
    /opt/chef/embedded/lib/ruby/gems/1.9.1/gems/chef-10.12.0/lib/chef/application/solo.rb:195:in `run_application'
    /opt/chef/embedded/lib/ruby/gems/1.9.1/gems/chef-10.12.0/lib/chef/application.rb:70:in `run'
    /opt/chef/embedded/lib/ruby/gems/1.9.1/gems/chef-10.12.0/bin/chef-solo:25:in `<top (required)>'
    /usr/bin/chef-solo:19:in `load'

Stacktraces follow execution down to the very place that the exception was raised. Because of this the point of which the exception originated from your recipe is almost never the last line executed. In this example you can see that 5 lines from the top we see the last thing that was executed by our recipe.

    /tmp/vagrant-chef-1/chef-solo-1/cookbooks/myface/recipes/default.rb:10:in `from_file'

Line 10 in the default recipe is the start of the `artifact_deploy` block that we put into the recipe. The NameError exception says that we can't find a resource for artifact_deploy for the CentOS 6.3 platform. This is because we haven't told our cookbook about the Artifact Cookbook that contains the Light-weight Resource and Provider (LWRP) that provides `artifact_deploy` to our recipe.

## Working with cookbook metadata

To tell our myface cookbook about the Artifact cookbook we need to modify the `metadata.rb` file at the root of our cookbook's directory. This is an often overlooked file to new Chef developers but it is one of the most important. 

The metadata file is a lot like a RubyGems `gemspec`, it tells your Chef Server some important things about your cookbook such as:

* the name of a cookbook set by the `name` attribute
* the version of a cookbook set by the `version` attribute
* a list of dependent cookbooks and optionally their versions set by `depends` definitions
* a list of conflicting cookbooks and opitonally their versions set by `conflicts` definitions
* the maintainer of the cookbook set by the `maintainer` attribute
* the email adderess of the maintainer of the cookbook set by the `maintainer_email` attribute
* license information set by the `license` attribute
* a description of the cookbook set by the `description` and `long_description` attributes

It is important to note that not all of these attributes are required. Surprisingly, the name attribute is optional. It is dangerous to leave this attribute blank because the name of the cookbook will then be inferred by the directory containing the contents of the cookbook when it is loaded. Remember to __always set the name attribute for your cookbook__ and save operators or fellow cookbook authors a headache.

For more information see the complete documentation for [Cookbook Metadata](http://wiki.opscode.com/display/chef/Metadata).

Open up the `metadata.rb` file in our cookbook and add the following line of code to the bottom

    depends "artifact", "~> 0.10.1"

This tells the Chef server and clients that the myface cookbook depends on the artifact cookbook. We've also provided a version constraint to the dependency which ensures that other cookbook authors or operators are using a version of the artifact cookbook that we approve works with our cookbook. You should __always set resonable version constraints for your dependencies__ to save your operators and fellow cookbook authors from wanting to [light you on fire](http://24.media.tumblr.com/tumblr_m7fpxfkHM81rzupqxo1_500.png).

Cookbooks follow the [SemVer](http://semver.org) versioning scheme and accept constraints containing anyone of the approved constraint operators. In this case we've used the optimistic operator `~>` to tell our cookbook that we allow any version of artifact that is greater than 0.10.1 but _not_ 0.11.0 or higher. This means we accept 0.10.1, 0.10.2, or 0.10.30002, etc.

Now you should have a `metadata.rb` file that looks like this

    name             "myface"
    maintainer       "YOUR_NAME"
    maintainer_email "YOUR_EMAIL"
    license          "All rights reserved"
    description      "Installs/Configures myface"
    long_description IO.read(File.join(File.dirname(__FILE__), 'README.md'))
    version          "0.0.1"

    depends "artifact", "~> 0.10.1"

Now re-run the vagrant provisioner and see what we get

    $ bundle exec vagrant provision
    [default] Running provisioner: Vagrant::Provisioners::ChefSolo...
    [default] Generating chef JSON and uploading...
    [default] Running chef-solo...
    [Mon, 23 Jul 2012 21:58:45 +0000] INFO: *** Chef 10.12.0 ***
    [Mon, 23 Jul 2012 21:58:46 +0000] INFO: Setting the run_list to ["recipe[myface::default]"] from JSON
    [Mon, 23 Jul 2012 21:58:46 +0000] INFO: Run List is [recipe[myface::default]]
    [Mon, 23 Jul 2012 21:58:46 +0000] INFO: Run List expands to [myface::default]
    [Mon, 23 Jul 2012 21:58:46 +0000] INFO: Starting Chef Run for localhost
    [Mon, 23 Jul 2012 21:58:46 +0000] INFO: Running start handlers
    [Mon, 23 Jul 2012 21:58:46 +0000] INFO: Start handlers complete.
    [Mon, 23 Jul 2012 21:58:46 +0000] ERROR: Running exception handlers
    [Mon, 23 Jul 2012 21:58:46 +0000] ERROR: Exception handlers complete
    [Mon, 23 Jul 2012 21:58:46 +0000] FATAL: Stacktrace dumped to /tmp/vagrant-chef-1/chef-stacktrace.out
    [Mon, 23 Jul 2012 21:58:46 +0000] FATAL: NameError: Cannot find a resource for artifact_deploy on centos version 6.3
    Chef never successfully completed! Any errors should be visible in the
    output above. Please fix your recipes so that they properly complete.

Looks like we have the same error as before even after we've told the myface cookbook that requires the artifact cookbook. What did we forget?

## Using Berkshelf to gather dependencies

After you've got your `metadata.rb` setup with some cookbook dependencies you need gather those dependencies and make them available to Vagrant. This can easily be done with Berkshelf

    $ bundle exec berks install --shims
    Using myface (0.0.1) at path: '/Users/reset/code/myface'
    Installing artifact (0.10.1) from site: 'http://cookbooks.opscode.com/api/v1/cookbooks'
    Shims written to: '/Users/reset/code/myface/cookbooks'

The `--shims` flag is very important to include. In case you have forgotten what shims are; shims are the go between for your cookbooks on your host machine to your virtual machine. They are what make your cookbooks available within your virtual machine.

You may be wondering how Berkshelf was able to figure out where the artifact cookbook was located and how it retrieved it. When we generated the cookbook earlier on it created a file called `Berksfile` at the root of the myface cookbook. This file is read by the `berks` command when you run the install command. If you open the Berksfile that was generated for you, you will see just one line with one word

    metadata

If you are familiar with RubyGems and Bundler this is similar to the their `gemspec` keyword. Berkshelf will inspect the `metadata.rb` file of the cookbook and recursively download the dependencies of your cookbook and their dependencies (and so on). Berkshelf will search the Opscode community site for a cookbook matching the version constraint if you do not explicitly provide a source for the artifact cookbook in your Berksfile.

If the version of the artifact cookbook wasn't available on the Opscode community site or you just want to host things locally an explicit source can be provided for where the cookbook can be found

    cookbook 'artifact', '~> 0.10.1', chef_api: :knife

This entry in your Berksfile would tell Berkshelf to look at at Chef API using your Knife configuration for authorization to download the artifact cookbook. Let's leave things as is for now for the purposes of this guide.

Now if we re-run the Vagrant provisioner we should no longer have a NameError exception raised for the missing artifact_deploy Light-weight Resource and Provider (LWRP).

    $ bundle exec vagrant provision
    ...
    [Mon, 23 Jul 2012 22:04:20 +0000] ERROR: Running exception handlers
    [Mon, 23 Jul 2012 22:04:20 +0000] ERROR: Exception handlers complete
    [Mon, 23 Jul 2012 22:04:20 +0000] FATAL: Stacktrace dumped to /tmp/vagrant-chef-1/chef-stacktrace.out
    [Mon, 23 Jul 2012 22:04:20 +0000] FATAL: Chef::Exceptions::UserIDNotFound: artifact_deploy[myface] (myface::default line 10) had an error: Chef::Exceptions::UserIDNotFound: directory[/tmp/vagrant-chef-1/artifact_deploys/myface/1.0.0] (/tmp/vagrant-chef-1/chef-solo-1/cookbooks/artifact/providers/deploy.rb line 201) had an error: Chef::Exceptions::UserIDNotFound: cannot determine user id for 'myface', does the user exist on this system?

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
    [default] Running provisioner: Vagrant::Provisioners::ChefSolo...
    [default] Generating chef JSON and uploading...
    [default] Running chef-solo...
    [Mon, 23 Jul 2012 22:14:05 +0000] INFO: *** Chef 10.12.0 ***
    [Mon, 23 Jul 2012 22:14:06 +0000] INFO: Setting the run_list to ["recipe[myface::default]"] from JSON
    [Mon, 23 Jul 2012 22:14:06 +0000] INFO: Run List is [recipe[myface::default]]
    [Mon, 23 Jul 2012 22:14:06 +0000] INFO: Run List expands to [myface::default]
    [Mon, 23 Jul 2012 22:14:06 +0000] INFO: Starting Chef Run for localhost
    [Mon, 23 Jul 2012 22:14:06 +0000] INFO: Running start handlers
    [Mon, 23 Jul 2012 22:14:06 +0000] INFO: Start handlers complete.
    [Mon, 23 Jul 2012 22:14:06 +0000] INFO: Processing group[myface] action create (myface::default line 10)
    [Mon, 23 Jul 2012 22:14:06 +0000] INFO: Processing user[myface] action create (myface::default line 12)
    [Mon, 23 Jul 2012 22:14:06 +0000] INFO: Processing artifact_deploy[myface] action deploy (myface::default line 18)
    ...
    [Mon, 23 Jul 2012 22:14:06 +0000] INFO: Chef Run complete in 0.492397486 seconds
    [Mon, 23 Jul 2012 22:14:06 +0000] INFO: Running report handlers
    [Mon, 23 Jul 2012 22:14:06 +0000] INFO: Report handlers complete

You should now have your application code deployed to `/srv/myface/current`

    $ bundle exec vagrant ssh -c "ls -lah /srv/myface/current"
    lrwxrwxrwx 1 root root 26 Jul 23 22:14 /srv/myface/current -> /srv/myface/releases/1.0.0

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

You should __always abstract your tunebles and constants into attributes__. Let's [DRY](http://en.wikipedia.org/wiki/Don't_repeat_yourself) up our code and replace these strings with attributes.

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

Within a recipe attributes are accessed on the [Node Object](http://wiki.opscode.com/display/chef/Recipes#Recipes-NodeObject). Attributes are accessed in a similar fashion to how you would interact with a Hash in Ruby. You can use a symbol or a string for keys in your attributes and they are interchangable, Chef will not complain and tell you that your attribute is not defined if you use a string when you initialized it with a symbol. Although you can use strings, it is __strongly recommended that you [use symbols for keys in Ruby](http://www.robertsosinski.com/2009/01/11/the-difference-between-ruby-symbols-and-strings/)__.

So let's re-provision with Vagrant and see how our refactor went

    $ bundle exec vagrant provision
    [Mon, 23 Jul 2012 23:37:07 +0000] INFO: *** Chef 10.12.0 ***
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

Not so well... it sesems that we have an undefined method somewhere!? 

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
    [Tue, 24 Jul 2012 01:27:09 +0000] INFO: *** Chef 10.12.0 ***
    [Tue, 24 Jul 2012 01:27:09 +0000] INFO: Setting the run_list to ["recipe[myface::default]"] from JSON
    [Tue, 24 Jul 2012 01:27:09 +0000] INFO: Run List is [recipe[myface::default]]
    [Tue, 24 Jul 2012 01:27:09 +0000] INFO: Run List expands to [myface::default]
    [Tue, 24 Jul 2012 01:27:09 +0000] INFO: Starting Chef Run for localhost
    [Tue, 24 Jul 2012 01:27:09 +0000] INFO: Running start handlers
    [Tue, 24 Jul 2012 01:27:09 +0000] INFO: Start handlers complete.
    [Tue, 24 Jul 2012 01:27:09 +0000] ERROR: Running exception handlers
    [Tue, 24 Jul 2012 01:27:09 +0000] ERROR: Exception handlers complete
    [Tue, 24 Jul 2012 01:27:09 +0000] FATAL: Stacktrace dumped to /tmp/vagrant-chef-1/chef-stacktrace.out
    [Tue, 24 Jul 2012 01:27:09 +0000] FATAL: NoMethodError: undefined method `[]' for nil:NilClass

Or at least it should have... There is currently a limitation with the way shims are implemented in Berkshelf. When a file is changed the changes are automatically picked up since we have already created the hard link. Unfortunately when a new file is added we hadn't created a hard link for it earlier on so we need to re-run berks install to get this file tracked inside Vagrant. I am currently working on a fix for this and will update this documentation when it has been implemented.

    $ bundle exec berks install --shims
    Using myface (0.0.1) at path: '/Users/reset/code/myface'
    Using artifact (0.10.1)
    Shims written to: '/Users/reset/code/myface/cookbooks'

Now re-run the provisioner and everything will be K

    $ bundle exec vagrant provision
    [default] Running provisioner: Vagrant::Provisioners::ChefSolo...
    [default] Generating chef JSON and uploading...
    [default] Running chef-solo...
    [Tue, 24 Jul 2012 01:30:30 +0000] INFO: *** Chef 10.12.0 ***
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

You should __always write idempotent recipes__ that execute cleanly on their very first run and perform no work if no work needs to be done.

# Incrementing versions

# Running lint tests

    $ bundle exec thor foodcritic:lint
