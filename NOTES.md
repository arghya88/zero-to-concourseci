## Notes from platform dojo

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

