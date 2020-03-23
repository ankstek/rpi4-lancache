# How i setup lancache on a Raspberry Pi 4

I wanted a small lancache setup for my home where my SO and I share a 100Mbps (up/down) network connection. I don't think this setup is suitable for anything bigger really. With this setup, i was able to achieve a peak of 450Mbps download for a cached game.

## Hardware
* Raspberry Pi 4 (4GB version), running latest Raspbian (as of 2020-03-21)
* Flirc case (It's just a nice case, not a necessity)
* 16GB SanDisk microSD card
* 2 TB Seagate 2,5" external harddrive (USB 3.0)

## Setup
After reading a bunch of guides, documentation and forum posts, I have now made a fairly easy setup that should be pretty quick to get up and running. I decided on a docker compose file instead of having to manually start and stop each individual container.

#### Disclaimer:
> I assume you have your RPi up and running and have a terminal windows open (ssh or not, doesn't matter). I won't show you how to setup your Raspberry Pi, there are tons of guides on how to do that. I also assume that you have your external drive mounted under some directory. Again, tons of guides available.

Assign a static IP to your RPi in your router. My router is an Asus RT-AC87U and I set the WAN DNS to `1.1.1.1` and the LAN DNS (under the DHCP settings) to the static IP of my RPi. I also told the router to add itself as a DNS in the DHCP request, this is handy during the setup phase but should not be enabled when everything is running. My RPi4 is connected to my router with an ethernet cabel.

* First install `docker` and `docker-compose` on your RPi.
* Clone/copy/download the docker compose file to the RPi.
* Edit the config to your needs, I've commented the `.yaml` file extensivly incase you are unsure what each setting does. Feel free to suggest and share your own environment variables and why you use them!
* Then it should only be a matter of navigating to the `docker-compose.yaml` file I've provided and running `sudo docker-compose up -d`.

## Tuning
As a few extra steps, giving direct network access to the lancache container by using host network can inprove performance. I noticed that I had a lot of cache misses and timeouts, [these instructions](https://github.com/lancachenet/monolithic/issues/80#issuecomment-566738358) can fix it.

In your RPi terminal, with the containers running:
* Run `sudo docker ps`
* Run `docker exec -it lancache /bin/bash`, where lancache is the container ID of the lancache container
You should now be in the lancache container
* Run `apt -y install nano` to get a simple text editor (or use vim)
* Open a config file with nano `nano /etc/nginx/sites-available/10_generic.conf`
* At the top of this file you will see the line</br>
`listen 80 reuseport;`</br>
* Change this to</br>
`listen 192.168.1.92:80 reusport;` replace the `192.168.1.92` IP with the IP of your RPi
* `ctrl+s` and then `ctrl+x` to save
* Then run `service nginx restart`</br>

If it failes, then just run 
> `service nginx start`</br>

If that failes then I have no idea what could have gone wrong as I'm not an nginx expert.
It it returns `[OK]` then your lancache server will now have full network access and should not have as many timeouts anymore.

> #### Remember to test if it works; download a game on Steam that's a gigabyte or two in size, uninstall it and download it again to test your cache speeds.

I'm using [Pegasy's prebuilt RPi lancache images](https://github.com/pegasy/lancachenet_rpi), he has a build script incase you want to build your own images.

## Gotchas

* The RPi should not have itself as DNS for it's own request. When starting the docker container and docker needs to pull an image it won't be able to resolve the hostname since it cannot ask it's own dns server since it's not started.

> `sudo docker-compose up -d`</br>
> `Creating network "lancache_default" with the default driver`</br>
> `Pulling lancache-dns (pegasy/lancachenet_lancache-dns:latest)...`</br>
> `ERROR: Get https://registry-1.docker.io/v2/: dial tcp: lookup registry-1.docker.io on 192.168.1.92:53: read udp 192.168.1.92:42648->192.168.1.92:53: read: connection refused`</br>

Fix it by using a text editor to edit the file `/etc/resolv.conf`
> `sudo nano /etc/resolv.conf`</br>

Reference the same IP as your upstream DNS. Make sure that the IP of your RPi isn't in there. Save the file and exit.
Make sure to make the file read-only (except for yourself) to prevent DHCP to overwrite it.
> `sudo chmod 744 /etc/resolv.conf`</br>


## TODO
Stuff I'd like to setup, but haven't (yet).

* Log rotation
* Automatic start of containers on boot

## Credits/sources
[lancache](https://github.com/lancachenet/monolithic)</br>
[lancache docs](http://lancache.net/docs/containers/monolithic/variables/)</br>
[Pegasy](https://github.com/pegasy/lancachenet_rpi)</br>
[decryptedchaos](https://github.com/lancachenet/monolithic/issues/80#issuecomment-566738358)</br>
[DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-optimize-nginx-configuration)</br>
