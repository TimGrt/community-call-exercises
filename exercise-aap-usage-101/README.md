# AAP Usage 101

We will get to know the new Ansible Automation Platform and do some basic stuff.

## Create User

You will login as the `admin` user first, but we will also create a local user, who will only be able to run the job template which we will create during this session. Additionally, he won't have any permissions besides having *read* access to the *Project* or the *Inventory*. He won't be able to edit the job template either.

Go to *Users* and click the *Add* button:

| Parameter | Value |
| --------- | ----- |
| **Username**  | *\<Your CC username>* |
| **Password** | *\<Up to you, choose something easy>* |
| **User Type** | *Normal User* |
| **Organization** | *Default* |

## Create project

Create a new *project* with the following parameters:

| Parameter | Value |
| --------- | ----- |
| **Name**  | *Community Call Project* |
| **Organization** | *Default* |
| **Source Control Type** | *Git* |
| **Source Control URL** | https://github.com/TimGrt/community-call-exercises.git |
| **Options > Clean** | &#9745; |
| **Options > Update Revision on Launch** | &#9745; |

![Create Project](.pictures/AAP-Create-Project.png)

Ensure that the *Git Sync* is successful (*Details > Last Job Status > Successful*).

Add Access permissions to your project for your User, go to your project and select the *Access* tab, press the *Add* button. Your User should have **Read** permissions only!

![Project Access](.pictures/AAP-Project-User-Read.png)

## Create Inventory

Create a new *inventory* with the following parameters:

| Parameter | Value |
| --------- | ----- |
| **Name**  | *Community Call Inventory* |
| **Organization** | *Default* |

Add a new host to your inventory with the following parameters:

| Parameter | Value |
| --------- | ----- |
| **Name**  | *mkdocs-host* |
| **Variables** | *ansible_host: \<IP address of node3 from Workshop inventory>* |

We will use the *node3* from the *Workshop inventory*, go to *Inventories > Workshop Inventory > Hosts > node3* and copy the variables section. Copy the YAML variable to *Inventories > Community Call Inventory > Hosts > mkdocs-host*.

![New Inventory - New host](.pictures/AAP-New-Inventory-new-host.png)

> Do not copy this IP address, yours will be different!

Add Access permissions to the newly created inventory for your user. Your user should have **Read** permissions only.

![Inventory Access](.pictures/AAP-New-Inventory-Access.png)

## Create job template

Create a new *job template* with the following parameters:

| Parameter | Value |
| --------- | ----- |
| **Name**  | *Community Call Playbook Run* |
| **Job Type** | *Run* |
| **Inventory** | *Community Call Inventory* |
| **Project** | *Communtiy Call Project* |
| **Playbook** | *exercise-aap-usage-101/mkdocs-deployment.yml* |
| **Credentials** | *Machine Credentials > Workshop Credentials* |

![Job Template](.pictures/AAP-Template.png)

Set permissions for the job template, your user should be able to **Execute** the template. If you would only set *Read* permissions, your user won't be able to do much...

![Job Template Access](.pictures/AAP-Template-Access.png)

All set, now the fun part begins.
 
## Log out and log in as you personal user

> If you are the `admin` user, log out and log in as your previously created user!

## Run the template

Are you logged in with your personal user?

**Launch the template...**

The playbook ran "successful", but nothing really happened, the play got skipped (*Could not match supplied host pattern*). Fix the error!

<p>
<details>
<summary><b>Help wanted?</b></summary>
 
Remember that every play always targets a group of hosts. Do we even defined a group in our inventory?

</details>
</p>

<p>
<details>
<summary><b>More help wanted?</b></summary>
 
The playbook targets the group `mkdocs`, create this group in your inventory (*Community Call Inventory*) and add `mkdocs-host` to that group. You will need to login as the *admin* user again, because you only have *read* permissions on the inventory.  
Log in as your personal user again and run the job template again.

</details>
</p>

**Launch the template again...**

Still an error, installing stuff fails (*This command has to be run under the root user.*). Well, this error should be easy to fix for you.

<p>
<details>
<summary><b>Help wanted?</b></summary>
 
We need *become* permissions, enable these for the job template.

</details>
</p>

Errors fixed? Look at the last task and input the IP address into your browser.

**Let the discussion begin!**