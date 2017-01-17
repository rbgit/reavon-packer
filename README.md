# Packer Templates

The packer template `templates/CentOS-7-x86_64-Minimal.vbox.json` uses [Virtualbox](https://www.virtualbox.org) to build identical machine images for:

* [Vagrant Box](https://www.vagrantup.com/docs/boxes.html)
* [Amazon AMI](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AMIs.html)

Clone this repository

```
git clone git@github.com:reavon/reavon-packer.git
cd reavon-packer
```

## Directory structure
```
packer-templates/
                ├── bin/                <-- Misc scripts
                ├── builds/             <-- Repository of build's ( .ami's,.box's and .ova's )
                ├── docs/               <-- Stuff you need to know
                ├── files/              <-- File to be uploaded to vm (keep destination structure)
                ├── .git/               <-- Hopefully you know what this is
                ├── http/               <-- Keep kickstart files here
                ├── logs/               <-- Logs from packer-build script
                ├── manifests/          <-- Puppet Provisioner site.pp and modules download script
                ├── packer_cache/       <-- Cached' items used by packer... including downloaded iso's
                ├── scripts/            <-- Scripts to be executed by Shell Provisoner
                ├── templates/          <-- Packer json tempaltes
                ├── .editorconfig       <-- so we are all consistant
                ├── .gitignore          <-- We don't want to share everything with the world
                ├── packer-build*       <-- Script that kicks off packer run
                └── README.md           <-- You are reading it
```

## Variables

We use variables when working with packer so that we do not leak credentials via `git`

Create a file with all of your needed packer variables and source it.

```
cat << PACKER > /home/${USER}/.packer_variables
# Personal AWS Creds
AWS_ACCESS_KEY=
AWS_SECRET_KEY=

AWS_KEY_PAIR=nocbot@nlm

export AWS_ACCESS_KEY
export AWS_SECRET_KEY
export AWS_KEY_PAIR

# Packer AMI Builder creds
PACKER_AWS_ACCESS_KEY=
PACKER_AWS_SECRET_KEY=

# Packer root password
PACKER_PASSWORD

export PACKER_AWS_ACCESS_KEY
export PACKER_AWS_SECRET_KEY
export PACKER_PASSWORD
PACKER
```

Fill in the variables and source it
```
. /home/${USER}/.packer_variables
```

Export the template we will be using
```
export packer_template="CentOS-7-x86_64-Minimal.json"
```
Make sure packer is installed and runs
```
packer --version
```

Validate the template
```
packer validate "$packer_template"
```

Get an overview of what the template includes (variables, builders, provisioners)
```
packer inspect "$packer_template"
```
Produce machine images for all defined builders
```
packer build "$packer_template"
```

Dynamic resize of a volume in CentOS 7 (WIP)
--------------------------------------------

```
runcmd:
  - /usr/bin/growpart /dev/xvda 2
  - pvresize /dev/xvda2
  - lvresize -r -L +100%FREE /dev/vg00/lv_nlm
```

**HINT** To create passwords for kickstart files, etc:
```
python -c 'import crypt; print(crypt.crypt("MyS3CR3TP455", crypt.mksalt(crypt.METHOD_SHA512)))'
```
