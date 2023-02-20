# Setup a server with Docker, Traefik Load Balancer / Reverse Proxy with automatic HTTPS with Letsencrypt and Portainer to manage your workloads easily.
# Designed to work WITHOUT Docker Swarm. 
### For monitoring, see my repo https://github.com/conxtor/docker-monitoring (work in progress)

## Traefik and Portainer installation ready. 

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

   `apt install docker-compose-plugin git apache2-utils`

3. Make sure the Docker daemon is running on the host
4. Clone this repository to a location of choice on your host: 

    `git clone https://github.com/conxtor/traefik-letsencrypt-portainerCE-docker-setup.git`
    
5. The DNS of all hostnames you want to serve with Traefik / SSL will need to point to the server where you install this. I am using a wildcard DNS entry so I don't have to maintain each and every hostname manually. Automatic LetsEncrypt certificate issuing and renewal WON'T work without properly configured DNS. 
6. Create two Docker networks named "traefik" and "monitoring". More on that later, but for now, run: 


   `docker network create traefik`


   `docker network create monitoring` 

### 2. Deploy Traefik

---
>  **NOTE**


> Traefik needs a location to store the certificates it will receive from LetsEncrypt. It will store them in a JSON file. If we don´t provide a volume, a restart or redeploy of Traefik will trigger a new certificate request to LetsEncrypt servers. If we do that too often, we may hit LetsEncrypt rate limits. The use of a Docker volume for this purpose is recommended, the solution I prefer (and use in this repo) is a bind mount, which is a path on the host which is mounted in the container like any other volume. I use `\data` and then have subdirectories for each of my apps.


> The same goes for logs, which is taken into consideration here as well. I recommend to store them the same way as indicated above for the LetsEncrypt certificates.

---
    
  1. Change to the directory where you cloned the repository, then change into the folder ´traefik´. 
  2. Copy the `.env_sample` file to `.env`.
  3. Create the two directories `/data/traefik/letsencrypt` and `/data/traefik/logs` like `$ mkdir -p /data/traefik/letsencrypt /data/traefik/logs`. 
  4. Generate a password to protect access to the Traefik dashboard: `echo $(htpasswd -nB user) | sed -e s/\\$/\\$\\$/g` (MD5, secure) or `htpasswd -snb <user> <password>` (SHA, less secure)
  4. Edit the `.env` file and enter your real email for LetsEncrypt registration and certificate expiration reminder emails, Traefik Dashboard hostname and Traefik Dashboard credentials 
  5. Run the following command: `docker compose up -d` to deploy Traefik. 

Part 1 done. Now let´s add Portainer CE, served securely with Traefik, which we just installed.

### 3. Deploy Portainer CE

  1. Change to the directory where you cloned the repository, then change into the folder `portainer`.
  2. Create the directoriy `/data/portainer` like `$ mkdir -p /data/portainer`. 
  3. Copy the `.env_sample` file to `.env`.
  4. Edit the `.env` file and enter the hostname (complete, including domain) you want to use for your Portainer instance. 
  5. Run the following command: `docker compose up -d` to deploy Traefik.
