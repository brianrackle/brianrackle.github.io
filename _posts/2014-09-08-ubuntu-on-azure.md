---
layout: post
title:  "Ubuntu on Azure"
date:   2014-09-08 09:25:30
categories: update c++
tags: c++ llvm clang c++11 c++14 emacs ubuntu libc++ linux shell azure
---

I have decided to move my projects to use Clang on Linux. The primary reason that spurred this decision is to be able to keep pace with the latest C++ features. C++14 refines C++11 and makes the language much more intuitive and concise. This being said, I need a compiler that supports the latest language features and currently Clang is king of [feature support](http://clang.llvm.org/cxx_status.html){:target="_blank"}. I could whine about Visual Studio but it is a great IDE. I use Visual Studio daily, and will continue to use it in projects when I see fit. However, right now for this blog, Ubuntu, Emacs and Clang is my new platform. 

It has been six years since I last developed on Linux, and it seems some things have changed since then. As such, I have put together a guide to make setup as easy as possible. This guide is exactly how I have my machine setup and once you have your Ubuntu machine running all you have to do is run a single line of shell commands I have put together.

## The Machine

First, thing you are going to need is your Linux machine. Sure you could grab an old laptop, dual boot, or install a local VM, hell you might even have a server at home to run your VM on. If so, you will want to head over to [http://www.ubuntu.com/download/server](http://www.ubuntu.com/download/server){:target="_blank"} and download an ISO and install it. This is not what I did. To make things easy, and to have a VM that I could access from anywhere, I created a cloud VM on Microsoft's Azure platform.

If you are unfamiliar with Azure, go to [http://azure.microsoft.com/](http://azure.microsoft.com/){:target="_blank"} and click the [Free Trial](http://azure.microsoft.com/en-us/pricing/free-trial/){:target="_blank"} button in the top right corner. You will get free credits to spend on Azure services and if you manage them correctly they will last you quite a long time.

## Azure Ubuntu Server 14.04 LTS

Once you have your Azure account setup, you need to create your Linux VM. Go to your [Portal](https://manage.windowsazure.com){:target="_blank"}. Click the [Virtual Machines](https://manage.windowsazure.com/#Workspaces/VirtualMachineExtension/vms){:target="_blank"} tab and down at the bottom left click "New", which looks something like: 

<img src="/assets/moving-to-ubuntu/new-vm.png" alt="new vm" width="400">

Then click "From Gallery":

<img src="/assets/moving-to-ubuntu/vm-from-gallery.png" alt="new vm" width="400">

Then Choose an Image, selecting Ubuntu, and then Ubuntu Server 14.04 LTS:

<img src="/assets/moving-to-ubuntu/choose-ubuntu-image.png" alt="choose image" width="400">

Now configure your VM with the following options. Basic A1 is the second cheapest at $55 per month and Basic A0 is cheaper at $13 per month. I recommend using Basic A1 at least while you setup the machine so that it doesn't take hours, you can switch to Basic A0 afterwards. The user name and password will be used to log into your VM:

<img src="/assets/moving-to-ubuntu/vm-config.png" alt="vm config" width="400">

Next you will choose the region for your VM, generally you want the VM close to you. Also set the "Cloud Server DNS Name", which is the the address you will use to SSH into the VM. To add other "services" to your VM such as RDP or VNC you will add the appropriate endpoints to the Cloud Service you are setting up now.

<img src="/assets/moving-to-ubuntu/cloud-service.png" alt="cloud service" width="400">

Click through the rest of the setup and wait about 5 minutes for Azure to provision your brand spanking new Ubuntu VM. Head over to your [Virtual Machines](https://manage.windowsazure.com/#Workspaces/VirtualMachineExtension/vms){:target="_blank"} portal and when Status says "Running", click the circled area:

<img src="/assets/moving-to-ubuntu/vm-page.png" alt="vm page" width="400">

You will be at your VM's Azure Portal and can get all sorts of useful information like the server load and the address for the service and SSH endpoint you set up earlier (under quick glance "SSH Details"). Copy the address under SSH Details.

## Accessing the VM

To access your VM you need an [SSH client](http://en.wikipedia.org/wiki/Secure_Shell){:target="_blank"}. On Windows, I recommend KiTTY, it is a branch of the long-time popular open source PuTTY. [Download KiTTY](http://www.fosshub.com/download/kitty_portable.exe){:target="_blank"}

Open KiTTY. Fill out the SSH Details you copied earlier. Give the connection a nickname. Save.

<img src="/assets/moving-to-ubuntu/kitty.png" alt="kitty" width="400">

Now click Open in the bottom left of the KiTTY window. This will connect your with your Azure VM. You will get a warning because you didn't upload an SSH certificate, click Ok. And then enter your username and password which you setup earlier in the Azure portal.

You now have a Linux VM in the cloud, and that probably didn't take you any more than 10 minutes!

## Setting Up Your Ubuntu VM

[On Github](https://github.com/brianrackle/brainstem_breakfast/blob/master/scripts/one_liner.sh){:target="_blank"}

I have to credit [Solarian Programmer](https://solarianprogrammer.com/2013/01/17/building-clang-libcpp-ubuntu-linux/){:target="_blank"} for providing many of the next steps. However, I have modified and added to them so that they will work with your Azure Ubuntu 14.04 VM. The steps on Solarian will not work for Ubuntu 14.04.

You can follow [each step individually](#each-step-explained), or you can just copy the [all in one version](#all-steps-in-one)

### All Steps in One

This is all the steps for setting up your Ubuntu VM concatenated into a single command. After pasting this into your command prompt, you will have to stick around for a few minutes to type 'y' and 'enter' a few times. Do not close KiTTY or let your computer go to sleep because your session will time-out and your VM will not be set up properly. 

{% highlight bash %}
sudo apt-get update && sudo apt-get upgrade && sudo apt-get install g++ subversion cmake emacs git xdg-utils htop ncurses-dev && sudo mkdir repos && cd ~/repos/ && sudo mkdir clang && cd clang && sudo svn co http://llvm.org/svn/llvm-project/llvm/branches/release_35/ llvm && cd llvm/tools && sudo svn co http://llvm.org/svn/llvm-project/cfe/branches/release_35/ clang && cd ~/repos/clang && sudo mkdir build && cd build && sudo ../llvm/configure --enable-optimized --enable-targets=host --disable-compiler-version-checks && sudo make -j 8 && sudo make install && cd ~/repos/clang && sudo svn co http://llvm.org/svn/llvm-project/libcxx/trunk libcxx && sudo mkdir build_libcxx && cd build_libcxx && sudo CC=clang CXX=clang++ cmake -G "Unix Makefiles" -DLIBCXX_CXX_ABI=libsupc++ -DLIBCXX_LIBSUPCXX_INCLUDE_PATHS="/usr/include/c++/4.8;/usr/include/x86_64-linux-gnu/c++/4.8" -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=/usr $HOME/repos/clang/libcxx && sudo make -j 8 && sudo make install && xdg-mime default emacs.desktop text/plain && cd ~/repos/ && sudo git clone https://github.com/brianrackle/brainstem_breakfast.git && cd brainstem_breakfast/source && sudo clang++ --std=c++14 --stdlib=libc++ main.cpp
{% endhighlight %}

### Each Step Explained

Here is all the steps you need to get your Ubuntu installation up and running with a great compiler. Just copy each line individually intop the command line, and by the end you will be redy to develop on Linux. These commands are the same as the above commands, but are seperated for easy viewing.

#### Initialize Environment

Here we update the software on the VM and install some new software.

{% highlight bash %}
sudo apt-get update
sudo apt-get upgrade
sudo apt-get install g++ subversion cmake emacs git xdg-utils htop ncurses-dev
sudo mkdir repos
{% endhighlight %}


#### Pull Clang and LLVM

Get the source code for Clang and LLVM. LLVM is the backend technology that Clang is built upon.

{% highlight bash %}
cd ~/repos/
sudo mkdir clang
cd clang
sudo svn co http://llvm.org/svn/llvm-project/llvm/branches/release_35/ llvm
cd llvm/tools
sudo svn co http://llvm.org/svn/llvm-project/cfe/branches/release_35/ clang
{% endhighlight %}

#### Build Clang and LLVM

Build and install Clang.

{% highlight bash %}
cd ~/repos/clang
sudo mkdir build
cd build
sudo ../llvm/configure --enable-optimized --enable-targets=host --disable-compiler-version-checks
sudo make -j 8
sudo make install
{% endhighlight %}

#### Pull and Make libc++

Get the libc++ source and build it with Clang. [libc++](http://libcxx.llvm.org/){:target="_blank"} is a newer C++ STL implementation.

{% highlight bash %}
cd ~/repos/clang
sudo svn co http://llvm.org/svn/llvm-project/libcxx/trunk libcxx
sudo mkdir build_libcxx
cd build_libcxx
sudo CC=clang CXX=clang++ cmake -G "Unix Makefiles" -DLIBCXX_CXX_ABI=libsupc++ -DLIBCXX_LIBSUPCXX_INCLUDE_PATHS="/usr/include/c++/4.8;/usr/include/x86_64-linux-gnu/c++/4.8" -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=/usr $HOME/repos/clang/libcxx
sudo make -j 8
sudo make install
{% endhighlight %}

#### Setup Emacs

Emacs is my code editor of choice on Linux. This will set Emacs as the default text editor.

{% highlight bash %}
xdg-mime default emacs.desktop text/plain
{% endhighlight %}

#### Use Git and Compile BSB

Use Git to pull some source code (the brainstem breakfast code) and compile it using clang.

{% highlight bash %}
cd ~/repos/
sudo git clone https://github.com/brianrackle/brainstem_breakfast.git
cd brainstem_breakfast/source
sudo clang++ --std=c++14 --stdlib=libc++ main.cpp
{% endhighlight %}


If you are able to compile the BSB source then you have successfully set up you Clang on Ubuntu and you can now write C++14 code. 

Enjoy your new Linux environment. If you have any problems, let me know in the comments or over twitter and I will do my best to help you.

Cheers!
Brian