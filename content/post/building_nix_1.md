+++ 
title = "Building Nix: Step 1"
date = 2023-01-11
author = "Jeremy Borgman"
tags = ['Nix']
+++

## Introduction
TODO: Fix language. this is just notes for now
This series will walk through how to build a repo that consolidates macos and linux config files.

### Starting point
Install nixos in a VM and copy out the auto generated configuration.nix and hardware-configuration.nix. 
Make sure the following changes are made from the default:

1. Make sure git is included as an installed package (required for flakes)
2. Enable flakes since they are currently experimental: 
```
nix.settings.experimental-features = ["nix-command" "flakes" ];
```

```
repository
├── configuration.nix
├── hardware-configuration.nix
```

{{<content "post/nix_tutorial/post1/configuration.nix" >}}
{{<content "post/nix_tutorial/post1/hardware_configuration.nix" >}}

## Converstion to Flake
The next step in this journey is to make as few changes as possible to convert this to a flake setup for our "real" starting point. Notice
there is a new top level file named 'flake.nix'. 'configuration.nix' has been minimally changed and 
hardware-configuration.nix is the same.

 ```
repository
├── configuration.nix
├── flake.nix
├── hardware-configuration.nix
```


{{<content "post/nix_tutorial/post1/flake.nix" >}}
{{<content "post/nix_tutorial/post1/configuration.nix" >}}