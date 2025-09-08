NODEJS
=================================

Implements [Node JS](https://nodejs.org/es/) and NPM on system.

Works with

- Debian and Ubuntu;
- RHEL Distros (RedHat, Fedora, CentOs);

Tasks
-----

- Install packages;


Role Variables
--------------

See [defaults/main.yml](defaults/main.yml). For specific values for your servers, create a file on [environments/senseinode](environments/senseinode) folder with your own values.
This role needs a group `[npm]` to specific route.

* host_vars/ : for specific host values
* group_vars/ : for specific values in a buch of hosts


Variable | Description
------| ------------
nodejs_version | NodeJS version to be installed. 
npm_config_prefix | he directory for NPM global installations.
nodejs_npm_global_packages | Define a list of global packages to be installed with NPM.




Playbook
----------------

playbook.yml

```
- hosts: programming-languages
  roles:
    - nodejs
```


Example running
----------------


```
ansible-playbook -i environments/senseinode/hosts playbook.yml -t nodejs -l <host-name>
```



Author Information
------------------

Rickk Barbosa <ricardo@senseinode.com>
