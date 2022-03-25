## Run the playbook


```
cd /deploy/ansible
ansible-playbook -i inventories/cuba.marlonfalcon.com -e domain_name=cuba.marlonfalcon.com playbook-master.yml
```

```
http://traefik.cuba.marlonfalcon.com/dashboard/#/
http://swarm.cuba.marlonfalcon.com/
```