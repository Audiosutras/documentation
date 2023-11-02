# PI-HOLE Docker Compose

[Pi-hole](https://docs.pi-hole.net/) is a DNS sinkhole that is effective at blocking ads and malware by closing connections to blacklisted domains before a client can connect to them. [Unbound](https://nlnetlabs.nl/projects/unbound/about/) is a validating, recursive, caching DNS resolver that increase the privacy of its users.

This project provision three docker containers on a user's chosen machine that always run unless stopped; one for Pihole and the other Unbound. By default, PiHole is configured to use Unbound as its only upstream DNS server. This cuts Google, Cloudflare, and other DNS providers out from having a record of the domains you have requested to visit. In simple terms, this always means the sites you have visited. However note that your Internet Service Provider will still be able to access your DNS history without any obfuscation. For more information I found [this](https://www.reddit.com/r/pihole/comments/ydkkup/what_are_the_benefits_to_unbound/) reddit thread helpful.

The benefits of running pi-hole and unbound in docker containers are many. To speak to a few, it is the ability to run this software across operating system (linux, mac, windows) and across computing devices. You can benefit from adblocking and malware protection on your devices at home by running pihole containerized on a raspberry pi. You can also have this benefit on your laptop when connected to an external network that is not your own like hotel or airport wifi.

Unbound privacy benefits as your configured DNS resolver, but it's important to mention before starting that you will still be able to disable this resolver and use Cloudflare for example for the benefits of internet speed.

Let's get started.

## DNS Resolver CheatSheet

The table below documents the port and internal IP address of custom dns resolvers that the instance of Pihole comes pre-configured with. 

| DNS Resolver | Internal IP Address |
| ------- | ------------------- |
| Unbound | 10.1.1.3#53         |
| Cloudflared | 10.1.1.4#5053   |

## Prerequiste Installation

### Docker & Docker Compose

This project depends on having Docker and Docker Compose installed on the machine 
you plan to run Pihole on. Ensure your machine has at least 4GB of RAM.

To check if `docker` and `docker-compose` is already installed run the following commands from the command line:

```
$ docker --version
Docker version 24.0.6, build ed223bc
$ docker-compose --version
docker-compose version 1.28.0, build d02a7b1a
```

If nothing was returned when running the above commands follow docker's recommended installation method found here under [Scenario one: Install Docker Desktop](https://docs.docker.com/compose/install/#scenario-one-install-docker-desktop) for your operating system and/or linux distribution. If your machine is a Raspberry Pi or another single board computer check out these operating systems for getting docker up and running

- [Casa OS](https://github.com/IceWhaleTech/CasaOS)
- [Hypriot OS](https://blog.hypriot.com/downloads/)

## Running PiHole & Unbound (Linux/Mac OS)

- Clone this repository

    ```
    # linux/mac commands

    # Using https. Use SSH if contributing to the project 
    $ git clone https://github.com/Audiosutras/pihole-unbound-dockercompose.git

    # make pihole-unbound-dockercompose our working directory
    $ cd pihole-unbound-dockercompose
    ```

- Download repository as a zip file

    ```
    # linux/mac commands

    # download zip file of master branch
    $ wget -O pihole-unbound-dockercompose-master.zip https://github.com/Audiosutras/pihole-unbound-dockercompose/archive/refs/heads/master.zip

    # unzip the zip file
    $ unzip pihole-unbound-dockercompose-master.zip

    # make pihole-unbound-dockercompose our working directory
    $ cd pihole-unbound-dockercompose-master
    ```

1. Create an `.env` file in the same directory as the `docker-compose.yml` file

    ```
    # linux/mac commands

    $ ls
    docker-compose.yml  READMe.md

    # opens nano editor
    $ touch .env && sudo nano .env
    ```

    Copy & Paste the code block below. Replace `<super secret password for logging into pihole dashboard>` with your password. Also select your current 

    ```.env
    PIHOLE_PWD=<super secret password for logging into pihole dashboard>
    # Switch With your local TimeZone, ex: PIHOLE_TZ=America/New_York 
    PIHOLE_TZ=<timezone>
    ```

    Press `CTRL + X` then `Y` and then `ENTER` to exit the editor.

2. Run the project detached as a background process. 

    If you are running this project on Ubuntu (and maybe Fedora) there are [additional steps](#ubuntu-additional-steps) that need to be completed before continuing with step 2.

    ```
    $ docker-compose up -d
    ```

    Pihole and Unbound will restart automatically unless explicitly stopped by the user.


3. Get the IP Address of the Pihole instance

    - If wanting to use Pihole on the machine you just installed it on without local 
    network coverage, the IP address you will use for your DNS server is `127.0.0.1` (localhost).
    - For local network coverage, you will need to know the local IP address for the machine you placed Pihole on. Get that on linux by running
    ```
        # linux/mac commands

        # make sure to write down the first entry in this list
        $ hostname -I
    ```

4. Confirm PiHole is using Unbound as the upstream DNS

    - Navigate to `http://<ip-address>/admin` replacing `<ip-address>` with the address you obtained in step 3.
    - Input the `PIHOLE_PWD` password you chose in step 1 to access the admin
    - Navigate to `Settings`, click on the `DNS` tab. Under `Upstream DNS` `Custom 1 (IPv4)` you should see checked `10.1.1.3#53`. This is `Unbound`'s internal IP address. 
    - You can uncheck this and use any of the other upstream dns servers like `Cloudflare` and `Quad9` whenever you want to.

5. Start using Pihole - [Article: Configure Clients to use Pihole](https://discourse.pi-hole.net/t/how-do-i-configure-my-devices-to-use-pi-hole-as-their-dns-server/245)

    - On the Pihole installed machine you can navigate to Wifi or Network Settings and update the `DNS` section for your internet connection by inputing `127.0.0.1` as the value for this section.
    - For local network coverage of all devices you will need to update Static DNS settings found in your router's admin page. You will set 
    the DNS value to the local IP address you retrieved in step 3.

    For more Information see the article linked above for step 5.

### Ubuntu Additional Steps

In order to proceed you will need to update `systemd-resolved` or disable it.

- [Offical Pihole Solution for Ubuntu & Fedora - Update It](https://github.com/pi-hole/docker-pi-hole/#installing-on-ubuntu-or-fedora)

- [The Unoffical Solution for the Streets - Disable It](https://askubuntu.com/questions/907246/how-to-disable-systemd-resolved-in-ubuntu).

Here are the steps for the unoffical solution.

    ```
    # The Unoffical Solution for the Streets

    $ sudo systemctl disable systemd-resolved
    $ sudo systemctl stop systemd-resolved
    $ sudo nano /etc/NetworkManager/NetworkManager.conf 
    ```

Add `dns=default` under `[main]` section in `/etc/NetworkManager/NetworkManager.conf`


    ```
    # inside '/etc/NetworkManager/NetworkManager.conf'
 
    [main]
    ...
    dns=default
    ```
Press `CTRL + X` then `Y` and then `ENTER` to exit the editor.

Delete the sysmlink `/etc/resolv.conf`

    ```
    $ rm /etc/resolv.conf
    ```

Restart NetworkManager

    ```
    $ sudo systemctl restart NetworkManager
    ```

Now we can proceed back to step 2 in [Running PiHole & Unbound](#running-pihole--unbound).

## More Useful Resources & Articles

### Resources

- [Pi-hole website](https://pi-hole.net/)
- [Setup Pihole Docker (Official)](https://github.com/pi-hole/docker-pi-hole/#running-pi-hole-docker)
- [Unbound as an Upstream DNS](https://nlnetlabs.nl/projects/unbound/about/)

### Articles 

- [PiHole Docker - Pi My Life UP](https://pimylifeup.com/pi-hole-docker/)
- [Run PI Hole In Localhost And Some Extras - Medium](https://thanosmour-tk.medium.com/run-pi-hole-in-localhost-and-some-extras-4b50e76611e6)
- [How To Install Unbound and Pi-hole in Docker using Docker Compose](https://www.reddit.com/r/docker/comments/rbgrm8/how_to_install_unbound_and_pihole_in_docker_using/)

## Additional Information

See [Wiki](https://github.com/Audiosutras/pihole-unbound-dockercompose/wiki) for additional information
