## Notes from platform dojo

* SORT OUT AWS LIMITS EARLY !
* Need to have the IAM users sorted for both bbl and pcf-pipelines
* Need to have subnets sorted for:
  * bosh/concourse/pcf-pipelines
  * each platform
* remember to set an IP range to allow (ssh|https) access to opsman
* need to make sure you use legacy pivnet token otherwise `upload-ert` will fail
  to download
* `bbl up` leaves a mess in your directory - raise an issue to have it in a temp directory - see https://github.com/aussielunix/zero-to-concourseci/blob/master/.gitignore
* make sure to add some ephemeral disk to concourse workers
  * https://github.com/pivotal-cf/go-pivnet/issues/19
  * set `worker_ephemeral_disk`: to `100GB_ephemeral_disk`
  * add operations file `operations/worker-ephemeral-disk.yml`
* Concourse 3.8 https://github.com/concourse/concourse-bosh-deployment/tree/133f54c41fd9e3e0cf95b38a014b09e35cdfafd0
* BBL_ENV_NAME will be used for vpc name ($BBL_ENV_NAME-vpc)
* BBL_ENV_NAME is for concourse and not pcf
* BBL_ENV_NAME is shared concourse for all platform instances
* why is the BBLUP cidr so huge ?
  * it has to be a /20 or larger !


## credhub + uaa

* get the credhub-cli from [github](https://github.com/cloudfoundry-incubator/credhub-cli/releases) or `brew install cloudfoundry/tap/credhub-cli`
* get the uaa cli ??
* read https://github.com/cloudfoundry-incubator/credhub/blob/master/docs/operator-quick-start.md#credhub-credential-types

