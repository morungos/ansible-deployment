# ansible-deployment

A comprehensive set of Ansible roles for full application hosting

### Deployment

There are several deployments scripts, depending on the use case. A normal
rolling update of the high-availability deployment works as follows:

    ansible-playbook -i inventories/production/host.ini application.yml

This will loop through the servers, removing each one from the load balancer,
updating it, and then adding it back in. It generally doesn't touch the
databases.

The database update is entirely separate. And it is incomplete, in that changes
to the cluster structure are not handled identically. 

    ansible-playbook -i inventories/production/host.ini --extra-vars "bootstrap=true" bootstrap.yml

### Backups

    ansible-playbook -i inventories/production/host.ini backup.yml
