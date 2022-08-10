# Setup a server with Docker, Traefik Load Balancer / Reverse Proxy with automatic HTTPS with Letsencrypt and Portainer to manage your workloads easily.

For some time, I have been investigating different options for managing multiple services and applications on my host on the Internet.  

Kubernetes or OpenShift / OKR is definitely overkill for my needs, at least at a personal level. [CapRover](https://caprover.com/) is a lightweight PaaS based on Docker Swarm that works great, but has all the limitations of Docker Swarm (concerning privileged containers and some more) and can't be automated end-to-end, including application (service and stack) creation and configuration. 

I also don't need high availability or redundancy for any of the services I run, so I took the approach to going back to simple docker-compose based deployments. I also wanted automatic HTTPS with free certificates from LetsEncrypt. The [Traefik reverse proxy](https://traefik.io/traefik/) handles all this and the virtual hosts for each of the appications, and [Portainer](https://www.portainer.io/) provides a great Web UI to manage Docker containers and compose files.  

There are thousands of docker-compose.yml files available on the internet allowing you to run a lot of applications. Of course, you will need to do some changes to them so that they work with this setup, but I will explain everything step by step here.  

