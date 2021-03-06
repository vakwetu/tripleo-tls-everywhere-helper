Role Name
=========

This is a set of helper ansible roles for validating and guiding you
through a TripleO TLS everywhere deployment.

As shown in the example playbook below, the three playbooks are
tls-everywhere-helper-common, tls-everywhere-helper-undercloud and
tls-everywhere-helper-overcloud.

The tls-everywhere-helper-common playbook defines some parameters needed
by the other playbooks and should always be included as shown. 


Role Variables
--------------

* `helper_report_path`: Is the path where the report of the helper run will
                        be stored. Defaults to **/tmp/report.txt**.


Example Playbook
----------------

    ---
    - hosts: Undercloud
      roles:
        - tls-everywhere-helper-common
        - tls-everywhere-helper-undercloud

    - hosts: overcloud
      roles:
        - tls-everywhere-helper-common
        - tls-everywhere-helper-overcloud

Running the playbooks
---------------------

These playbooks is targeted at TripleO deployments. The following assumptions
are made:

* You are running this playbook from the undercloud host.
* You are using a dynamic inventory that includes all the hosts in TripleO
  (including the undercloud)
* The dynamic inventory was created with the `tripleo-ansible-inventory` tool.

It is assumed that these playbooks will be run from the undercloud.

### Using the dynamic inventory

As mentioned above, it is recommended that you use the
`tripleo-ansible-inventory` tool in order to run this playbook. When running
this tool, you might need to adjust the user for the undercloud.

You can run the tool as follows:

```
tripleo-ansible-inventory --static-yaml-inventory hosts.yaml
```

This will create an inventory file called **hosts.yaml**.

Edit this file to have the undercloud `ansible_ssh_user` match what you're using.

Finally, you can run the playbooks as follows:

```
ansible-playbook -i hosts.yaml tripleo-tls-everywhere-helper/verify-tls-everywhere.yaml
```

A report will be created in the path that `helper_report_path` specifies.

### Manually writing an inventory

Note that it is also possible to write an inventory manually in order to run this.

For the undercloud; It would look as follows:

```
Undercloud:
  hosts:
    undercloud: {}
  vars:
    ansible_host: localhost
    ansible_ssh_user: stack
```

The main thing to note is to have a group called `Undercloud`.

Working with tags
-----------------

This role can be used in different stages of the deployment. This behavior is
controlled with Ansible tags. The default behavior (with no tags specified) is
to run all of the validations.

Here is a description of the existing tags, as well as what they mean in your
deployment:

* `pre-undercloud-deploy`: This is a stage where the user is about to deploy the
                           undercloud. This will check for correct settings in
                           the undercloud.conf file. Note that this will only
                           be executed in the undercloud.

* `post-undercloud-deploy`: This is a stage where the user has already deployed
                            the undercloud. This will check that the undercloud
                            has been correctly enrolled to IdM/FreeIPA and can
                            communicate with it. Note that some of these checks
                            may also run on the overcloud hosts if these have
                            been deployed and were included in the Ansible
                            inventory.

* `post-overcloud-deploy`: This is a stage where the user has already deployed
                           the overcloud. This will potentially tell you how
                           to resolve common issues with the overcloud deployment
                           related to TLS everywhere. This will check that the nodes
                           have been enrolled to IdM/FreeIPA, and have
                           been configured correctly. This will also check that
                           the certificates have been issued and are in an
                           acceptable state (MONITORING).

Reading/Parsing the report
--------------------------

The report has a default format that looks as follows:

```
* BEGIN <name of the check>
  [<status>] <status explanation>
  - RECOMMENDATION: <recommendation text>
* END <name of the check>
```

Note that the recommendations will only appear if there were issues found in
the deployment.

This format is meant to be easily parsed with the `grep` command.

For instance, to see all the checks that passed, you can do:

```
grep OK /tmp/report.txt
```

To see all the checks that didn't pass, you can do:

```
grep ERROR /tmp/report.txt
```

To see all the recommendations about the errors, you can do:

```
grep RECOMMENDATION /tmp/report.txt
```

License
-------

Apache License 2.0

Author Information
------------------

* [Juan Antonio Osorio Robles](https://jaormx.github.io/)
