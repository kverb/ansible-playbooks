# Ansible Playbooks for homelab containers

This repo contains ansible files to support automated installation and management of homelab services.
This is 99.999% only relevant to me and my configuration, but it's here for inspiration to others and to remind myself how i set all of this up.

## Services

- nzbget


## how to use these

### Base Assumptions and Requirements

These are all installed on base arch linux LXC containers, within proxmox. While it's very straightforward to start a fresh arch linux container in proxmox, there are some boostrapping steps required to get a usefully-functioning system. I'll try to summarize them here:

- create an arch LXC container on proxmox. Include your ssh public key during creation.
- open the shell or console in PVE and enable sshd: `systemctl enable --now sshd` . You can then connect over ssh or proceed over the PVE shell / console.
- run `pacman-key --init`
- you'll need to enable the latest mirrorlist from [here](https://archlinux.org/mirrorlist/all/https/) (uncomment the US section at the bottom) and copy it to `/etc/pacman.d/mirrorlist`
- then:
```
trust extract-compat  # see https://archlinux.org/news/ca-certificates-update/ for why
pacman-key --populate archlinux
pacman -Syy --noconfirm archlinux-keyring
```

- then you can update the system packages with `pacman -Syyu --noconfirm`
- install python (required for ansible): `pacman -S python`

 Now you can run ansible via ssh, but be sure to include any new containers in your ansible inventory.

 These containers are part of an NFS-based file sharing system. They are currently configured with bindmounts to directories prefixed with /nfs in each container. It's not a requirement that the binding is actually an nfs share; just that the mount is actually somwhere useful to you and the mount is at /nfs

### ansible 

Clone this repository to where you want to run ansible from. Make sure you have ansible installed, and your inventory file is properly configured.

Normally, I would install the 3rd-party role requirements: `ansible-galaxy install -r requirements.yml` and then modify them locally to suit my needs.
I've included my own custom patched versions here in the 'roles' directory.

the [kewlfft/ansible-aur](https://github.com/kewlfft/ansible-aur) module is a dependency for installing `yay`: 
`ansible-galaxy collection install kewlfft.aur`

### nzbget

Installs and configures nzbget on the target host. If you look at the playbook, you'll see it installs some commonly-used packages and yay.

create an encrypted, secret variables file using `ansible-vault`. The initial contents should be a simple key: value yaml file. Here's an example:

```
$ cat nzbget.secrets

newsgroup_ninja_username: flynn 
newsgroup_ninja_password: supersecretpassword 
tweaknews_username: flynn 
tweaknews_password: othersecretpassword

$ ansible-vault encrypt nzbget.secrets
```
The encrypt step will prompt for a password to encrypt the file with. You will be prompted to enter this when decrypting.

`ansible-playbook containers/nzbget.yml --limit <nzbget-host-from-inventory> -e @secrets/nzbget.secrets --ask-vault-pass`
