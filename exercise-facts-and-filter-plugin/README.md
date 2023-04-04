# Custom facts and custom plugins

The following exercises will cover extending Ansible Core functionality with self-developed custom facts and plugins. You will write some Python code, don't worry, we will guide you through the *complicated* stuff.

**Good luck!**

The folder contains a playbook (`playbook.yml`) and a inventory file (`inventory.ini`) with a group `test`.  
The inventory group only contains your own host (Ansible Core Control node), no additional host is needed.

## Task 1

Add a *static* custom fact for localhost, which should identify your host as part of the `development` stage.

Running the playbook should result in this output:

```bash
TASK [Output host identifier from custom fact] *********************************************************************
ok: [localhost] => {
    "msg": {
        "identifier": {
            "environment": {
                "stage": "development"
            }
        }
    }
}
```

Take a look at the [Ansible Best Practice Guide](https://timgrt.github.io/Ansible-Best-Practices/development/extending/#static-facts) on how to create a custom static fact.

> Note: The file-extension must be `.fact`! 

To test your custom fact, either run the playbook or use an *ad-hoc* command. Facts are gathered by the `ansible.builtin.setup` module, you can provide a *filter* to the module, if you want to output only specific facts.

<p>
<details>
<summary><b>Help wanted?</b></summary>

Use the following ad-hoc command to output custom facts:

```bash
ansible -i inventory.ini test -m ansible.builtin.setup -a filter=ansible_local
```

</details>
</p>

## Task 2

The `playbook.yml` contains a list of hostnames, you need to add the domain to every hostname. The domain should be the value of the *stage* fact from the previous task. For example, the host `node1` in the list should be adjusted to `node1.development`.  
This must be achieved with a *custom filter plugin*.

Create a new collection `computacenter.utils` **in your projects folder**, the [Ansible Best Practice Guide](https://timgrt.github.io/Ansible-Best-Practices/development/extending/#store-custom-content) is helpful here, too.

Create the correct folder for the plugin type and create the Python file containing the code for your filter (copy the [example filter](https://timgrt.github.io/Ansible-Best-Practices/development/extending/#filter-plugins) from the Best Practice Guide as a starting point).

The `playbook.yml` contains a task with a list of three (unsorted) IPs, the filter plugin brings them in order. Run the playbook!

> Note: The filter plugin example needs the Python package `netaddr`, this is installed by the playbook. Manual install can be done with `pip3 install netaddr --user`.

<p>
<details>
<summary><b>Help wanted?</b></summary>

Ensure that your project has the following structure:

```bash
.
├── README.md
├── collections
│   └── ansible_collections
│       └── computacenter
│           └── utils
│               ├── README.md
│               ├── galaxy.yml
│               └── plugins
│                   ├── README.md
│                   └── filter
│                       └── cc_filter_plugins.py
├── inventory.ini
└── playbook.yml
```

</details>
</p>

## Task 3

The new filter should be called `add_stage_as_domain`. Add a new function (a function is defined using the `def` keyword) which expects two arguments (multiple arguments are separated with a comma) and returns a list.  
The first argument is a list of strings (e.g. called `input_list`, this gets *the hosts* later), the second argument is a string variable a (e.g. called `identifier` or `domain`, this gets *the domain* later).  

<p>
<details>
<summary><b>Help wanted?</b></summary>

This is the defintion start:

```python
def add_stage_as_domain(input_list, identifier):
```

</details>
</p>

The function should return another list (e.g. called `output_list`), with the string of the second argument appended to every list item. The logic to achieve this, is the following, include it in your Python code:

```python
map(lambda orig_item: orig_item + '.' + identifier, input_list)
```

You may need to adjust the variables, if you used other names.

Adjust the fifth task in the playbook, add the missing `msg` parameter of the *debug* module.

Running the playbook should result in the following output:

```bash
$ ansible-playbook -i inventory.ini playbook.yml 

PLAY [Community Call Dojo - Custom facts and custom filter plugin] *************************************************

TASK [Gathering Facts] *********************************************************************************************
ok: [localhost]

TASK [Output host identifier from custom fact] *********************************************************************
ok: [localhost] => {
    "msg": {
        "identifier": {
            "environment": {
                "stage": "development"
            }
        }
    }
}

...

TASK [Add stage fact as domain with custom filter plugin and output result] ****************************************
ok: [localhost] => {
    "msg": [
        "node1.development",
        "node4.development",
        "node3.development",
        "node2.development"
    ]
}

...

PLAY RECAP *********************************************************************************************************
localhost                  : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

```

> Filter plugin not found? Remember to use FQCN not only for your modules...

## Task 4

As you can see, the hosts are not sorted correctly. Can you implement another filter plugin called `sort_hosts` which brings them in the correct order?

Use the example filter which sorts IP addresses as a reference, the Python builtin `sorted` function can be used. Add a new function which expects a list (the unsorted list) as the only argument and returns another list (the now sorted list).

In your playbook, chain both filter plugins together, like this:

```yaml
- name: Sort hosts with custom filter plugin
      ansible.builtin.debug:
        msg: "{{ host_list | filter1 | filter2 }}"
```

Adjust the sixth task, the result should look like this:

```bash
TASK [Sort hosts with custom filter plugin and output result] *************************************
ok: [localhost] => {
    "msg": [
        "node1.development",
        "node2.development",
        "node3.development",
        "node4.development"
    ]
}
```
