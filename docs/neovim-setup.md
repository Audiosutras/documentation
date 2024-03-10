---
category: Command Line
---

# Neovim IDE Setup

[Neovim](https://neovim.io/) is a hyper-extensible [Vim](https://www.vim.org/)-based text editor. Through additional configuration provided by plugins and a window manager like [tmux](https://github.com/tmux/tmux/wiki) we can create a powerful Integrated Development Environment (IDE) comparable to [VS Codium](https://vscodium.com/).

## Table of Contents

- [Alacritty](#alacritty)
- [Neovim](#neovim)
- [Tmux](#tmux)
- [Patched Fonts](#patched-fonts)

## Alacritty

To get the most out of neovim and tmux you will want to use a terminal that can utilizes your graphics card. I recommend using [alacritty](https://github.com/alacritty/alacritty/blob/master/INSTALL.md) a cross-paltform, OpenGL terminal emulator.

## Neovim

### Installation

Installation methods for your particular operating system can be found [here](https://github.com/neovim/neovim/blob/master/INSTALL.md).

For Debian

```bash
$ sudo apt-get install neovim python3-neovim
```

For Ubuntu

```bash
$ sudo apt install neovim python3-neovim
```

For Fedora

```bash
$ sudo dnf install -y neovim python3-neovim
```

For Arch Linux

```bash
$ sudo pacman -S neovim
```

### Configuration

I recommend installing [LazyVim](https://www.lazyvim.org/installation) as your package manager. Installation is completed by cloning the repository from github.

Backup existing plugins if you are just switching to LazyVim

```bash
# required
$ mv ~/.config/nvim{,.bak}

# optional but recommended
$ mv ~/.local/share/nvim{,.bak}
$ mv ~/.local/state/nvim{,.bak}
$ mv ~/.cache/nvim{,.bak}
```

Clone the starter or copy my config

**starter** if you would like to configure lazy.vim yourself and follow the docs

```bash
$ git clone https://github.com/LazyVim/starter ~/.config/nvim
```

Use my config

```bash
$ git clone https://github.com/Audiosutras/nvim.git ~/.config/nvim
```

Remove the `.git` folder, so you can add it to your own repo later

```bash
$ rm -rf ~/.config/nvim/.git
```

Start Neovim

```bash
$ nvim
```

You can enable/disable plugins with `:LazyExtras` and using `x` to select plugins from the menu. Press `:q` to exit. Make sure to familiarize yourself with standard vim commands using `:help`. You learn more about plugins in use by seeing what plugins are enabling and navigating to there git repositories.

## tmux

[tmux](https://github.com/tmux/tmux/wiki) is a terminal multiplexer. It lets you switch easily between several programs in one terminal, detach them (they keep running in the background) and reattach them to a different terminal.

## Patched Fonts

In order for icons to shows up with the nvim editor 9 out of 10 times you will need a patched font. These are fonts that include the icons that are often times missing from other sources. You can download a patched font of your choice over at [Nerd Fonts](https://www.nerdfonts.com/).

- Create or ensure `~/.local/share/fonts` directory exists on your file system
- Copy the link address for the font you want to download from the `Download` button and the Command

```bash
$ wget https://github.com/ryanoasis/nerd-fonts/releases/download/v3.1.1/AurulentSansMono.zip ~/.local/share/fonts/
```

- `unzip` the file and check the file extension. The file extension could end in `otf` or `ttf`. Create a directory within fonts that matches the file extension(s). Move the unzipped files into the corresponding file extension directory within the fonts directory.

- Update [alacritty](#alacritty) to use the patched. Sticking with our example.

```yaml
$ nvim ~/.config/alacritty/alacritty.yml
# Add the following to the config file
font:
  normal:
    family: AurulentSansMNerdFontMono
    style: Regular
```

[alacritty config cheatsheet](https://sunnnychan.github.io/cheatsheet/linux/config/alacritty.yml.html)
