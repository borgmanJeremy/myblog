+++ 
title = "Nix Update #2"
date = 2022-12-02
author = "Jeremy Borgman"
tags = ['Nix', 'MacOS']
+++

## Introduction
In the [First Nix Post]({{<ref "nix_update_1" >}}) I ended with three questions. I have solved those questions
and wanted to post the answers here.

## How to make a package available on aarch64-darwin
This turned out to be extremely simple. As a easy test case I wanted to make flameshot available
on my macbook. I did so with [this pull request](https://github.com/NixOS/nixpkgs/pull/202228).

It turned out I just needed to add platforms.darwin to the listed platforms. 
```
     homepage = "https://github.com/flameshot-org/flameshot";
     maintainers = with maintainers; [ scode oxalica ];
     license = licenses.gpl3Plus;
-    platforms = platforms.linux;
+    platforms = platforms.linux ++ platforms.darwin;
```

## How do I package a project for nix from scratch
This is a bit more nuanced as is extremely dependent on the type of project. After packaging
a python, rust, c++, and go application. I think the best advice is to find a similar project already
packaged for nix and use that as a starting point. 

## What are flakes and why should I use them
I won't be able to state this better than this [fantastic series of posts](https://www.tweag.io/blog/2020-05-25-flakes/), but my main take aways were:
  - Fully reproducable packages due to a flake.lock file
  - MUCH faster development shells with direnv due to caching.  I have moved most of my shell.nix files to
  a flake.nix. Below are some samples


Overall I'm very much enjoying using both NixOs and nix-shells for development environments.

## Hugo Environment
```
{
description = "Hugo environment";

  outputs = { self, nixpkgs, ... }:
    let

      # Generate a user-friendly version number.
      version = builtins.substring 0 8 self.lastModifiedDate;

      # System types to support.
      supportedSystems =
        [ "x86_64-linux" "x86_64-darwin" "aarch64-linux" "aarch64-darwin" ];

      # Helper function to generate an attrset '{ x86_64-linux = f "x86_64-linux"; ... }'.
      forAllSystems = nixpkgs.lib.genAttrs supportedSystems;

      # Nixpkgs instantiated for supported system types.
      nixpkgsFor = forAllSystems (system: import nixpkgs { inherit system; });

    in {

      devShells = forAllSystems (system:
        let pkgs = nixpkgsFor.${system};
        in {
          default = pkgs.mkShell {
            buildInputs = with pkgs; [
              hugo
];
          };
      });

      devShell = forAllSystems (system: self.devShells.${system}.default);
    };
}

```

## Golang Environment
```
{
description = "golang dev environment";

  outputs = { self, nixpkgs, ... }:
    let

      # Generate a user-friendly version number.
      version = builtins.substring 0 8 self.lastModifiedDate;

      # System types to support.
      supportedSystems =
        [ "x86_64-linux" "x86_64-darwin" "aarch64-linux" "aarch64-darwin" ];

      # Helper function to generate an attrset '{ x86_64-linux = f "x86_64-linux"; ... }'.
      forAllSystems = nixpkgs.lib.genAttrs supportedSystems;

      # Nixpkgs instantiated for supported system types.
      nixpkgsFor = forAllSystems (system: import nixpkgs { inherit system; });

    in {

      devShells = forAllSystems (system:
        let pkgs = nixpkgsFor.${system};
        in {
          default = pkgs.mkShell {
            buildInputs = with pkgs; [
              go
              gotools
              gopls
              go-outline
              gocode
              gopkgs
              gocode-gomod
              godef
              golint
];
          };
      });

      devShell = forAllSystems (system: self.devShells.${system}.default);
    };
}
```

## Rust Environment
```
{
description = "rust dev environment";

  outputs = { self, nixpkgs, ... }:
    let

      # Generate a user-friendly version number.
      version = builtins.substring 0 8 self.lastModifiedDate;

      # System types to support.
      supportedSystems =
        [ "x86_64-linux" "x86_64-darwin" "aarch64-linux" "aarch64-darwin" ];

      # Helper function to generate an attrset '{ x86_64-linux = f "x86_64-linux"; ... }'.
      forAllSystems = nixpkgs.lib.genAttrs supportedSystems;

      # Nixpkgs instantiated for supported system types.
      nixpkgsFor = forAllSystems (system: import nixpkgs { inherit system; });

    in {

      devShells = forAllSystems (system:
        let pkgs = nixpkgsFor.${system};
        in {
          default = pkgs.mkShell {
            buildInputs = with pkgs; [
              rustc
              cargo
              gcc
              rustfmt
              clippy
            ];
          };
      });

      devShell = forAllSystems (system: self.devShells.${system}.default);
    };
}
```