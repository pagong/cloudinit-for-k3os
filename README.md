Cloud Init ISO
==============

Many premade cloud images (e.g. Fedora Cloud, Ubuntu Cloud) use some form of 
cloud-init to set instance/user metadata, such as hostnames and SSH keys. This 
works well when used in a cloud infrastructure such as EC2 or OpenStack that can 
seed this data, but not so well when used for local VMs, or an out-of-the-box 
XenServer installation.

However, [cloud-init supports an CD/ISO datasource][1], which can be loaded 
whether running the machine locally in KVM, VirtualBox, or on a XenServer host. 

This repository and guide aims to make this task a lot easier by pre-seeding a 
template and script. It draws heavily on existing resources ([2][2], [3][3]).

## Special K3os version
[K3os][5] is Rancher Labs' bespoke operating system for running the tiny K8s distro [k3s][6].
When configured to use `cloud-init` with `k3os.data_sources=cdrom`, the embedded version
of linuxkit/metadata is requiring a file called `config` in the top level of the ISO.

That's why this fork is needed: create a hard link from `user-data` to `config`
before making the ISO image. (Bonus info: for Suse Linux use `mkisofs`.)

Example: run `k3os-build.sh K3os-091-a` to generate an ISO file called `cidata-K3os-091-a.iso`

## Dependencies
You need `genisoimage` installed. For RHEL, CentOS, Fedora, Oracle Linux, etc.,
`sudo dnf install genisoimage` (use `yum` if you are on RHEL/CentOS/OL 7). For 
Ubuntu, Debian, and derivatives thereof, `sudo apt install genisoimage`.

## Usage

1. Clone this repository. (optional: maybe create and checkout your own branch?)
2. Modify the `meta-data` YAML file to specify your `instance-id` and `local-hostname`.
3. Modify the `user-data` YAML file -- according to [cloud-config syntax][4] -- to 
   specify a password and/or SSH keys. The cloud image determines the default user's 
   login name, but you can override that according to the [cloud-config documentation][4].
4. (optional) Commit your changes in git. This helps the build script name your ISO.
5. Build the ISO using `./build.sh`. You can either specify an output filename as the 
   first parameter (e.g. `./build.sh output-file.iso`), or you can let the script decide 
   on the filename. If you are working inside a git repository, the build script should 
   name your file after the branch and commit hash, such as 
   `frost-init-20141228.4d48bab28b6d8f9c43f7b6e36238ef6863b41e90.iso`.
6. If everything went well, attach the ISO file to your VM by methods 
   conventional to your virtualization hypervisor.
7. Boot the VM!

## License

Some parts of this project are clearly not original. However, the bash script and 
any exemplary portions of the config files are hereby licensed under the MIT License; 
see `LICENSE`.

[1]: https://cloudinit.readthedocs.io/en/latest/topics/datasources/nocloud.html
[2]: https://www.technovelty.org/linux/running-cloud-images-locally.html
[3]: http://www.projectatomic.io/blog/2014/10/getting-started-with-cloud-init/
[4]: https://cloudinit.readthedocs.io/en/latest/topics/examples.html
[5]: https://github.com/rancher/k3os
[6]: https://github.com/rancher/k3s
