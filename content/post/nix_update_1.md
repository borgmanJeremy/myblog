+++ 
title = "Nix Update #1"
date = 2022-11-21
author = "Jeremy Borgman"
tags = ['Nix', 'MacOS']
+++

## Introduction
I am one month into testing out Nix. I moved my linux desktop to NixOS and am trying to use Nix
on my M1 Air instead of brew. This post will discuss what is going well and whats not working
one month into this test.

### NixOS Desktop
Using NixOS as my daily driver linux machine is going great. I spent 5 years on arch before moving 
to NixOS and so far am happy with the change. It's great to be able to look at a single config file 
(/etc/nixos/configuration.nix) and understand the state of my entire desktop. I am even able to manage
my Gnome extensions here. I really didn't like that after 6-12 months of using Arch I would loose 
track of what was installed and why, especially for deep dependencies.

The snapshot feature is also great and has recovered my desktop after an issue with my ZFS configuration
prevented the box from booting. 

### nix-shell
The nix-shell command has been amazing for testing out new applications. I love being able to install something
in a temporary shell without cluttering my actual installing. This pairs nicely with dir-env so I can 
have development dependencies for python and node isolation to a project folder. For example for this hugo
blog I have these two files, and then as soon as I enter the project directory all my tools are ready to go:

shell.nix:
```
{ pkgs ? import <nixpkgs> {} }:

pkgs.mkShell {

  buildInputs = [
    pkgs.hugo
  ];

}
```

.envrc:
```
use nix
```

### Nix as a brew replacement
Since NixOs was going to well I wanted to see how far I could push nix on my M1 Air. I wanted to test out
nix-shell as well as nix-darwin: https://github.com/LnL7/nix-darwin.

Overall the experience has not been as smooth as on Linux. The primary issue is many packages are not prebuilt
for aarch64-darwin, and fail when they build from source. This means I cannot drop brew yet. Some packages that 
I have only been able to install via brew include:
- flameshot
- libreoffice
- vlc
- tmux (available for aarch64-darwin but doesn't work)
- darktable

Nix-shell with direnv is working well on the M1 at least. 


## Next Steps
Now that I am more familiar with the Nix ecosystem I want to learn the following in the next month:

- What does it take to make a package available for aarch64-darwin?
- What are flakes and why should I use them?
- How to I package a project for nix from scratch (Flameshot may be a good candidate)