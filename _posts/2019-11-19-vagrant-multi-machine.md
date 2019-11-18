---
layout: post
title: Vagrant Multi-Machine
author: foger
tags: virtualization
---

Simple example of Vagrantfile that will create two virtual machines: Ubuntu 18.04 and CentOS 7.7

```ruby
Vagrant.configure("2") do |config|
  config.vm.define "v-ubuntu" do |vu|
    vu.vm.hostname = "v-ubuntu"
    vu.vm.box = "bento/ubuntu-18.04"
    vu.vm.network "public_network", bridge: "en1: Wi-Fi (AirPort)"

    vu.vm.provider "virtualbox" do |vb|
      vb.name = "v-ubuntu"
      vb.gui = false
      vb.memory = 512
      vb.cpus = 1
    end

    vu.vm.provision "shell", run: "always", inline: <<-SHELL
      echo "Hello from Ubuntu!"
    SHELL

  end

  config.vm.define "v-centos" do |vc|
    vc.vm.hostname = "v-centos"
    vc.vm.box = "bento/centos-7.7"
    vc.vm.network "public_network", bridge: "en1: Wi-Fi (AirPort)"

    vc.vm.provider "virtualbox" do |vb|
      vb.name = "v-centos"
      vb.gui = false
      vb.memory = 512
      vb.cpus = 1
    end

    vc.vm.provision "shell", run: "always", inline: <<-SHELL
      echo "Hello from CentOS!"
    SHELL

  end

end
```

```vu.vm.network "public_network", bridge: "en1: Wi-Fi (AirPort)"``` - I used bridged public network from my en1 wireless interface on my iMac

Usage:

```bash
# boot all machines:
$ vagrant up
# boot custom machine:
$ vagrant up v-centos
```

How to create multiple machines in a loop?

```ruby
Vagrant.configure("2") do |config|
    config.vm.box = "bento/ubuntu-18.04"
    config.vm.network "public_network", bridge: "en1: Wi-Fi (AirPort)"
    (1..5).each do |i|
        config.vm.define "v-host-#{i}" do |node|
            node.vm.hostname = "v-host-#{i}"
            node.vm.provider "virtualbox" do |vb|
                vb.name = "v-host-#{i}"
                vb.gui = false
                vb.memory = 512
                vb.cpus = 1
            end
        end
    end
end
```

Will create five Ubuntu 18.04 virtual machines.
