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

## Install bosh-bootloader

Grab the latest [binary](https://github.com/cloudfoundry/bosh-bootloader/releases) for your OS, place it in a directory in your $PATH and set the exectue bit on it. (chmod +x ...)

Or if you are on MacOS you can install via brew cask.

```bash
$ brew tap cloudfoundry/tap
$ brew install bosh-cli
$ brew install bbl
```

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

Get some downloads from Pivnet and place them in the archives directory.

* grab concourse release from pivnet and check the sha256
* grab garden-runc from pivnet and check the sha256
* grab the latest AWS Stemcell from pivnet and check the sha256
* upload to director
```bash
  bosh upload-release ../artifacts/concourse-3.8.0.tgz
  bosh upload-release ../artifacts/garden-runc-1.9.0.tgz
  bosh upload-stemcell ../artifacts/light-bosh-stemcell-3468.27-aws-xen-hvm-ubuntu-trusty-go_agent.tgz
```
* Get a copy of the Concourse deployment repo
  `git clone https://github.com/concourse/concourse-bosh-deployment.git`
  `cd concourse-bosh-deployment/cluster`
* Checkout v3.9.2
  `git checkout v3.9.2`
* Patch the deployment repo for Pivotal Concourse
  `git apply ../../pivotal_concourse.diff`
* Tune `settings.yml`
* Deploy concourse

```bash
bosh deploy -d concourse concourse.yml \
  -l ../versions.yml \
  --vars-store=../../cluster-creds.yml \
  --vars-file=../../settings.yml \
  -o operations/privileged-http.yml \
  -o operations/no-auth.yml \
  -o operations/web-network-extension.yml
```



## NOTES

### to get the external url

:FIXME: This is Linux only

```bash
export external_url="$(bbl lbs |awk -F':' '{print $2}' |sed 's/ //' | awk '{ print $2}' | awk -F '[\[\]]' '{print $2}')"
```
