# Ansible Navigator Usage

This exercise show the usage of the `ansible-navigator` utility. You will run Playbooks with the Navigator, create an Execution Environment image for the Navigator to use, inspect the image with the navigator and some more exercises.

## Install Navigator

Installing the `ansible-navigator` is done via the Python package manager. If you want to, (or have to) use a *Python virtual environment*, create it with this command:

```console
python3 -m venv community-call-ve
```

Activate the VE:

```console
source community-call-ve/bin/activate
```

Now, install the Navigator:

```console
pip3 install ansible-navigator
```

Ok, that was easy.  

A configuration file for the Navigator is already present, take a look at `ansible-navigator.yml`.  
This file defines some useful stuff, for example, a `logs` folder will be created, the mode is set to `stdout` (instead of interactive for the TUI) and an execution environment to use.

> The execution environment is not pulled from an external registry, but must be present and built locally (in our exercises)!

## Build EE image

The execution environment bundles all the dependencies for the automation project, this way we have a consistent and reproducible automation environment which can be used locally (with the Ansible Navigator), as well as in AAP (or AWX).

The `aci-automation.yml` playbook uses additional collections, you need to add them to your `execution-environment.yml` definition file, take a look at the [*ansible-builder* documentation](https://ansible.readthedocs.io/projects/builder/en/latest/definition/#dependencies) on how to add the `galaxy` dependencies (Ansible Collections are installed from galaxy.ansible.com or the Automation Hub via the ansible-galaxy utility).  

You need to install the following collections:

* **cisco.aci** - Most of the tasks use this collection
* **community.general** - plugin usage in the last task

<p>
<details>
<summary><b>Help needed?</b></summary>

Add the `galaxy` key and add the collections list under the `collections` key.

```yaml
---
version: 3

images:
  base_image:
    name: docker.io/redhat/ubi9:latest

additional_build_steps:
  prepend_final:
    - RUN update-alternatives --install /usr/bin/python3 python /usr/bin/python3.12 20

dependencies:
  ansible_core:
    package_pip: ansible-core
  ansible_runner:
    package_pip: ansible-runner
  python_interpreter:
    package_system: "python312"
    python_path: "/usr/bin/python3.12"
    
  galaxy:
    collections:
      - cisco.aci
      - community.general
```

</details>
</p>

After you added the missing collections, build the execution environment image with the following navigator command:

```console
ansible-builder build -t community-call-ee -v 3
```

> The `ansible-builder` utility is installed as a dependency with `ansible-navigator`.

Let's take a look at what we just built, we can inspect the image with the Navigator. Start an *interactive* session:

```console
ansible-navigator -m interactive
```

In the TUI, enter `:images` and press *enter*.  
After the Navigator collected all images (it will check if the locally present images are EEs or not), press the number of the desired EE image (the currently used one is highlighted in purple, all other EEs in green), for numbers greater than 9, use `:<number>` and press *enter*.  
Press `2` to inspect the installed Ansible version and all collections. You'll see some additional collections, these are Collections requirements of your Collections.  
Press `ESC` to move back one level and press `3` to inspect all installed Python packages.  
Press `ESC` multiple times until you reached the *Welcome* screen again.  
Input `:doc cisco.aci.aci_tenant` to take a look at the ACI Tenant module documentation.  

All right, thats enough for now, press `ESC` until you are back on your terminal.

## Run playbook

Running a playbook with `ansible-navigator` is as easy as running it with `ansible-playbook`, the correct subcommand/parameter is `run`. A playbook is already present, run the `aci-automation.yml` playbook with the following command:

```console
ansible-navigator run playbooks/aci-automation.yml
```

> **Error?**  
> The playbook requires some *environment variables* which are uses as connection parameters for the ACI modules.

Why environment variables? These variables are dynamic variables used by a shell and its child processes, they are *ephemeral*, after the next reboot or when opening a new session, they will be gone. As this may seem not very practical at first, this is a good way to temporarily share secrets as you don't need a file with the secret (which may get committed unencrypted) and it is only visible for your session. Additionally, Ansible can make use of environment variables (for config options) and loads of Ansible modules can make use of them as well and you don't have to add the module parameter to your tasks. And even more cool, AAP itself makes use of environment variables as well to share secrets while running the automation. When using them locally, we know that this will also work when running in AAP later.

A script is prepared to export the required variables for ACI connection, use the following command:

```console
. export-aci-credentials.sh
```

Ensure the environment variables are exported correctly:

```console
env | grep ACI
```

Ok, run the playbook again:

```console
ansible-navigator run playbooks/aci-automation.yml
```

> **The same error still?**  
> Ansible is running *inside* the execution environment container, your environment variables are not known inside the container (as well as Ansible is run by a completely different user).

You need to **pass** the environment variables into the EE container, you need to adjust the `ansible-navigator.yml` configuration, take a look at the [ansible-navigator documentation](https://ansible.readthedocs.io/projects/navigator/settings/#pass-environment-variable).

Pass in the list of the four `ACI_*` environment variables:

* ACI_HOST
* ACI_USERNAME
* ACI_PASSWORD
* ACI_VALIDATE_CERTS

<p>
<details>
<summary><b>Help needed?</b></summary>

Add the `environment-variables` key and `pass` the list of the four `ACI_*` environment variables.

```yaml
---
ansible-navigator:
  execution-environment:
    enabled: true
    image: localhost/community-call-ee:latest
    pull:
      policy: missing
    environment-variables:
      pass:
        - ACI_HOST
        - ACI_USERNAME
        - ACI_PASSWORD
        - ACI_VALIDATE_CERTS
  logging:
    level: warning
    file: logs/ansible-navigator.log
  mode: stdout
  playbook-artifact:
    enable: true
    save-as: "logs/{playbook_status}-{playbook_name}-{time_stamp}.json"
```

</details>
</p>

Ok, run the playbook again:

```console
ansible-navigator run playbooks/aci-automation.yml
```

> Well, another error...
> A missing package, we need to install the `jmespath` python package.

You know the drill, adjust your `execution-environment.yml` and add the missing Python dependency.

<p>
<details>
<summary><b>Help needed?</b></summary>

Add the `python` key and add the missing package as a list item.

```yaml
---
version: 3

images:
  base_image:
    name: docker.io/redhat/ubi9:latest

additional_build_steps:
  prepend_final:
    - RUN update-alternatives --install /usr/bin/python3 python /usr/bin/python3.12 20

dependencies:
  ansible_core:
    package_pip: ansible-core
  ansible_runner:
    package_pip: ansible-runner
  python_interpreter:
    package_system: "python312"
    python_path: "/usr/bin/python3.12"
    
  galaxy:
    collections:
      - cisco.aci
      - community.general
  python:
    - jmespath
```

</details>
</p>

After you added the missing python dependency, build the execution environment image with the following navigator command again:

```console
ansible-builder build -t community-call-ee -v 3
```

Ok, run the playbook **again**:

```console
ansible-navigator run playbooks/aci-automation.yml
```

> Finally, a successful run!

Now, let's run the playbook again in **interactive** mode using the **TUI** (Text-based User interface). The configuration file sets the mode to `stdout`, we can override this on the CLI:

```console
ansible-navigator run playbooks/aci-automation.yml -m interactive
```

You can drill down the playbook run by using the number on each line, for the play `0`, for the third task (*Get all contracts of common tenant*) `2` and so on. For numbers greater than 9, use `:11` and press *enter*.

## Running without EE

Let's clear the terminal:

```console
clear
```

If you don't want to use an execution environment but instead want to instruct the Navigator to use the **local** Ansible installation, you can disable the EE usage in the `ansible-navigator` by setting `ansible-navigator.execution-environment.enabled` to `false`.

After disabling the EE usage, try to run the `aci-automation.yml` playbook again. It will fail with missing collections and Python packages.  

Let's use another playbook, run the `fact-gathering.yml` playbook, it gathers some infos about `localhost`.

```console
ansible-navigator run playbooks/fact-gathering.yml
```

You'll see your Python version Ansible is using, the distribution version and some additional stuff.  

Enable the EE usage again and run the playbook again. You'll see (most likely) a different distribution and Python version as this is now **inside** the container.  
*Localhost* is a bit different as what you expected? Try to remember this when using EEs, for example, downloading stuff from a remote machine and store it *locally* does not work anymore. When the playbook run finishes, the Container and its storage will be deleted, your downloaded stuff is gone!

## Replay playbook runs

Oh, we cleared the terminal in the last step, now we can't see the playbook runs anymore which we did in the beginning. What was the error again with what the playbook failed earlier?  

Let's make use the of the gathered logs and *playbook artifacts* which are stored by the Navigator automatically (we only adjusted the path where they are stored).  

Use the `ansible-navigator replay` command and append the desired playbook artifact from the `logs` folder. **Use tab completion** to find the desired artifact, failed Job runs are prefixed with `failed-`, followed by the playbook name (`failed-aci-automation-...`) and a timestamp.
