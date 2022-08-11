# Setup a server with Docker, Traefik Load Balancer / Reverse Proxy with automatic HTTPS with Letsencrypt and Portainer to manage your workloads easily.

For some time, I have been investigating different options for managing multiple services and applications on my host on the Internet.  

Kubernetes or OpenShift / OKR is definitely overkill for my needs, at least at a personal level. [CapRover](https://caprover.com/) is a lightweight PaaS based on Docker Swarm that works great, but has all the limitations of Docker Swarm (concerning privileged containers and some more) and can't be automated end-to-end, including application (service and stack) creation and configuration. 

I also don't need high availability or redundancy for any of the services I run, so I took the approach to going back to simple docker-compose based deployments. I also wanted automatic HTTPS with free certificates from LetsEncrypt. The [Traefik reverse proxy](https://traefik.io/traefik/) handles all this and the virtual hosts for each of the appications, and [Portainer](https://www.portainer.io/) provides a great Web UI to manage Docker containers and compose files.  

There are thousands of docker-compose.yml files available on the internet allowing you to run a lot of applications. Of course, you will need to do some changes to them so that they work with this setup, but I will explain everything step by step here.  

## What will I learn in this repository? 

1. Setup Docker on your server (I use Ubuntu 22.04 LTS, so YMMV)
2. Setup Traefik as a reverse proxy for your applications / workloads on your server with automatic HTTPS via LetsEncrypt. 
3. Deploy PortainerCE (Community Edition) to easily manage your workloads
4. How to modify docker-compose.yml files for automatic HTTPS via Traefik / LetsEncrypt
5. How to deploy using Portainer manually or automatically from a repository. 
   
### 1. Setup the target server and all prerequisites. 

1. Install Docker engine (CE) on your target server. I follow the official description found [here](https://docs.docker.com/engine/install/ubuntu/), instructions for other distributions are available on the same website as well. 
2. Install some dependencies: 

   `apt install docker-compose-plugin`
3. Make sure the Docker daemon is running on the host

### 2. Deploy Traefik and PortainerCE

1. Traefik: 
  1. Traefik needs a location to store the certificates it will receive from LetsEncrypt. It will store them in a JSON file. If we don´t provide a volume, a restart or redeploy of Traefik will trigger a new certificate request to LetsEncrypt servers. If we do that too often, we may hit LetsEncrypt rate limits. The use of a Docker volume for this purpose is recommended, the solution I prefer (and use in this repo) is a bind mount, which is a path on the host which is mounted in the container like any other volume. I use `\data` and then have subdirectories for each of my apps. 
