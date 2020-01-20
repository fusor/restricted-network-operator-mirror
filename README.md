# Restricted Network OLM Mirror Helper

## Introduction

The ansible tasks in this repo are intended to help with the task of mirroring images to a disconnected or restricted network OpenShift install.

## Prequisites
This playbook has only been tested on Fedora. It may or may not work on other Linux distributions. CentOS 7 is known not to work because the python-jinja2 package is too old. It may be possible to circumvent this by installing a newer version using pip, but this is generally inadvisable because it can cause problems wtih dnf/yum updates.

* Install python3-openshift, jq, bsdtar, ansible, and moby-engine or docker-ce. For Fedora:
  * `sudo dnf -y install python3-openshift jq bsdtar ansible moby-engine`
  * `sudo grubby --update-kernel=ALL --args="systemd.unified_cgroup_hierarchy=0"` and reboot.
  * `sudo systemctl enable docker && sudo systemctl start docker`

* Configure docker and ensure it is accessible without root
  ```
  sudo groupadd docker
  sudo usermod -aG docker $USER
  sudo systemctl restart docker
  newgrp docker #or logout and login
  ```

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
### Mirror Operator
* Create or copy an example config.yml into place if you haven't already.
* `ansible-playbook mirror.yml`
* The playbook will pause and prompt you to add the registry to your insecure registries list.
  * For docker 1.13 this is done in /etc/sysconfig/docker
  * For moby-engine/docker-ce this is done in /etc/docker/daemon.js
  * if you've linked docker to podman this is done in /etc/containers/registries.conf
  * One of the TODO's involves attempting to deal with all these versions and file formats... PR welcome.

### Config Options
* Update `local_registry` with the local/internal registry you wish to use for mirroring images
* Update `local_registry_ns` with the organization/project/namespace you wish to push the images to on your registry host. If you leave the defaults either the internal registry will be used (OCP3 default) or an insecure registry will be configured (OCP4). The insecure registry is intended to be a temporary workaround.
* Update the `operators` list with the operators you wish to mirror
* Update `use_shas` to match whichever the operator uses. Going forward SHAs will be the standard.
* Update `deployment_namespace` with the namespace you want to deploy the operator to
* Update `images` with the image names and a command for finding the image tags/SHAs. See `config.yml.cam-operator-1.0.1-example` as an example
* Alternatively provide a list of resolved_images. See `config.yml.cam-operator-1.0.0-example` as an example

### Retrieve list of Operators
* You can use the `ansible-playbook list-operators.yml playbook to get a list of operators available.

* You can use `ansible-playbook get-operator-csv.yml` playbook to retrieve the CSV for the operators defined in `config.yml`. This is useful for determining what images are required by the operator and whether or not the operator uses SHAs so you can provide an `images` or `resolved_images` list. Right now providing this list is the most tedious step since there is no standard way to define images in the CSV. This process should improve in 4.3.

## TODO
* There are quite a few shell commands that can probably be updated to use other ansible modules in order to work more smoothly
* relatedImages is a new feature of CSV's. We can use this to get the list of images to mirror now instead of putting this work on the user
* Deal with internal registry and global pull secret. See if we can get this to work for mirrors
* Add some block/rescue with debug messages at typical failure points to hint at what the problem is.
* Get docker version an attempt to configure insecure registry automatically
