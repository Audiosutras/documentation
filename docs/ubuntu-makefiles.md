# The Ubuntu Makefile Scripts

## New Machine Setup

```Makefile
# Make sure to restart machine after all setup and install rules

# Setup Ubuntu Shell environment 
# PKG: ZSH shell, Git, Curl, Wget, Oh My ZSH
shell_setup:
	sudo apt update
	sudo apt install zsh
	which zsh
	echo "\n Should say '/usr/bin/zsh' \n"
	chsh -s $(which zsh)
	sudo apt install git curl wget
	sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"


# Debian/Ubuntu Package Manager Setup
# PKG: Flatpak & Homebrew
pkg_manager_setup:
	sudo apt install flatpak
	sudo add-apt-repository ppa:flatpak/stable
	sudo apt update
	sudo apt install flatpak
	sudo apt install gnome-software-plugin-flatpak
	flatpak remote-add --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo
	/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
	test -d ~/.linuxbrew && eval "$(~/.linuxbrew/bin/brew shellenv)"
	test -d /home/linuxbrew/.linuxbrew && eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"
	echo "eval \"\$($(brew --prefix)/bin/brew shellenv)\"" >> ~/.zshrc


# Installs Desktop apps using flatpak
# flatpak install flathub <software>
# To run: flatpak run <software>
# Apps - VS Code, Brave Browser, Authy Desktop
# Discord, LibreOffice, Raspberry Pi Imager
install_desktop_apps:
	flatpak install flathub com.visualstudio.code
	flatpak install flathub com.brave.Browser
	flatpak install flathub com.authy.Authy
	flatpak install flathub com.discordapp.Discord
	flatpak install flathub org.libreoffice.LibreOffice
	flatpak install flathub org.raspberrypi.rpi-imager


# Install Packages using Homebrew & Apt
# Pkgs: Docker, NVM (Node Version Manager), Python
# Note - You can install multiple versions of python just 
# with 'sudo apt install python<version>
install_pkgs:
	brew install --cask docker
	brew install nvm
	echo "export NVM_DIR="$HOME/.nvm"" >> ~/.zshrc
	echo "[ -s "$HOMEBREW_PREFIX/opt/nvm/nvm.sh" ] && \. "$HOMEBREW_PREFIX/opt/nvm/nvm.sh"  # This loads nvm" >> ~/.zshrc
	echo "[ -s "$HOMEBREW_PREFIX/opt/nvm/etc/bash_completion.d/nvm" ] && \. "$HOMEBREW_PREFIX/opt/nvm/etc/bash_completion.d/nvm"  # This loads nvm bash_completion" >> ~/.zshrc
	brew install postgresql@16
	sudo apt install software-properties-common
	sudo add-apt-repository ppa:deadsnakes/ppa
	sudo apt update
	sudo apt install python3.12
```

## Update System Packages

```Makefile
# Snap refresh can removed if not already installed
# or installed manually
update_syspkg:
	flatpak update
	snap refresh
	brew update && brew upgrade
	sudo apt update && sudo apt upgrade
```