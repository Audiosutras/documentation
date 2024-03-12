---
category: Hosting
---

# Pi-hole & Unbound Setup Using Docker Compose

## Article Sections

- [Background](#background)
- [DNS Resolver CheatSheet](#dns-resolver-cheatsheet)
- [Download The Latest Release](#download-the-latest-release)
- [Prerequiste Installation](#prerequiste-installation)
  - [Docker & Docker Compose](#docker--docker-compose)
  - [Direnv](#direnv)
- [Running PiHole & Unbound (Linux / Mac OS)](#running-pihole--unbound-linuxmac-os)
  - [Ubuntu Additional Steps](#ubuntu-additional-steps)
- [More Useful Resources & Articles](#more-useful-resources--articles)
  - [Resources](#resources)
  - [Articles](#articles)

## Background

[Pi-hole](https://docs.pi-hole.net/) is a DNS sinkhole that is effective at blocking ads and malware by closing connections to blacklisted domains before a client can connect to them. [Unbound](https://nlnetlabs.nl/projects/unbound/about/) is a validating, recursive, caching DNS resolver that increases the privacy of its users.

This project provisions two docker containers on a user's chosen machine that always run unless stopped; one for Pihole and the other Unbound. When PiHole is configured to use Unbound as its only upstream DNS server it cuts Google, Cloudflare, and other DNS providers out from having a record of the domains you have requested. In simple terms, this means the sites you have visited. However note that your Internet Service Provider will still be able to access your DNS history without any obfuscation. For more information I found [this](https://www.reddit.com/r/pihole/comments/ydkkup/what_are_the_benefits_to_unbound/) reddit thread helpful.

The benefits of running pi-hole and unbound in docker containers are many. To speak to a few, it is the ability to run this software across operating system (linux, mac, windows) and across computing devices. You can benefit from adblocking and malware protection on your devices at home by running pihole on a raspberry pi attached to your home network. You can also have this benefit on your machine when connected to an external network like hotel or airport wifi.

Let's get started.

## DNS Resolver CheatSheet

The table below documents the port and internal IP address of the custom dns resolvers that this instance of Pihole comes pre-configured with.

| DNS Resolver | Internal IP Address |
| ------------ | ------------------- |
| Unbound      | 10.1.1.3#53         |

## Download The Latest Release

- Download the [latest release](https://github.com/Audiosutras/pihole-dockercompose/releases) (Currently `v1.0.3`)

  ```bash
  -> wget https://github.com/Audiosutras/pihole-dockercompose/archive/refs/tags/v1.0.3.tar.gz
  -> tar -xvf v1.0.3.tar.gz
  -> rm v1.0.3.tar.gz && cd pihole-dockercompose-1.0.3
  ```

## Prerequiste Installation

### Docker & Docker Compose

This project depends on having Docker and Docker Compose installed on the machine
you plan to run Pihole on. Ensure your machine has at least 4GB of RAM.

To check if `docker` and `docker-compose` is already installed run the following commands from the command line:

```bash
-> docker --version
Docker version 24.0.6, build ed223bc
-> docker-compose --version
docker-compose version 1.28.0, build d02a7b1a
```

If nothing was returned when running the above commands follow docker's recommended installation method found here under [Scenario one: Install Docker Desktop](https://docs.docker.com/compose/install/#scenario-one-install-docker-desktop) for your operating system and/or linux distribution. If your machine is a Raspberry Pi or another single board computer check out these operating systems for getting docker up and running

- [Casa OS](https://github.com/IceWhaleTech/CasaOS)
- [Hypriot OS](https://blog.hypriot.com/downloads/)

### Direnv

You can export the environment variables we'll discuss but [direnv](https://direnv.net/docs/installation.html) is a great utility to use to have the environment variables automatically exported for you when this project is in your working directory.

For ubuntu-based systems:

```bash
$ apt install direnv
```

Via homebrew:

```bash
$ brew install direnv
```

You may need to re-open your terminal shell here. Now let's configure our shell.

For bash:

```bash
$ echo 'eval "$(direnv hook bash)"' >> ~/.bashrc
$ source ~/.bashrc
```

For zsh:

```bash
$ echo 'eval "$(direnv hook zsh)"' >> ~/.zshrc
$ source ~/.zshrc
```

## Running PiHole & Unbound (Linux/Mac OS)

- [Download the latest release](#download-the-latest-release)

1. Create an `.envrc` file in the same directory as the `docker-compose.yml` file

   ```bash
   -> ls
   docker-compose.yml  READMe.md
   -> touch .envrc && vim .envrc # opens vim editor
   ```

   Copy & Paste the code block below into the `vim editor`.

   - Replace `<super secret password for logging into pihole dashboard>` with your password.
   - Replace `<timezone>` with your timezone. For example `America/New_York`
   - Replace `<dns addresses>` with the upstream dns you would like to use. If you would like to use `Unbound` write `'10.1.1.3#53'` or if you would like to use `Cloudflare` and `Quad 9` write `'1.1.1.1;9.9.9.9'` for example. There are more options available in the pihole admin than this. This just selects the default upstream providers that will be checked in the admin on system start.

   ```envrc
   export PIHOLE_PWD=<super secret password for logging into pihole dashboard>
   # Switch With your local TimeZone, ex: export PIHOLE_TZ=America/New_York
   export PIHOLE_TZ=<timezone>
   export PIHOLE_DNS=<dns addresses>
   ```

   `:wq` then `ENTER` to exit the vim editor.

   Now allow `direnv` to automatically export environment variables by running:

   ```bash
   $ direnv allow .
   ```

   Each time changes are made to the `.envrc` file this command will need to be run.

2. Run the project detached as a background process.

   If you are running this project on Ubuntu (and maybe Fedora) there are [additional steps](#ubuntu-additional-steps) that need to be completed before continuing with step 2.

   ```bash
   -> docker-compose up -d
   ```

   Pihole and Unbound will restart automatically unless explicitly stopped by the user.

   If you are running this project on a raspberry pi or a device using the arm64 architecture and you plan on using unbound, you will need to modify the `unbound.image` value in the `docker-compose.yml` file. Change `mvance/unbound:latest` to `mvance/unbound-rpi:latest`. Save the file and then run `docker-compose up -d`.

3. Get the IP Address of the Pihole instance

   - If wanting to use Pihole on the machine you just installed it on without local
     network coverage, the IP address you will use for your DNS server is `127.0.0.1` (localhost).
   - For local network coverage, you will need to know the local IP address for the machine you placed Pihole on. Get that on linux by running

   ```bash
       # make sure to write down the first entry in this list
   -> hostname -I
   ```

   **Note**: The machine you placed pihole on would benefit from having a [static IP address](https://www.tomshardware.com/how-to/static-ip-raspberry-pi)

4. Confirm PiHole is the selected upstream DNS servers selected in Step 1

   - Navigate to `http://<ip-address>/admin` replacing `<ip-address>` with the address you obtained in step 3. If `127.0.0.1` append the port as so: `http://127.0.0.1:8080/admin`
   - Input the `PIHOLE_PWD` password you chose in step 1 to access the admin
   - Navigate to `Settings`, click on the `DNS` tab. Under `Upstream DNS` `Custom 1 (IPv4)` you should see checked `10.1.1.3#53`. This is `Unbound`'s internal IP address.
   - You can uncheck this and use any of the other upstream dns servers like `Cloudflare` and `Quad9` whenever you want to.
   - _Note_ Unbound being checked is contigent upon what upsteam dns providers you selected for `PIHOLE_DNS` in Step 1. If you chose `"1.1.1.1;9.9.9.9"` you will see `Cloudflare` and `Quad 9` checked.

5. Start using Pihole - [Article: Configure Clients to use Pihole](https://discourse.pi-hole.net/t/how-do-i-configure-my-devices-to-use-pi-hole-as-their-dns-server/245)

   - On the Pihole installed machine you can navigate to Wifi or Network Settings and update the `DNS` section for your internet connection by inputing `127.0.0.1` as the value for this section.
   - For local network coverage of all devices you will need to update Static DNS settings found in your router's admin page. You will set
     the DNS value to the local IP address you retrieved in step 3.

   For more Information see the article linked above for step 5.

### Ubuntu Additional Steps

In order to proceed you will need to update `systemd-resolved` or disable it.

- [Offical Pihole Solution for Ubuntu & Fedora - Update It](https://github.com/pi-hole/docker-pi-hole/#installing-on-ubuntu-or-fedora)

- [The Unoffical Solution for the Streets - Disable It](https://askubuntu.com/questions/907246/how-to-disable-systemd-resolved-in-ubuntu).

Here are the steps for the unoffical solution

```bash
    # The Unoffical Solution for the Streets
-> sudo systemctl disable systemd-resolved
-> sudo systemctl stop systemd-resolved
-> sudo nano /etc/NetworkManager/NetworkManager.conf
```

Add `dns=default` under `[main]` section in `/etc/NetworkManager/NetworkManager.conf`

```bash
# inside '/etc/NetworkManager/NetworkManager.conf'
[main]
...
...
dns=default
...
```

`:wq` then `ENTER` to exit the vim editor.

Delete the sysmlink `/etc/resolv.conf`

```bash
-> rm /etc/resolv.conf
```

Restart NetworkManager

```bash
-> sudo systemctl restart NetworkManager
```

Now we can proceed back to step 2 in [Running PiHole & Unbound](#running-pihole--unbound-linuxmac-os).

## More Useful Resources & Articles

### Resources

- [Pi-hole website](https://pi-hole.net/)
- [Setup Pihole Docker (Official)](https://github.com/pi-hole/docker-pi-hole/#running-pi-hole-docker)
- [Unbound as an Upstream DNS](https://nlnetlabs.nl/projects/unbound/about/)

### Articles

- [PiHole Docker - Pi My Life UP](https://pimylifeup.com/pi-hole-docker/)
- [Run PI Hole In Localhost And Some Extras - Medium](https://thanosmour-tk.medium.com/run-pi-hole-in-localhost-and-some-extras-4b50e76611e6)
- [How To Install Unbound and Pi-hole in Docker using Docker Compose](https://www.reddit.com/r/docker/comments/rbgrm8/how_to_install_unbound_and_pihole_in_docker_using/)
