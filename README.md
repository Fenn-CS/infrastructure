# infra

The infrastructure configuration, currently written in Terraform, Packer and Ansible.

Notes: there is currently a provisioner dependency between this repository and the [devenv repository](https://github.com/permanentorg/devenv). Please ensure that this repository is checked out as well.

## Install

Install [Terraform](https://www.terraform.io/downloads.html), [Packer](https://www.packer.io/downloads) and [Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#installing-ansible-on-debian) to get started with infrastructure provisioning.

```
curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo apt-key add -
sudo apt-add-repository "deb [arch=amd64] https://apt.releases.hashicorp.com $(lsb_release -cs) main"
sudo apt update
sudo apt install terraform packer
sudo pip3 install ansible
```

Ansible can also be installed with your preferred local package manager (e.g. apt).

## Create Images

```
cp .env.template .env # add your AWS access credentials
source .env && cd images && packer build dev.json
```

## Deploy Images

If you've never run Terraform before:

```
cd instances
terraform init
```

After initializing the plugins, run Terraform.

```
terraform apply
```

This command will first show you what actions it plans to execution, and then ask for confirmation. Terraform only manages resources that were created with Terraform.

## Manage SSH access

To onboard a new user with ssh access, create a file in the `ssh` directory. The file name should be the new user's Linux username, and the file contents should be their public key(s).

To offboard a user, remove their ssh file.

In both cases, the AMI needs to be rebuilt and deployed.

```
source .env
cd images/
packer build dev.json
cd ../instances/
terraform apply
```

## Quirks

Q: Why is `ANSIBLE_PIPELINING=True` for the deploy provisioner?
A: Because `aws s3 cp` must be run as the appropriately-credentialed `app_user`. Running an Ansible command as a non-root user requires using Ansible's `become_user`. Ansible normally writes files to a temporary filesystem initially, which requires root, but with pipelining enabled, there is no need to write to this temporary filesystem. If this sounds confusing, that's because it is. See here for more info: https://docs.ansible.com/ansible/latest/user_guide/become.html#risks-of-becoming-an-unprivileged-user