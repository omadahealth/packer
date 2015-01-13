# Omada Packer.io Amazon AMI Creation

## Downloading and Installing Packer

You can download the binary from:

` Linux: https://dl.bintray.com/mitchellh/packer/packer_0.7.5_linux_amd64.zip`

OSX Users that use the brew package manager can install via:

`brew tap homebrew/binary`

`brew install packer`

Unpack the binary.

` Linux: cd /usr/local; unzip ~/Downloads/packer_0.7.5*`

## Anatomy of a Packer Template

Packer templates are broken into multiple stanzas (json objects). Possible stanzas are:
    - Builders
    - Provisioners
    - Post-Processors

Each stanza is a JSON array of objects.
e.g. Multiple provisioners can be defined.
```
"provisioners": [
    {
        "type": "shell",
        "script": "packs/install-cli-tools"
    },
    {
        "type": "shell",
        "script": "packs/encrypt-provision"
    },
    {
        "type": "ansible",
        "inventory_file": "inventory.packer"
        "playbook_file": "site.yml"
    }
]
```

For more information on the template, see: [Templates Introduction](http://packer.io/docs/templates/introduction.html)

`min_packer_version` defines the minimum packer version required to perform the build. If you are following these instructions, use `0.7.5`.

### Builders

Packer is able to build images into a plethora of formats. The scope of this document only allows for a discussion on the `amazon-ebs` builder. For more information on builders see: [Builders Introduction](http://packer.io/docs/templates/builders.html)

The `amazon-ebs` builder requires the following keys to have values:
* `region` :: the region in which to launch the build instance.
* `vpc_id` :: the VPC id in which to launch the build instance.
* `subnet_id` :: the subnet id on which to connect the build instance.
* `availability_zone` :: the availability zone in which to launch the build instance.
* `source_ami`:: the AMI to launch as the build instance.
* `instance_type`:: the type of the build instance
* `ssh_username` :: the username of the provisioning user.
* `ami_name` :: the name of the *new* AMI
* `ami_description` :: the description of the *new* AMI

The ami_block_device_mappings key is optional. It uses the same keys as the EC2 CLI tools.
e.g. For an additional 25G encrypted volume as sdg
```
[
    {
        "DeviceName": "sdg",
        "size": "25",
        "encrypted": true
    }
]
```

[Documenation on additional tunables can be found here.](http://packer.io/docs/builders/amazon.html)

### Provisioners

Packer is able to provision the build instance in a variety of ways. The scope of this document only allows for a discussion of the `shell` and `ansible` provisioners. For more information see: [Provisioners Introduction](http://packer.io/docs/templates/provisioners.html)

Below is an example of the provisioners object:
```
"provisioners": [
    {
        "type": "shell",
        "script": "packs/install-cli-tools"
    },
    {
        "type": "shell",
        "scripts": [ "packs/encrypt-provision", "packs/detach-old-root", "packs/attach-new-root" ]
    },
    {
        "type": "ansible",
        "inventory_file": "inventory.packer"
        "playbook_file": "site.yml"
    }
]
```

The `shell` provisioner can use `inline`, `script`, or `scripts`. `inline` is a string one would type into the command prompt. `script` is a relative path to a shell script. `scripts` is an array ([]) of relative paths to shell scripts.

The `ansible` provisioner allows one to provision the build instance using *local* ansible inventory and playbook files.

### Post-Processors

Post-processors are not within the scope of this document. [Please refer here](http://packer.io/docs/templates/post-processors.html).

## Building with Packer

It is necessary to set the following environment variables:
* `PACKER_SSH_KEYFILE` :: path to the ssh private key to use for the build instance
* `AWS_ACCESS_KEY` :: EC2 API id
* `AWS_SECRET_KEY`:: EC2 API key
* `AWS_GENERATION_ENCRYPTION_KEY` :: KMS encrypting key arn
* `JENKINS_HOST` :: the fqdn of the jenkins host
* `JEKNINS_TOKEN` :: the jenkins api token used to trigger the swap of root devices

To build:
```
packer validate packer-r0a.json
packer build packer-r0a.json
```
