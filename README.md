# Deploy Traefik v2 in Docker Swarm

References:
* [Docker Documentation](https://docs.docker.com)
* [Traefik Documentation](https://doc.traefik.io)

## Getting Started

Install Docker for Debian using Ansible (prepare your inventory file accordingly)

```
ANSIBLE_CALLBACK_WHITELIST=profile_tasks ANSIBLE_PYTHON_INTERPRETER='/usr/bin/env python3' \
ansible-playbook -i inventory playbooks/install-docker-debian.yaml -vKD
```

Login to the docker server then initialize Docker Swarm

```
docker swarm init
docker node ls
```

Create Docker network for Traefik

```
docker network create --driver=overlay traefik
```

Add custom label to the Docker node

```
export NODE_ID=$(docker info -f '{{.Swarm.NodeID}}')
docker node update --label-add traefik.certificates-data=true $NODE_ID
```

Prepare environment variables and hashed password for Traefik dashboard (you will be prompted to create password)

```
export DOMAIN=example.com
export EMAIL=admin@example.com
export USERNAME=admin
export HASHED_PASSWORD=$(openssl passwd -apr1)
```

Deploy Traefik

```
docker stack deploy -c stacks/mytraefik.yml mytraefik
# OR
docker stack deploy -c stacks/mytraefik-host.yml mytraefik
```

Check if Traefik already deployed

```
docker stack ls
docker stack ps mytraefik
docker service ls
docker service logs -f mytraefik_traefik
```

Access the Traefik dashboard [https://mytraefik.example.com](https://mytraefik.example.com)

## Deploy Simple Service and Serve using Traefik with HTTPS

Deploy service

```
docker stack deploy -c stacks/myservice.yml myservice
```

Check if the service already deployed

```
docker stack ls
docker stack ps myservice
docker service ls
docker service logs -f myservice_whoami
```

Access the service [https://myservice.example.com](https://myservice.example.com)
