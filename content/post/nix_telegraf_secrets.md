+++ 
title = "Nix Secrets and Telegraf"
date = 2023-09-21
author = "Jeremy Borgman"
tags = ['Nix', 'sops-nix', 'Telegraf']
+++

## Introduction
I wanted to set up telegraf on my nixos desktop, but needed to solve the problem of storing the InfluxDB token. This documents how I solved the problem and what issues I faced.


## Telegraf setup
The initial telegraf setup was pretty straightforward. There is a telegraf nixos service available in the nixos [package repository](https://github.com/NixOS/nixpkgs/blob/master/nixos/modules/services/monitoring/telegraf.nix)

For an example of how I've configured telegraf my nixos repository can be [referenced](https://github.com/borgmanJeremy/nix_config/blob/3ca4ec3f0abd6a39a958c9a1954309860cfe9a4e/hosts/desktop/configuration.nix#L33C24-L33C24).

The problem I ran into was I needed a way to store the telegraf token without it appearing as plain text in my repository.

## sops-nix
When one researches how to store secrets in Nix, one of the top results is [sops-nix](https://github.com/Mic92/sops-nix). This is a very cool project for encrypting secrets with either GPG or age. However, there are some subtleties that were hard to understand as a newcomer. 

### Getting Started
After installing sops-nix (which is well documented on the github page), the first step is to generate a key. There are a few ways to do this, but one thing that seemed appealing was to generate an age key from my ssh key.

The first issue I encountered was while trying to use my SSH key stored on my Yubikey. My regular ssh key is of type id_ed25519_sk. This lets me store the private key on the yubikey without using a GPG key for ssh. The problem is ed25519_sk is [not yet supported in sops](https://github.com/getsops/sops/issues/1103)


### Generating a new key
Since that did not work I generated a new ssh key to use with sops. after that I generated an age key from the ssh key. This is done with the following command (documented on the sops-nix page as well):

```bash
$ nix-shell -p ssh-to-age --run "ssh-to-age -private-key -i ~/.ssh/id_ed25519 > ~/.config/sops/age/keys.txt"
```


### Initial sops configuration
Now that there is an age key to use we need to configure sops to use it. This is done by creating a sops-nix configuration file. This is done by creating a file adjacent to flake.nix with the following contents:

```yaml
keys:
  - &admin_jeremy age1jdquv26e5w7y7vt26zsfzusd36wgjvkq33t5vkvt2vpnczqah55qeqyy9c
creation_rules:
  - path_regex: secrets/[^/]+\.(yaml|json|env|ini)$
    key_groups:
    - age:
      - *admin_jeremy
```

The first item under keys is defining a user named admin_jeremy, and linking that to the public age key that was placed in .config/sops/age/keys.txt. The creation_rules section is defining a regex that will match any file in the secrets directory with the extensions yaml, json, env, or ini. The key_groups section is defining which keys can be used to encrypt files that match the regex. In this case, only the admin_jeremy key can be used to encrypt files in the secrets directory.

### Storing Secrets

Now that sops is configured we can start storing secrets. For my use case I wanted to keep my secrets in a yaml file so I generated a new file with:

```bash
$ nix-shell -p sops --run "sops secrets/example.yaml"
```

This will open an editor with a generic yaml file that can be modified. I modified mine to include my influxdb token:

To edit this file again the same sops command can be used.

```yaml
influx_token: influx_token=<my_influx_token>
```


## Configure telegraf to us this new key

The first thing to configure is sops. Here is an example of how mine is configured:

```
  sops = {
    secrets = {
      influx_token = {};
    };
    defaultSopsFile = ../../secrets/example.yaml;
    age.sshKeyPaths = ["/home/jeremy/.ssh/sopsnix"];
  };
```

This is indicated to sops that it should look in ../../secrets/example.yaml and decrypt the influx_token.

Then the telegraf service can be configured to look for an environment file which it will load before the service.

```
 services.telegraf = {
   ...
   ...
    environmentFiles = [config.sops.secrets.influx_token.path];
    ...
    ...
    extraConfig = {
      outputs = {
        influxdb_v2 = {
            ...
            ...
          token = "\${influx_token}";
          ...
        };
      };
```

What this will basically do is decrypt the influx token and place it in the nix store only readable by root. Then telegraf is instructed to look at the **path** and source the file as environmental variables. Finally we configure telegraf to use the influx_token environmental variable as the token for the influxdb_v2 output.

## Conclusion
This is a bit of a roundabout way to load the token, but as of right now there is no way for sop-nix to decrypt the key and substitute it directly into the telegraf configuration file.