# Concourse for PCF Pipelines

**THIS IS A WIP**

## Introduction

At the end of this tutorial you should have:

* a new AWS VPC
* a bosh director
* Concourse CI cluster

## Install Dependencies

The following should be installed on your local machine:
* [bosh-cli](https://bosh.io/docs/cli-v2.html)
* [bosh create-env dependencies](https://bosh.io/docs/cli-env-deps.html)
* [terraform](https://www.terraform.io/downloads.html) >= 0.11.0
* ruby (necessary for bosh create-env)
* [direnv](https://direnv.net/)

## Install bosh-bootloader

Grab the latest [binary](https://github.com/cloudfoundry/bosh-bootloader/releases) for your OS, place it in a directory in your $PATH and set the exectue bit on it. (chmod +x ...)

Or if you are on MacOS you can install via brew cask.

```bash
$ brew tap cloudfoundry/tap
$ brew install bosh-cli
$ brew install bbl
```

## Prepare your terminal environment

In the `nonprod` directory:

_ignore direnv at this stage when prompted_

`cp .envrc-example .envrc`

Tune this file. It should be self explanitory.

`direnv allow`

## Prepare an AWS VPC

This will build a new VPC, subnets etc and create an ELB configured as an ELB for ConcourseCI.

*Note*: This will assume a whole bunch of defaults like VPC CIDR etc. You should
read about [customizing IaaS paving with terraform](https://github.com/cloudfoundry/bosh-bootloader/blob/master/docs/advanced-configuration.md#customizing-iaas-paving-with-terraform) before proceeding.  
There is an example of setting a custom VPC CIDR included in `{nonprod|prod}/vars/terraform.tfvars`. Be sure to delete this file if you do not want to set your own.

`bbl up --lb-type concourse`

You should now have a working bosh director in a new VPC.  
Update your terminal's environment variables that are needed so your environment includes info on the new resources just built.

`direnv allow`

## Pivotal Concourse

We are now ready to deploy a Concourse CI to our director.  

Get some downloads from Pivnet and place them in the artifacts directory.

* grab concourse release from pivnet and check the sha256
* grab garden-runc from pivnet and check the sha256
* grab the latest AWS Stemcell from pivnet and check the sha256
* upload to director
```bash
  bosh upload-release ../artifacts/concourse-3.8.0.tgz
  bosh upload-release ../artifacts/garden-runc-1.9.0.tgz
  bosh upload-stemcell ../artifacts/light-bosh-stemcell-3468.27-aws-xen-hvm-ubuntu-trusty-go_agent.tgz
```
* set the external url
  * linux
  ```bash
  export external_url="$(bbl lbs |awk -F':' '{print $2}' |sed 's/ //' | awk '{ print $2}' | awk -F '[\[\]]' '{print $2}')"
  ```
  * MacOS
  ```bash
  export external_url="https://$(bbl lbs | awk -F':' '{print $2}' | awk -F'[' '{print $2}' | awk -F']' '{print $1}')"
  ```
* Tune `manifests/settings.yml`
* Deploy concourse

```bash
bosh deploy -d concourse manifests/concourse.yml \
  -l manifests/versions.yml \
  --vars-store=cluster-creds.yml \
  --vars-file=manifests/settings.yml \
  -o manifests/operations/privileged-http.yml \
  -o manifests/operations/no-auth.yml \
  -o manifests/operations/web-network-extension.yml \
  -o manifests/operations/worker-ephemeral-disk.yml
```


## NOTES

