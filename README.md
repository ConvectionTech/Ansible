# Ansible Structure

If infrastructures are to be treated as a code than projects that manage them must be treated as software projects. As your infrastructure code gets bigger and bigger you have more problems to deal with it. Code layout, variable precedence, small hacks here and there. Therefore, organization of your code is very important, and in this repository you can find some of the best practices (in our opinion) to manage your infrastructure code. Problems that are addressed are:

* Overall organization
* Usage of variables
* Naming
* Staging
* Complexity of plays
* Installation of ansible and module dependencies


## TL;DR

* Keep all your variables in one place, if possible
* Do not use variables in your play
* Use variables in the roles instead of hard-coding
* Keep the names consistent between groups, plays, variables, and roles
* Different environments (development, test, production) must be close as possible, if not equal
* Use tags in your play


## 1. Directory Layout
This is the directory layout of this repository with explanation.


    production.ini            # inventory file for production stage
    development.ini           # inventory file for development stage
    test.ini                  # inventory file for test stage
    group_vars/
        all/                  # variables under this directory belongs all the groups
            apt.yml           # ansible-apt role variable file for all groups
        webservers/           # here we assign variables to webservers groups
            apt.yml           # Each file will correspond to a role i.e. apt.yml
            nginx.yml         # ""
        postgresql/           # here we assign variables to postgresql groups
            postgresql.yml    # Each file will correspond to a role i.e. postgresql
            postgresql-password.yml   # Encrypted password file
    plays/
        ansible.cfg           # Ansible.cfg file that holds all ansible config
        webservers.yml        # playbook for webserver tier
        postgresql.yml        # playbook for postgresql tier

    roles/
        roles_requirements.yml# All the information about the roles
        external/             # All the roles that are in git or ansible galaxy
                              # Roles that are in roles_requirements.yml file will be downloaded into this directory
        internal/             # All the roles that are not public

    extension/
        setup/                 # All the setup files for updating roles and ansible dependencies


## 2. Keep your plays simple
If you want to take the advantage of the roles, you have to keep your plays simple.
Therefore do not add any tasks in your main play. Your play should only consist of the list of roles that it depends on. Here is an example:

```
---

- name: postgresql.yml | All roles
  hosts: postgresql
  sudo: True

  roles:
    - { role: common,                   tags: ["common"] }
    - { role: ANXS.postgresql,          tags: ["postgresql"] }
```

As you can see there are also no variables in this play, you can use variables in many different ways in ansible, and to keep it simple and easier to maintain do not use variables in plays. Furthermore, use tags, they give wonderful control over role execution.


## 3. Stages
Most likely you will need different stages (e.g. test, development, production) for the product you are either developing or helping to develop. A good way to manage different stages is to have multiple inventory files. As you can see in this repository, there are three inventory files. Each stage you have must be identical as possible, that also means, you should try to use few as possible host variables. It is best to not use at all.


## 4. Variables
Variables are wonderful, that allows you to use all this existing code by just setting some values. Ansible offers many different ways to use variables. However, soon as your project starts to get bigger, and more you spread variables here and there, more problems you will encounter. Therefore it is good practice to keep all your variables in one place, and this place happen to be group_vars. They are not host dependent, so it will help you to have a better staging environment as well. Furthermore, if you have internal roles that you have developed, keep the variables out of them as well, so you can reuse them easily.


## 5. Name consistency
If you want to maintain your code, keep the name consistency between your plays, inventories, roles and group variables. Use the name of the roles to separate different variables in each group. For instance, if you are using the role nginx under webservers play, variables that belong to nginx should be located under *group_vars/webservers/nginx.yml*. What this effectively means is that  group_vars supports directory and every file inside the group will be loaded. You can, of course, put all of them in a single file as well, but this is messy, therefore don't do it.