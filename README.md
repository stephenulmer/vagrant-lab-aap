# Ansible Automation Platform Lab Implemented with Vagrant

Creates an AAP server and some managed nodes for experimentation.

## Prerequsites
The following files need to be placed into the `bundle` directory:

- an AAP containerized installation bundle
- a license manifest file that has entitlements for AAP

Both of these can be obtained from Red Hat with a free Developer License.


## Lab Creation

The lab can be launched with the command:

    vagrant up

If the `vagrant-registration` is installed and you want `subscription-manager` to configure access to Red Hat software repos, set the `RH_USERNAME` and `RH_PASSWORD` environment variables.


## Details
This version of the lab installs an AAP server and several nodes to manage. The AAP server is installed from the bundle file mentioned above.

The following AAP resources are created:

- a machine credential for access to the managed nodes
- an inventory of the managed nodes
