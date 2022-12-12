# Best practices

Ansible is simple, flexible, and powerful. Like any powerful tool, there are many ways to use it, some better than others.  
This exercise helps you on identifying good and bad practices on a given playbook.

## Prerequisites

If you want to run the playbook, you need:

* Docker Container runtime
* Ansible (obviously)
* Python Package Manager *pip3*

The following Python packages:

* docker
* requests

The following Ansible collections:

* community.docker

Installing Docker is up to you, the dependencies can be installed with the provided *requirements* files:

```bash
pip3 install -r requirements.txt
```
```bash
ansible-galaxy collection install -r requirements.yml
```

## Task 1

Inspect the playbook `best-practice-container.yml`. 

**Can you spot all bad practices/improvements?**

There are some obvious ones and multiple ones which are hard to spot without help...

<p>
<details>
<summary><b>Do you want to know an aproximate number of bad practises to look out for? You will be surprised...</b></summary>
 
> At least **25 bad practices** can be found!

</details>
</p>

The playbook is valid, you may run it and it will build a Docker image and run a container with a web-based Best-Practice Guide.

```bash
ansible-playbook best-practice-container.yml
```

The Guide shows loads of Good and Best Practices for your Ansible projects.

<p>
<details>
<summary><b>Help wanted?</b></summary>
 
Install `ansible-lint` and run it.

```bash
$ pip3 install ansible-lint
Defaulting to user installation because normal site-packages is not writeable
Collecting ansible-lint
  Downloading ansible_lint-6.9.0-py3-none-any.whl (235 kB)
...
Successfully installed ansible-lint-6.9.0
$ ansible-lint best-practice-container.yml
```

</details>
</p>

