# Gsoc 2024 Report
## Project Goals
- Organization: [CCExtractor Developement](https://summerofcode.withgoogle.com/programs/2024/organizations/ccextractor-development)

- Project Link: [Create nix derivations for Regolith Linux ](https://summerofcode.withgoogle.com/programs/2024/projects/kWEY5sJj)

## Project Description
The project aims to integrate Regolith Linux (mordern desktop enviroment build on top of GNOME with i3 and sway window managers) into NixOS, an operting system with a declarative and reproducible configuration model. 

The goal of this project was to create a bootable session out of nix derivations of required applications/bash-scripts/systemd services and config files. So that finaly the whole package should be linked to a nixos module using which regolith linux can be installed and customized with a single nixos configuration option (`desktopEnvironment.wayland.regolith.enable=true;`)

## Final Outcome
Final Outcome of the project is to have a completely working regolith desktop environment completely modifiable via nix config. Example :
```
wayland.regolith = {
enable =true;
gnome-control-center-integration=true;
launcher= pkgs.ilia;
notfication-manager= pkgs.rofication;
color-scheme= "gruvbox-dark";
gdm.enable=true;
...
...
};
```

Doing so would include creating nixos options for all of these settings by
1. Porting all the singular components by creating nix derivations.
2. Creating Nixos Options for these elements.
3. Combining options to a singular flake.nix file

## What is a nix derivation?
A derivation is essentially a recipe that describes how to build a particular package, including its dependencies, build instructions, environment variables, and other relevant details.

Nix derivations are used to build binaries but in a completely sandboxed enviroment with only the declared dependencies. Nixos does not follow the traditional Filesystem Hireracrchy Standard because of its fundamental differences. But it creates it own file system `/run/current-system/sw` here. 

Using derivations we essentially want to map the outputs of these derivations to this file system structure. 

Nix provides its own set of tools and methods to help create perfectly reproducible packages. More explanation here - https://nix.dev/

## Work Done
Currently I have created working derivations for 
- regolith-session
- trawld
- sway-regolith
- libtrawldb
- regolith-sway-root-config
- regolith-look-default
- regolith-displayd
- regolith-inputd
- regolith-powerd
- regolith-wm-config
- rofication
- ilia
- remontoire
- i3xrocks
- xrescat
 
### Regolith-Session
I have created a bootable version of regolith-session-wayland on nixos.
Repository linking regolith-session working nix flake file - https://github.com/sandptel/regolith-nix

#### How to test/use the bootable version of regolith-session
1. Add experimental features option to nixos configuration option
`nix.settings.experimental-features = [ "nix-command" "flakes" ];`

2. Link the regolith-nix url(https://github.com/sandptel/regolith-nix) to flake.nix and add the nixos module `inputs.regolith.nixosModules.regolith`
```
{
  description = "A very basic flake";
  inputs = {
    nixpkgs.url = "github:nixos/nixpkgs/nixos-24.05";
     regolith.url = "path:/home/roronoa/Documents/regolith-nix";   
  };
  outputs = { self,nixpkgs,home-manager, ... }@inputs: 
  let 
  system = "x86_64-linux";
  pkgs = nixpkgs.legacyPackages.${system};
  in
  {
    nixosConfigurations.roronoa = nixpkgs.lib.nixosSystem {
      specialArgs = {inherit system inputs;};
      modules = [
        ./configuration.nix
        ./hardware-configuration.nix
        inputs.regolith.nixosModules.regolith
      ];
    };
  };
}
```

2. launch the regolith-session-wayland binary to launch the session.
```
$ regolith-session-wayland
```

#### Testing Single Packaged applications on Nixos/Any other distribution
##### Nixos 
` nix run github:sandptel/<application-name>`
##### Other Distributions
1. Intalling Nix
```
sh <(curl -L https://nixos.org/nix/install) --no-daemon
 . ~/.nix-profile/etc/profile.d/nix.sh
export NIX_CONFIG="experimental-features = nix-command flakes"
```
2. Running the application
``` nix run nix run github:sandptel/<application-name>```

##### <application name>

1. regolith-session
2. trawld
3. ilia
4. remontoire
5. i3xrocks
6. xrescat
7. regolith-displayd
8. regolith-inputd
9. regolith-powerd
10. sway-regolith

These applications does not have a binary or have multiple binaries or have config files only.
11. libtrawldb 
12. regolith-sway-root-config
13. regolith-look-default
14. regolith-wm-config
15. rofication

Therefore you can create devshells from the regolith-nix flake file to
test all the application singularly.

`nix develop github:sandptel/regolith-nix` 

This will create a tmp devShell with all the derivations present.

