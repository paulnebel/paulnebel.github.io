---
title: "HypriotOS: Get Swarm working on a Raspberry Pi Cluster"
categories: API Containers Lean Node Raspberry_Pi Swarm
meta: "Use HypriotOS to run Docker Swarm on your Raspberry Pi cluster. See how to resolve networking issues and update to the latest version of Docker."
slug: "hypriotos-swarm-raspberry-pi-cluster"
excerpt: "I chose to set up a Docker cluster from scratch so as to learn as much as I could. Well, I learned that sometimes discretion is the better part of valour. This post explains why I decided to install HypriotOS instead and how I finally got it to work."
---
# Introduction
In a [previous post]({% post_url 2016-08-09-building-a-4-node-raspberry-pi-cluster %}){:target="_blank"} I described how I set up docker on a 4-node Raspberry Pi cluster. The reason for doing this was to be able to investigate [Docker Swarm][2]{:target="_blank"} in more detail. I chose to set up the cluster from scratch so as to learn as much as I could. Well, I learned that sometimes discretion is the better part of valour. This post explains why I decided to install [HypriotOS][3]{:target="_blank"} instead and how I finally got it to work. 

# Updating the Cluster
So, having installed Docker on my Pi cluster I wanted to create a Swarm. I also wanted to use Docker 1.12 release, and so far I had only managed to install release 1.10.3. Since I have Docker for Mac installed on my laptop I followed [this][4]{:target="_blank"} article to try to run release 1.12 on my Pi nodes even though the local version is 1.10. This is where I started to run into problems: 
{% highlight bash %}
$ docker-machine create --driver generic --generic-ip-address=192.168.0.17 --generic-ssh-key=/Users/****/.ssh/id_rsa --generic-ssh-user=pi --engine-storage-driver=overlay rpi1
Running pre-create checks...
Creating machine...
(rpi1) Importing SSH key...
Waiting for machine to be running, this may take a few minutes...
Detecting operating system of created instance...
Waiting for SSH to be available...
Detecting the provisioner...
Error creating machine: Error detecting OS: OS type not recognized
{% endhighlight %}

Right. OK. So my setup might be missing something after all! Let's try doing the same from one of my Pi nodes and see what happens: 
{% highlight bash %}
$ docker-machine create --driver generic --generic-ip-address=192.168.0.17 --generic-ssh-key=/home/pi/.ssh/id_rsa --generic-ssh-user=pi --engine-storage-driver=overlay rpi1
Creating CA: /home/pi/.docker/machine/certs/ca.pem
Creating client certificate: /home/pi/.docker/machine/certs/cert.pem
Importing SSH key...
SSH cmd error!
command: cat /etc/hypriot_release
err : exit status 1
output : cat: /etc/hypriot_release: No such file or directory
{% endhighlight %}

Ugh. I'm sure I could have worked my way around these issues. I'm also sure it would take a long time, and I don't want to waste time on the details of how Docker works (or fails to work!) on a bare-bones Raspberry Pi. Time to back-track and make my life easier by installing the HypriotOS. I followed [this][5]{:target="_blank"} fantastic article to get HypriotOS installed. 

# Install HypriotOS
The first task was to overwrite my SD cards with the new image using the [Hypriot flash tool][6]{:target="_blank"}: 

{% highlight bash %}
$ flash --hostname rpi1 https://github.com/hypriot/image-builder-rpi/releases/download/v0.8.0/hypriotos-rpi-v0.8.0.img.zip
{% endhighlight %}

Be aware that `raspi-config` is not part of HyproitOS, but it automatically resizes the filesystem to the size of the SD card so this isn't much of an issue. It does mean, however, that to change your password you will have to use the `passwd` command-line tool (the default username/password credentials are `pirate/hypriot`).

There is no need to update the `/etc/hosts` file as HypriotOS starts the `avahi-daemon` to announce the hostname through mDNS, so each Pi is immediately reachable through `{hostname}.local`. That's the theory anyway. I had enormous problems with the 'configuration free' `avahi-daemon`. 

# Network, what network?
The first disaster on starting up the newly configured Pis was that as soon as I stopped any of them I lost my internet connection within a minute or two of re-starting them. From that point onwards, no matter what I did, restarting any of the Pis completely hosed my internet connection. I couldn't even access the GUI for my router (a bog-standard Virgin box) but I could access my local network. Sort of.

There was no problem `ssh`-ing onto any individual Pi fom my Mac using `{hostname}.local` but once I was in any Pi I couldn't see the others using the same mechanism. A bit of searching led me to realise that an essential package was missing from my HypriotOS. I needed `libnss-mdns` before the Pis would resolve each other. I thought it somewhat strange that this would be missing, but there you go. Perhaps this was why I was having problems with my router? Let's install it then: 
{% highlight bash %}
$ sudo apt-get install libnss-mdns
{% endhighlight %}
Nope, that didn't work. I could now see one Pi from another but I still couldn't access my router or internet connection. I spent the next day stopping and starting Pis and stopping and starting my router to no avail. No matter what I did I kept losing my internet connection when I started the Pis. As a last resort I disconnected all but one of the boards and disabled the `avahi-daemon`. Finally, I could stop and start this Pi without losing my internet connection. 

# Reinstall `avahi-daemon`
I was about to give up and do this on each board when I thought I'd try one last thing: uninstall and reinstall `avahi-daemon` to see if it made a difference.

I reviewed the daemon status before uninstalling: 
{% highlight bash %}
$ sudo service avahi-daemon status
● avahi-daemon.service - Avahi mDNS/DNS-SD Stack
   Loaded: loaded (/lib/systemd/system/avahi-daemon.service; disabled)
   Active: active (running) since Wed 2016-08-17 12:49:49 UTC; 5s ago
 Main PID: 1252 (avahi-daemon)
   Status: "avahi-daemon 0.6.31 starting up."
   CGroup: /system.slice/avahi-daemon.service
           ├─1252 avahi-daemon: running [rpi1.local]
           └─1254 avahi-daemon: chroot helper

Aug 17 12:49:52 rpi1 avahi-daemon[1252]: Interface eth0.200.IPv4 no longer relevant for mDNS.
Aug 17 12:49:52 rpi1 avahi-daemon[1252]: Withdrawing address record for fe80::ba27:ebff:fe8b:840d on eth0.200.
Aug 17 12:49:52 rpi1 avahi-daemon[1252]: Leaving mDNS multicast group on interface eth0.200.IPv6 with address fe80::ba27:ebff:fe8b:840d.
Aug 17 12:49:52 rpi1 avahi-daemon[1252]: Interface eth0.200.IPv6 no longer relevant for mDNS.
Aug 17 12:49:52 rpi1 avahi-daemon[1252]: Joining mDNS multicast group on interface eth0.200.IPv4 with address 192.168.200.1.
Aug 17 12:49:52 rpi1 avahi-daemon[1252]: New relevant interface eth0.200.IPv4 for mDNS.
Aug 17 12:49:52 rpi1 avahi-daemon[1252]: Registering new address record for 192.168.200.1 on eth0.200.IPv4.
Aug 17 12:49:52 rpi1 avahi-daemon[1252]: Files changed, reloading.
Aug 17 12:49:52 rpi1 avahi-daemon[1252]: Loading service file /services/cluster-leader.service.
Aug 17 12:49:53 rpi1 avahi-daemon[1252]: Service "Cluster-Leader=rpi1" (/services/cluster-leader.service) successfully established.
{% endhighlight %}

OK, the fact that a new address of `192.168.200.1` is being registered goes some way toward explaining why I can reach my local network but not my router (or, therefore, the internet). The router is on `192.168.0.1` and the new address doesn't match the subnet mask (`255.255.255.0`). Why the heck it's inserted the `200` and how it's persuaded my Mac to change its local record is something of a mystery to me but not one I care to investigate in any great detail. Perhaps I should mention at this point that I'm still on OS X Yosemite which may be part of the problem (I've seen a lot of posts indicating that the bonjour service on Yosemite is at least partly broken).

Whatever, let's uninstall and re-install `avahi-daemon` and see what happens: 

{% highlight bash %}
$ sudo apt-get purge avahi-daemon
$ sudo apt-get install avahi-daemon
{% endhighlight %}

I can immediately see that things are getting better as `libnss-mdns` is installed as a dependency of `avahi-daemon` (which makes it all the more strange that it wasn't there in the first place). Now let's look at the service status: 

{% highlight bash %}
$ sudo service avahi-daemon status
● avahi-daemon.service - Avahi mDNS/DNS-SD Stack
   Loaded: loaded (/lib/systemd/system/avahi-daemon.service; enabled)
   Active: active (running) since Wed 2016-08-17 14:35:10 UTC; 18s ago
 Main PID: 612 (avahi-daemon)
   Status: "avahi-daemon 0.6.31 starting up."
   CGroup: /system.slice/avahi-daemon.service
           ├─612 avahi-daemon: running [rpi1.local]
           └─614 avahi-daemon: chroot helper

Aug 17 14:35:10 rpi1 avahi-daemon[612]: Joining mDNS multicast group on interface eth0.IPv6 with address fe80::ba27:ebff:fe8b:840d.
Aug 17 14:35:10 rpi1 avahi-daemon[612]: New relevant interface eth0.IPv6 for mDNS.
Aug 17 14:35:10 rpi1 avahi-daemon[612]: Joining mDNS multicast group on interface eth0.IPv4 with address 192.168.0.12.
Aug 17 14:35:10 rpi1 avahi-daemon[612]: New relevant interface eth0.IPv4 for mDNS.
Aug 17 14:35:10 rpi1 avahi-daemon[612]: Network interface enumeration completed.
Aug 17 14:35:10 rpi1 avahi-daemon[612]: Registering new address record for fe80::ba27:ebff:fe8b:840d on eth0.*.
Aug 17 14:35:10 rpi1 avahi-daemon[612]: Registering new address record for 192.168.0.12 on eth0.IPv4.
Aug 17 14:35:10 rpi1 avahi-daemon[612]: Registering HINFO record with values 'ARMV7L'/'LINUX'.
Aug 17 14:35:10 rpi1 systemd[1]: Started Avahi mDNS/DNS-SD Stack.
Aug 17 14:35:11 rpi1 avahi-daemon[612]: Server startup complete. Host name is rpi1.local. Local service cookie is 2923648342.
Aug 17 14:35:12 rpi1 avahi-daemon[612]: Service "Cluster-Leader=rpi1" (/services/cluster-leader.service) successfully established.
{% endhighlight %}
Not only does that look better, it is better as now I can stop and re-start Pis at will and still connect to the internet (and from one Pi to another). Problem solved. 

# Configure `docker.service`
From here I continued exactly as I had [before][1]{:target="_blank"} and generated SSH keys for each node which I copied to the other nodes so that I could ssh between them (using the `{hostname}.local` address). There's no need to update `/etc/hosts` as `avahi-daemon` is now taking care of that correctly. I re-mounted the USB drive on the head node, installed and started `nfs-kernel-server` on the head node and `nfs-common` and `autofs` on the client nodes to share the drive between nodes. Finally, I updated docker engine on each node to release 1.12: 
{% highlight bash %}
$ curl -sSL https://jenkins.hypriot.com/job/armhf-docker/15/artifact/bundles/latest/build-deb/raspbian-jessie/docker-engine_1.12.0~rc3-0~jessie_armhf.deb &gt; /tmp/docker-engine_1.12.0~rc3-0~jessie_armhf.deb
$ sudo apt-get purge -y docker-hypriot
$ sudo dpkg -i /tmp/docker-engine_1.12.0~rc3–0~jessie_armhf.deb
$ sudo rm -rf /tmp/docker-engine_1.12.0~rc3–0~jessie_armhf.deb
{% endhighlight %}
Now we're ready to go, right? Let's check: 
{% highlight bash %}
$ docker version
Client:
 Version:      1.12.0-rc3
 API version:  1.24
 Go version:   go1.6.2
 Git commit:   91e29e8
 Built:        Sat Jul  2 14:58:48 2016
 OS/Arch:      linux/arm
Cannot connect to the Docker daemon. Is the docker daemon running on this host?
{% endhighlight %}
Oh boy, it never rains but it pours. Now what's wrong? 
{% highlight bash %}
$ sudo service docker status
● docker.service - Docker Application Container Engine
   Loaded: loaded (/etc/systemd/system/docker.service; enabled)
   Active: failed (Result: start-limit) since Thu 2016-08-18 14:35:57 UTC; 14min ago
     Docs: https://docs.docker.com
  Process: 1114 ExecStart=/usr/bin/docker daemon --storage-driver overlay --host fd:// --debug --host tcp://192.168.200.1:2375 --cluster-advertise 192.168.200.1:2375 --cluster-store consul://192.168.200.1:8500 --label hypriot.arch=armv7l --label hypriot.hierarchy=follower (code=exited, status=1/FAILURE)
 Main PID: 1114 (code=exited, status=1/FAILURE)

Aug 18 14:35:57 rpi3 docker[1114]: time="2016-08-18T14:35:57.038532811Z" level=debug msg="Listener created for HTTP on fd ()"
Aug 18 14:35:57 rpi3 docker[1114]: time="2016-08-18T14:35:57.040795121Z" level=warning msg="[!] DON'T BIND ON ANY IP ADDRESS WITHOUT setting -tlsverify IF YOU DON...E DOING [!]"
Aug 18 14:35:57 rpi3 docker[1114]: time="2016-08-18T14:35:57.042908470Z" level=fatal msg="listen tcp 192.168.200.1:2375: bind: cannot assign requested address"
Aug 18 14:35:57 rpi3 systemd[1]: docker.service: main process exited, code=exited, status=1/FAILURE
Aug 18 14:35:57 rpi3 systemd[1]: Failed to start Docker Application Container Engine.
Aug 18 14:35:57 rpi3 systemd[1]: Unit docker.service entered failed state.
Aug 18 14:35:57 rpi3 systemd[1]: Starting Docker Application Container Engine...
Aug 18 14:35:57 rpi3 systemd[1]: docker.service start request repeated too quickly, refusing to start.
Aug 18 14:35:57 rpi3 systemd[1]: Failed to start Docker Application Container Engine.
Hint: Some lines were ellipsized, use -l to show in full.
{% endhighlight %}
Hang on a minute, the IP address the docker service is being started with is the old incorrect one that `avahi-daemon` created before being reinstalled. How about if I update the service default values at `/etc/systemd/system/docker.service` and replace this IP address with the new one allocated by `avahi-daemon` (which in this case is `192.168.0.12`)? Well, I did that and ran: 
{% highlight bash %}
$ sudo service docker start
Warning: Unit file of docker.service changed on disk, 'systemctl daemon-reload' recommended.
Job for docker.service failed. See 'systemctl status docker.service' and 'journalctl -xn' for details.
{% endhighlight %}
OK, so the default values have been cached. Let's clear the cache and try again: 
{% highlight bash %}
$ sudo systemctl daemon-reload
$ sudo service docker start
$ sudo service docker status
● docker.service - Docker Application Container Engine
   Loaded: loaded (/etc/systemd/system/docker.service; enabled)
   Active: active (running) since Thu 2016-08-18 15:13:55 UTC; 2s ago
     Docs: https://docs.docker.com
 Main PID: 1126 (dockerd)
   CGroup: /system.slice/docker.service
           ├─1126 dockerd --storage-driver overlay --host fd:// --debug --host tcp://192.168.0.12:2375 --cluster-advertise 192.168.0.12:2375 --cluster-store consul://192.168....
           ├─1139 docker-containerd -l unix:///var/run/docker/libcontainerd/docker-containerd.sock --shim docker-containerd-shim --metrics-interval=0 --start-timeout 2m --sta...
           ├─1280 docker-containerd-shim 5887b36902a6b64371b640364cacab5c99a8618755818878a75ef7dcce403951 /var/run/docker/libcontainerd/5887b36902a6b64371b640364cacab5c99a861...
           ├─1284 docker-proxy -proto tcp -host-ip 0.0.0.0 -host-port 2378 -container-ip 172.17.0.3 -container-port 2375
           └─1313 docker-containerd-shim 9d8e309946e514456df7870bf3afd440ec2e2e97d573e524838119432b625131 /var/run/docker/libcontainerd/9d8e309946e514456df7870bf3afd440ec2e2e...
         Hint: Some lines were ellipsized, use -l to show in full.
{% endhighlight %}
Right, that actually worked. Now to check docker-engine: 
{% highlight bash %}
$ docker version
Client:
 Version:      1.12.0-rc3
 API version:  1.24
 Go version:   go1.6.2
 Git commit:   91e29e8
 Built:        Sat Jul  2 14:58:48 2016
 OS/Arch:      linux/arm

Server:
 Version:      1.12.0-rc3
 API version:  1.24
 Go version:   go1.6.2
 Git commit:   91e29e8
 Built:        Sat Jul  2 14:58:48 2016
 OS/Arch:      linux/arm
{% endhighlight %}
Phew, we're there! I tried using `{hostname}.local` instead of the IP address in `/etc/systemd/system/docker.service` but, as I expected, that didn't work. So I've got everything working now, but what if `avahi-daemon` registers a different IP address for this node on startup? Will I have to update the `docker.service` defaults? I don't know for now, let's wait and see.

Time to actually do some docker stuff.

 [1]: {% post_url 2016-08-09-building-a-4-node-raspberry-pi-cluster %}
 [2]: https://docs.docker.com/engine/swarm/
 [3]: http://blog.hypriot.com/post/hypriotos-barbossa-for-raspberry-pi-3/
 [4]: http://blog.hypriot.com/post/swarm-machines-or-having-fun-with-docker-machine-and-the-new-docker-swarm-orchestration/
 [5]: https://medium.com/@bossjones/how-i-setup-a-raspberry-pi-3-cluster-using-the-new-docker-swarm-mode-in-29-minutes-aa0e4f3b1768
 [6]: https://github.com/hypriot/flash