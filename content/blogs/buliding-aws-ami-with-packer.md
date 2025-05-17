---
date: '2025-05-16T14:47:56+05:30'
draft: true
title: 'Building Custom AWS AMIs With Hashicorp Packer'
tags: ["packer", "aws", "hcl"]
cover: 
    image:  images/build-ami-with-packer.jpg
    responsiveImages: true
    linkFullImages: true
---

EC2 is one of the core compute services provided by AWS. In most cases, we use compute services to deploy applications directly or indirectly, and to get the application working we go through the process of installing some packages or dependencies or hardening the server for security reasons. This process can be time-consuming and needs to be repeated for each new server created. This is where HashiCorp Packer comes into the picture - it can be leveraged to build preconfigured images for servers with a pipeline.

## What is Packer?

Packer is a tool for building machine images. It supports multiple platforms, which means you can build machine images from the same source for different platforms, e.g., AWS and Azure.


## Key Features

- **Super fast infrastructure deployment**: With Packer, machine images are preconfigured so virtual machines can be launched in seconds without having to run any configuration scripts.

- **Multi-provider portability**: Packer supports creating identical machine images for different platforms from the same source, making it easier to maintain Packer pipelines to build machine images for different platforms while keeping them compatible across environments.

- **Wide range of plugins/provsioners**: Packer supports many plugins like Ansible, Chef and more which makes it easier to build pipelines.

## Prerequisite

To use Packer to build AMIs on AWS from your local machine, you need to make sure the following tools are installed:

- Packer: [https://docs.docker.com/desktop/install/mac-install/](https://developer.hashicorp.com/packer/install)
- AWS CLI: [https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)
- AWS Access Keys: [https://docs.aws.amazon.com/cli/v1/userguide/cli-authentication-user.html](https://docs.aws.amazon.com/cli/v1/userguide/cli-authentication-user.html)


## Creating Packer template file

A Packer template is a configuration file that defines the image to be built and the steps to build it. Packer templates use HCL language.
First, let's create a folder for keeping all the files:
```bash
mkdir packer && cd packer
```

Let's start by creating a new file `docker-ubuntu.pkr.hcl` and add the following code:

### Packer Block

```hcl
packer {
  required_plugins {
    amazon = {
      version = ">= 1.3.6"
      source  = "github.com/hashicorp/amazon"
    }
    ansible = {
      version = ">= 1.1.3"
      source  = "github.com/hashicorp/ansible"
    }
  }
}
```
The `packer {}` block contains the Packer version to be used, and the `required_plugins {}` block contains the source and version for the plugins used in the template to build the image. Here, the Packer version isn't specified, so it will use the latest version by default. For plugins, we're using the Amazon plugin with version greater than or equal to `1.3.6` and version greater than or equal to `1.1.3` for Ansible.


### Source Block

```hcl
source "amazon-ebs" "ubuntu" {
  ami_name      = "ubuntu-aws-v0.0.1"
  instance_type = "t4g.micro"
  region        = "us-east-2"
  source_ami_filter {
    filters = {
      name                = "ubuntu/images/hvm-ssd-gp3/ubuntu-noble-24.04-arm64-server-*"
      root-device-type    = "ebs"
      virtualization-type = "hvm"
    }
    most_recent = true
    owners      = ["099720109477"]
  }
  ssh_username = "ubuntu"
}
```
The source block consists of two important labels: `builder type` and `name`. In this case, the builder type is `amazon-ebs` and the name is `ubuntu`. Each builder type has its unique set of configuration attributes.

In the above code, `amazon-ebs` specifies a `t4g.micro` instance type and an `ubuntu-noble-24.04` base AMI in the `us-east-2` region. `ssh_username` is also provided so Packer can SSH into the machine once provisioned.


### Build Block

```hcl
build {
  name = "Install docker"

  sources = [
    "source.amazon-ebs.ubuntu"
  ]

  provisioner "ansible" {
    playbook_file = "./install-docker.yml"
  }
}
```
The build block defines the steps that Packer will perform after the machine is launched. We can use multiple `source`s in the build block.
In the above code, we're using the build block to execute an Ansible playbook with the help of the Ansible provisioner to install and configure Docker on an Ubuntu server.

Following is the Ansible playbook used to install and set up Docker:
```yaml
---
- name: Install Docker on Ubuntu
  hosts: default
  become: true
  remote_user: ubuntu
  
  tasks:
    - name: Update apt package index
      apt:
        update_cache: yes

    - name: Install prerequisite packages
      apt:
        name:
          - ca-certificates
          - curl
        state: present

    - name: Create directory for Docker GPG key
      file:
        path: /etc/apt/keyrings
        state: directory
        mode: '0755'

    - name: Download Docker's official GPG key
      get_url:
        url: https://download.docker.com/linux/ubuntu/gpg
        dest: /etc/apt/keyrings/docker.asc
        mode: '0644'

    - name: Add Docker repository
      block:
        - name: Get system architecture
          command: dpkg --print-architecture
          register: system_arch
          changed_when: false

        - name: Get Ubuntu codename
          shell: |
            . /etc/os-release
            echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}"
          register: ubuntu_codename
          changed_when: false

        - name: Add Docker repository to sources
          copy:
            dest: /etc/apt/sources.list.d/docker.list
            content: |
              deb [arch={{ system_arch.stdout }} signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu {{ ubuntu_codename.stdout }} stable
            mode: '0644'

    - name: Update apt package index (after adding Docker repo)
      apt:
        update_cache: yes

    - name: Install Docker packages
      apt:
        name:
          - docker-ce
          - docker-ce-cli
          - containerd.io
          - docker-buildx-plugin
          - docker-compose-plugin
        state: present

    - name: Add ubuntu user to docker group
      user:
        name: ubuntu
        groups: docker
        append: yes
```


## Building the AMI

1. Initialize Packer configuration.
```bash
packer init .
```

2. Format and validate the Packer template
```bash
packer fmt . &&  packer validate .
```

3. Build the image
```bash
packer build docker-ubuntu.pkr.hcl

Install docker.amazon-ebs.ubuntu: output will be in this color.

==> Install docker.amazon-ebs.ubuntu: Prevalidating any provided VPC information
==> Install docker.amazon-ebs.ubuntu: Prevalidating AMI Name: ubuntu-aws-v0.0.1
    Install docker.amazon-ebs.ubuntu: Found Image ID: ami-0684db19b312ab1dc
==> Install docker.amazon-ebs.ubuntu: Creating temporary keypair: packer_682726fe-e009-0c87-60b1-ef42eccb976c
==> Install docker.amazon-ebs.ubuntu: Creating temporary security group for this instance: packer_68272703-29fc-cfab-06c1-dcc12706bb24
==> Install docker.amazon-ebs.ubuntu: Authorizing access to port 22 from [0.0.0.0/0] in the temporary security groups...
==> Install docker.amazon-ebs.ubuntu: Launching a source AWS instance...
    Install docker.amazon-ebs.ubuntu: Instance ID: i-08def819170356c2f
==> Install docker.amazon-ebs.ubuntu: Waiting for instance (i-08def819170356c2f) to become ready...
==> Install docker.amazon-ebs.ubuntu: Using SSH communicator to connect: 3.19.55.112
==> Install docker.amazon-ebs.ubuntu: Waiting for SSH to become available...
==> Install docker.amazon-ebs.ubuntu: Connected to SSH!
==> Install docker.amazon-ebs.ubuntu: Provisioning with Ansible...
    Install docker.amazon-ebs.ubuntu: Setting up proxy adapter for Ansible....
==> Install docker.amazon-ebs.ubuntu: Executing Ansible: ansible-playbook -e packer_build_name="ubuntu" -e packer_builder_type=amazon-ebs --ssh-extra-args '-o IdentitiesOnly=yes' -e ansible_ssh_private_key_file=/var/folders/gx/bbq1rf411c3_23hrqw5brp380000gn/T/ansible-key3453281947 -i /var/folders/gx/bbq1rf411c3_23hrqw5brp380000gn/T/packer-provisioner-ansible1617239015 /Users/mahesh/Desktop/playground/packer/netbird/install-docker.yml
    Install docker.amazon-ebs.ubuntu:
    Install docker.amazon-ebs.ubuntu: PLAY [Install Docker on Ubuntu] ************************************************
    Install docker.amazon-ebs.ubuntu:
    Install docker.amazon-ebs.ubuntu: TASK [Gathering Facts] *********************************************************
    Install docker.amazon-ebs.ubuntu: [WARNING]: Platform linux on host default is using the discovered Python
    Install docker.amazon-ebs.ubuntu: interpreter at /usr/bin/python3.12, but future installation of another Python
    Install docker.amazon-ebs.ubuntu: interpreter could change the meaning of that path. See
    Install docker.amazon-ebs.ubuntu: https://docs.ansible.com/ansible-
    Install docker.amazon-ebs.ubuntu: core/2.18/reference_appendices/interpreter_discovery.html for more information.
    Install docker.amazon-ebs.ubuntu: ok: [default]
    Install docker.amazon-ebs.ubuntu:
    Install docker.amazon-ebs.ubuntu: TASK [Update apt package index] ************************************************
    Install docker.amazon-ebs.ubuntu: changed: [default]
    Install docker.amazon-ebs.ubuntu:
    Install docker.amazon-ebs.ubuntu: TASK [Install prerequisite packages] *******************************************
    Install docker.amazon-ebs.ubuntu: ok: [default]
    Install docker.amazon-ebs.ubuntu:
    Install docker.amazon-ebs.ubuntu: TASK [Create directory for Docker GPG key] *************************************
    Install docker.amazon-ebs.ubuntu: ok: [default]
    Install docker.amazon-ebs.ubuntu:
    Install docker.amazon-ebs.ubuntu: TASK [Download Docker's official GPG key] **************************************
    Install docker.amazon-ebs.ubuntu: changed: [default]
    Install docker.amazon-ebs.ubuntu:
    Install docker.amazon-ebs.ubuntu: TASK [Get system architecture] *************************************************
    Install docker.amazon-ebs.ubuntu: ok: [default]
    Install docker.amazon-ebs.ubuntu:
    Install docker.amazon-ebs.ubuntu: TASK [Get Ubuntu codename] *****************************************************
    Install docker.amazon-ebs.ubuntu: ok: [default]
    Install docker.amazon-ebs.ubuntu:
    Install docker.amazon-ebs.ubuntu: TASK [Add Docker repository to sources] ****************************************
    Install docker.amazon-ebs.ubuntu: changed: [default]
    Install docker.amazon-ebs.ubuntu:
    Install docker.amazon-ebs.ubuntu: TASK [Update apt package index (after adding Docker repo)] *********************
    Install docker.amazon-ebs.ubuntu: changed: [default]
    Install docker.amazon-ebs.ubuntu:
    Install docker.amazon-ebs.ubuntu: TASK [Install Docker packages] *************************************************
    Install docker.amazon-ebs.ubuntu: changed: [default]
    Install docker.amazon-ebs.ubuntu:
    Install docker.amazon-ebs.ubuntu: TASK [Create Docker daemon configuration for log settings] *********************
    Install docker.amazon-ebs.ubuntu: changed: [default]
    Install docker.amazon-ebs.ubuntu:
    Install docker.amazon-ebs.ubuntu: TASK [Restart Docker to apply log configuration] *******************************
    Install docker.amazon-ebs.ubuntu: changed: [default]
    Install docker.amazon-ebs.ubuntu:
    Install docker.amazon-ebs.ubuntu: TASK [Add ubuntu user to docker group] *****************************************
    Install docker.amazon-ebs.ubuntu: changed: [default]
    Install docker.amazon-ebs.ubuntu:
    Install docker.amazon-ebs.ubuntu: PLAY RECAP *********************************************************************
    Install docker.amazon-ebs.ubuntu: default                    : ok=13   changed=8    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
    Install docker.amazon-ebs.ubuntu:
==> Install docker.amazon-ebs.ubuntu: Stopping the source instance...
    Install docker.amazon-ebs.ubuntu: Stopping instance
==> Install docker.amazon-ebs.ubuntu: Waiting for the instance to stop...
==> Install docker.amazon-ebs.ubuntu: Creating AMI ubuntu-aws-v0.0.1 from instance i-08def819170356c2f
    Install docker.amazon-ebs.ubuntu: AMI: ami-0a431d9bd86317d65
==> Install docker.amazon-ebs.ubuntu: Waiting for AMI to become ready...
==> Install docker.amazon-ebs.ubuntu: Skipping Enable AMI deprecation...
==> Install docker.amazon-ebs.ubuntu: Skipping Enable AMI deregistration protection...
==> Install docker.amazon-ebs.ubuntu: Terminating the source AWS instance...
==> Install docker.amazon-ebs.ubuntu: Cleaning up any extra volumes...
==> Install docker.amazon-ebs.ubuntu: No volumes to clean up, skipping
==> Install docker.amazon-ebs.ubuntu: Deleting temporary security group...
==> Install docker.amazon-ebs.ubuntu: Deleting temporary keypair...
Build 'Install docker.amazon-ebs.ubuntu' finished after 9 minutes 32 seconds.

==> Wait completed after 9 minutes 32 seconds

==> Builds finished. The artifacts of successful builds are:
--> Install docker.amazon-ebs.ubuntu: AMIs were created:
us-east-2: ami-0a431d9bd86317d65
```

Once the build is complete, you should see a new AMI `ubuntu-aws-v0.0.1` with Docker pre-installed and configured in the us-east-2 region.

## Final Thoughts

HashiCorp Packer simplifies AWS AMI creation by automating the build process, ensuring consistency, and speeding up deployments. With support for multiple platforms and provisioning tools like Ansible, Packer is a powerful choice for infrastructure automation. Start using Packer today to build custom AMIs effortlessly!