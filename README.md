# ansible-deployment

A comprehensive set of Ansible roles for full application hosting

## Overview

For a high-level picture, read [the blog post, *Stuart’s guide to
high-availability services on a
budget*](https://morungos.com/2024/08/14/cheap-infrastructure/). 

And be ye warned, this will not work out of the box. I just lifted it from my
old production systems, so it will definitely be coupled to the platforms.
Having said that, here are some comments.

1. The role `build` is very NodeJS-specific. It's run on your local system, and
can do whatever it needs to, to assemble and package your application for
deployment. This could easily be skipped, if, for example, you use Github
Actions for that. If I were doing it again today, I'd do that.

2. It is probably quite Debian-specific. Some of the tasks and a lot of the
paths may change if you do something else.

3. Certain variables, such as `digital_ocean_api_key`, will need to be set. I'd
normally do that through encrypting my `inventories/<mode>/group_vars/all.yml`
using `ansible vault` commands, but I'd check them in. The `.vault-password`
file then becomes your master password, if you like, so don't commit whatever
you do. Some of the variables default sensibly, but many, like domain names,
cannot.

4. The `bootstrap.yml` plays are there to drop infrastructure stuff like firewalls
and Samhain intrusion detection. These are then assumed by the `application.yml`. 
But -- you don't need to do the `bootstrap.yml` for updates, and it is done this
way to make updates faster, as they don't re-deploy things like that. 

5. This is far from perfect. It is not fit for purpose as a general re-usable
block of Ansible. 


## Deployment

There are several deployments scripts, depending on the use case. A normal
rolling update of the high-availability deployment works as follows:

    ansible-playbook -i inventories/production/host.ini application.yml

This will loop through the servers, removing each one from the load balancer,
updating it, and then adding it back in. It generally doesn't touch the
databases.

The database update is entirely separate. And it is incomplete, in that changes
to the cluster structure are not handled identically. 

    ansible-playbook -i inventories/production/host.ini --extra-vars "bootstrap=true" bootstrap.yml

## Backups

    ansible-playbook -i inventories/production/host.ini backup.yml
