# Restricted Network OLM Mirror Helper

## Introduction

The ansible tasks in this repo are intended to help with the task of mirroring images to a disconnected or restricted network OpenShift install.

## Prequisites
* Install python3-openshift, jq, bsdtar, ansible, and moby-engine or docker-ce
  * Fedora: `dnf -y install python3-openshift jq bsdtar ansible moby-engine`
  * Fedora: `sudo grubby --update-kernel=ALL --args="systemd.unified_cgroup_hierarchy=0"` and reboot. This is required by docker at present.
  * `sudo systemctl enable docker && sudo systemctl start docker`
  * CentOS 7: Jinja2 on CentOS 7 is too old to use this playbook. You may be able to get it working by installing a newer version with pip, but this is generally inadvisable on a production system as it can cause problems when updating packages with yum/dnf.

* Configure docker and ensure it is accessible without root
  `sudo groupadd docker`
  `sudo usermod -aG docker $USER`
  `sudo systemctl restart docker`
  `newgrp docker #or logout and login`

* Login to registry.redhat.io
  * These are the same credentials you log into https://access.redhat.com
  * `docker login -u $username` and enter your password when prompted.

* Ensure you are logged into your cluster
  * With Openshift 4 a kubeconfig file is created. You can export this in your `.bashrc` or elsewhere
  * For example: `export KUBECONFIG=/home/$USER/.agnosticd/ocp4/ocp4-workshop_ocp4_kubeconfig`

* Get Private Quay namespace logins
  * Fill out https://docs.google.com/spreadsheets/d/1OyUtbu9aiAi3rfkappz5gcq5FjUbMQtJG4jZCNqVT20/edit#gid=0
    * If you need to generate a new GPG key run `gpg --full-gen-key`
    * Upload the public key at https://keys.fedoraproject.org/ with the `Submit Key` button link.
    * To get the key id/fingerprint easily you can lookup your key after uploading it at https://keys.fedoraproject.org/

* Use the private quay namespace logins to obtain a basic auth token for API access
  * Once you received the quay robot username and token you need to use these to generate a basic auth token.
  * Ensure you export the username and password for the environment you intend to use, stage with stage, etc.
  * Run the following
  ```
  export AUTH_USER
  export AUTH_TOKEN
  echo $(curl -sH "Content-Type: application/json" -XPOST https://quay.io/cnr/api/v1/users/login -d '
  {
    "user": {
        "username": "'"${AUTH_USER}"'",
        "password": "'"${AUTH_TOKEN}"'"
    }
  }')
  ```
  * export the resulting token in your .bashrc or elsewhere:
  * For example: export QUAY_TOKEN="basic ...."

* Login to registry.stage.redhat.io
  *  `docker login -u redhat -p redhat`

## Instructions
* You can use the `ansible-playbook list-operators.yml playbook to get a list of operators available.

* You can use `ansible-playbook get-operator-csv.yml` playbook to retrieve the CSV for the operators defined in `config.yml`. This is useful for determining what images are required by the operator and whether or not the operator uses SHAs so you can provide an `images` or `resolved_images` list. Right now providing this list is the most tedious step since there is no standard way to define images in the CSV. This process should improve in 4.3.

When ready to mirror the operator:
* Ensure you are logged into your OpenShift cluster, registry.redhat.io, and you local/internal registry if you are not pushing to your OpenShift cluster.

* Update `local_registry` with the local/internal registry you wish to use for mirroring images
* Update `local_registry_ns` with the organization/project/namespace you wish to push the images to on your registry host
Note: If you leave the values for these two parameters the internal OpenShift registry will be exposed and images will be mirrored there.

* Create or copy an example config.yml into place if you haven't already.
* Update the `operators` list with the operators you wish to mirror
* Update `use_shas` to match whichever the operator uses. Going forward SHAs will be the standard.
* Update `deployment_namespace` with the namespace you want to deploy the operator to
* Update `images` with the image names and a command for finding the image tags/SHAs. See `config.yml.cam-operator-1.0.1-example` as an example
* Alternatively provide a list of resolved_images. See `config.yml.cam-operator-1.0.0-example` as an example

## TODO
* There are quite a few shell commands that can probably be updated to use other ansible modules in order to work more smoothly
