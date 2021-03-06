# Ansible playbook for Docker Swarm using Tinc
This ansible playbook creates a Docker Swarm that uses a Tinc vpn-mesh. It's purpose is to work cross-geolocation and cross-provider.

When using the makefile to deploy the following will happen:
- terraform will use the vps-instances.tf configuration to spin up a number of vps instances.
- an ansible inventory is dynamically created from the manually configured secrets/hosts file and the terraformed vps-instances.
- all nodes get some initial hardening. A user will be created using the local $USER env variable.
- a tinc vpn mesh will be created between all the nodes.
- Glusterfs will be set up for replicated storage across nodes using the tinc vpn mesh.
- docker swarm will be setup to use the tinc vpn mesh for communication between swarm nodes.
- initial admin stacks are deployed on the swarm and will be made available through reverse-proxy.

Requirements:
- Terraform
- Python-pip
- duckdns account

### Execute deployment

Setup a CNAME record in your domain registrars dns records pointed to your duckdns subdomain. The domain name will be used for all swarm services. Using duckdns links the domain dynamically to the swarm's ip address.
```
name: *
type: CNAME
value: my-duckdns-subdomain.duckdns.org.
```
Make sure to include the dot at the end.

- Create a hosts file in the secrets folder using hosts.default as a template.
- Adjust vps-instances.tf to your liking.

- Set the following environment variables:
    - DO_TOKEN: Digital Ocean API token
    - VULTR_TOKEN: Vultr API token
    - DUCKDNS_TOKEN: DuckDNS API token
    - ADMIN_PASSWORD: admin password for swarm services

Run:
```
$ make deploy
```

### Backups

Backup your certs to the local secrets folder by running:
```
$ make backup
```
These certs will get automatically restored on the next deployment.

### Destroy deployment

To destroy the swarm run:
```
$ make destroy
```
Before destroying the nodes, the backup process for the certs will run.
If the backup process fails (possibly by interupting deployment process), the nodes won't be destroyed. In that case run:
```
make force-destroy
```

Additionally to remove any local temporary files (terraform data, ansible data, etc), run:
```
$ make clean
```


#### Security todo:
- Set "PermitRootLogin no" after becoming different user in playbook
- Don't allow ssh as root; implement ansible user that becomes root
- Check services and implement non-root user where possible
- Block docker from adjusting iptables, and open up ports manually where required
- Docker overlay network with encryption? (probably not applicable since this already uses tinc vpn)
- tls verification for docker? (probably not applicable since this already uses tinc vpn)
- use ansible vault for secrets
- use chronyd to synchronize time between nodes
- implement admin vpn for secure access to admin services
