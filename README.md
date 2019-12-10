# Restricted Network OLM Mirror Helper

## Introduction

The ansible tasks in this repo are intended to help with the task of mirroring images to a disconnected or restricted network OpenShift install.

## Instructions
* You can use the `list-operators.yml' playbook to get a list of operators available.

* You can use `get-operator-csv.yml` playbook to get the CSV for a given operator. This requires either creating or copying an example config.yml into place and filling in the `operators` list variable. This is useful for determining what images are required, whether or not the operator uses SHAs, and how to find the tags or SHAs in order to fill out the images list. Right now figuring this out is the most tedious step as there is no standard way to define images in the CSV, although this should improve in 4.3.

When ready to mirror the operator:
* Ensure you are logged into your OpenShift cluster, registry.redhat.io, and you local/internal registry if you are not pushing to your OpenShift cluster.

* Update `local_registry` with the local/internal registry you wish to use for mirroring images
* Update `local_registry_ns` with the organization/project/namespace you wish to push the images to on your registry host
Note: If you leave the values for these two parameters the internal OpenShift registry will be exposed and images will be mirrored there.

* Create or copy an example config.yml into place if you haven't already.
* Update the `operators` list with the operators you wish to mirror
* Update `use_shas` to match whichever the operator uses. Going forward SHAs will be the standard
* Update `deployment_namespace` with the namespace you want to deploy the operator to
* Update `images` with the image names and a command for finding the image tags/SHAs.

## TODO
* There are quite a few shell commands that can probably be updated to use other ansible modules in order to work more smoothly
* Implement/test the alternative approach of just providing a resolve_images list with sha's already filled in. It may be less tedious than producing commands to get tags or SHAs
* Make it easier to get commuity and certified operators as well
