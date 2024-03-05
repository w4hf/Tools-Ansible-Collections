EE Factory
=========

A versions-aware idempotent role for building and publishing Ansible Execution Environments.

This role is designed to be used as the source of truth configuration as code declaration of EE.

A gitlab pipeline is provided with the role. To use it, copy the `.gitlab-ci.yml` file from the `tests` folder to the root directory where the calling playbook resides.


Requirements
------------

This role requires the collection `containers.podman` and the `ansible-builder` utility.

Install the collection with the following comands :

```shell
mkdir -p collections/
ansible-galaxy collection install -r requirements.yml -p collections/
```

Getting Started
------------

The following steps will help you get started by building and publishing an image named `demo-ee`

1. Create a working directory
```shell
mkdir -p testing/collections/
mkdir -p testing/roles/w4hf.ee_factory
cd testing
git clone https://gitlab.com/w4hf-lab/ee_factory.git roles/w4hf.ee_factory
```

2. Download the required collections
```shell
ansible-galaxy collection install -r roles/w4hf.ee_factory/requirements.yml -p collections/
```

3. Copy everything inside the tests folder to the root testing folder
```shell
cp roles/w4hf.ee_factory/tests/* .
```

4. Edit the ENV.sh file and insert your environment details
```shell
vim ENV.sh
source ./ENV.sh
```

5. Create one or more image definition file under the folder `images.d`. Below an example of an image definition file:

```yaml
---
EEs:
  - name: demo-ee
    version: 1
    build_method: ansible-builder
    deploy_to:
      - dev
      - preprod
      - prod
    collections:
      - name: ansible.utils
        version: 2.9.0
      - name: ansible.windows
        version: 1.14.0
    python_packages:
      - psutil
      - jmespath
    system_packages:
      - iputils [platform:rpm]
...
```

6. Run the build playbook
```shell
ansible-playbook build.yml
```

Role Variables
--------------

### Global Settings - Automation Hub (Container Registry) Variables

Automation Hub variables could be set in the `images.yml` file or declared as environment variables. The environment variables have precedence and will overwrite the variables in the file.

| Variable          | Corresponding Environment Variable | Type    | Description                                                                     |
|-------------------|------------------------------------|---------|---------------------------------------------------------------------------------|
| ah_hostname       | `AH_HOSTNAME`                      | string  | FQDN of your Automation Hub which stores the base image and the image to build  |
| ah_user           | `AH_USER`                          | string  | Username used to pull/push images from/to the Automation Hub                    |
| ah_password       | `AH_PASSWORD`                      | string  | Password used to pull/push images from/to the Automation Hub                    |
| ah_validate_certs | `AH_VALIDATE_CERTS`                | boolean | Validate or not the Automation Hub TLS certificates                             |

### Global Settings - Build Variables

| Variable                        | Default Value                           | Description                                                              |
|---------------------------------|-----------------------------------------|--------------------------------------------------------------------------|
| target_environment              | content of `CI_ENVIRONMENT_NAME`        | Actual target environment to which the EE will be deployed on this run   |
| deploy_to                       | `[ 'dev' ]`                             | Deploy all the images to this list of environment                        |
| http_proxy                      | N/A                                     |                                                                          |
| https_proxy                     | N/A                                     |                                                                          |
| no_proxy                        | N/A                                     |                                                                          |
| base_image_name                 | `ee-minimal-rhel8`                      | Name of the image used as base for building the new EE                   |
| base_image_tag                  | `latest`                                | Tag of the image used as base for building the new EE                    |
| cleanup_after_build             | `true`                                  | Remove context/ folder after a successful build                          |
| current_version_file_name       | `current_version`                       | Name of file where to store each image current active version            |
| execution_environment_template  | `execution-environment.yml.j2`          | Template location of execution-environment.yml                           |
| ansible_cfg_template            | `ansible.cfg.j2              `          | Template location of execution-environment.yml                           |
| package_manager_path            | `/usr/bin/microdnf`                     | Package manager used in image construction                               |

### Global Settings - Git Variables

| Variable                | Default Value                                              | Description                                                                                                |
|-------------------------|------------------------------------------------------------|------------------------------------------------------------------------------------------------------------|
| push_to_git_remote_repo | `false`                                                    | Set to true to automatically push files changes to git repo after build. Useful in the automated pipeline. |
| git_email               | `ee_factory@ansible.com`                                   | Email of the automated push git account                                                                    |
| git_username            | `ci-bot`                                                   | Username of the automated push git account                                                                 |
| git_repo                | `gitlab.com/w4hf/ee_factory`                               | Remote git repo without https://                                                                           |
| git_access_token        | looked up from the environment variable `GIT_ACCESS_TOKEN` | Token used for automated git push                                                                          |


## Image Specific Settings

Image specific settings are settings specific to the image they are declared along with.

Image declaration is done under the variable `EEs`.

`EEs` is a list of element. Each element represent an image to be built. The following table decribe parameters that could be declared for each element (image) :

| Variable             | Default Value   | Corresponding Environment Variable | Description                                                                                                                                            |
|----------------------|-----------------|------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------|
| name *               | N/A             | N/A                                | The name of the image to build                                                                                                                         |
| version *            | N/A             | N/A                                | The version of the image to build                                                                                                                      |
| build_method         | ansible-builder | N/A                                | How the image will be built. The current supported methods are `ansible-builder` or `Containerfile`                                                    |
| deploy_to            | ["default"]     | N/A                                | The environment to which the image will be pushed afer build. If defined, this will override the global parameter `deploy_to`                          |
| collections          | N/A             | N/A                                | List of Ansible collections to be installed in the new image. Each element of the list should be a dictionary containing 2 keys : `name` and `version` |
| python_packages      | N/A             | N/A                                | List of python packages to be installed in the new image.                                                                                              |
| system_packages      | N/A             | N/A                                | List of system programs to be installed in the new image.                                                                                              |
| ah_hostname          | N/A             | AH_<EE-NAME>_HOSTNAME              | The Automation Hub FQDN to be used specifically for this image. If not defined, the one defined in global settings will be used.                       |
| ah_user              | N/A             | AH_<EE-NAME>_USER                  | The Automation Hub username to be used specifically for this image. If not defined, the one defined in global settings will be used.                   |
| ah_password          | N/A             | AH_<EE-NAME>_PASSWORD              | The Automation Hub password to be used specifically for this image. If not defined, the one defined in global settings will be used.                   |
| ah_validate_certs    | N/A             | AH_<EE-NAME>_VALIDATE_CERTS        |                                                                                                                                                        |
| package_manager_path | N/A             | N/A                                | The package manager to be used specifically for this image. If not defined, the one defined in global settings will be used.                           |

Gitlab CI/CD setup
------------

1. In the Gitlab interface, create a Gitlab Project Scope Access Token

2. In the Gitlab interface, create the following Gitlab CICD Environment Variables

| Variable                 | Example Value      | Description                                                   |
|--------------------------|------------------------------------|-----------------------------------------------|
| GIT_ACCESS_TOKEN         | ********           | The gitlab project access token created in the previous step  |
| AH_HOSTNAME              | `hub.company.com`  | Automation Hub FQDN                                           |
| AH_PASSWORD              | ********           | Automation Hub Password                                       |
| AH_USER                  | `admin`            | Automation Hub Username                                       |
| ANSIBLE_COLLECTIONS_PATH | `collections/`     | Path where the required collections are installed             |

3. To run the pipeline, make sure you have available runners then change something in the `images.yml` file than commit and push to the `main` branch. A pipeline should launch to build and deploy your images.

Dependencies
------------

This role uses the collection `containers.podman`

Install it with the following comands :

```shell
mkdir collections/
ansible-galaxy collection install -r requirements.yml -p collections/
```

Example Playbook
----------------


```yaml
- name: Build and publish EEs
  hosts: localhost
  gather_facts: false

  vars_files:
    - images.yml

  roles:
    - w4hf.ee_factory
```

License
-------

BSD

Author Information
------------------

Hamza Bouabdallah