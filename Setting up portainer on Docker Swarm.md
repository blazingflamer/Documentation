# How to Set Up Portainer on Docker Swarm

Portainer is a web-based GUI that allows you to easily manage Docker containers, images, networks, and volumes. In this tutorial, we will guide you through the process of setting up Portainer on Docker Swarm.

## Prerequisites

Before you start, you will need:

- A Docker Swarm cluster
- Access to a terminal or command-line interface
- Docker Compose installed on your system

## Step 1: Create Portainer Stack

First, we need to create a Docker stack that will deploy Portainer on our Swarm cluster. Create a new file named `portainer.yml` and add the following contents:

```yml
version: "3.9"

services:
  agent:
    image: portainer/agent:latest
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - /var/lib/docker/volumes:/var/lib/docker/volumes
    networks:
      - agent_network
    deploy:
      mode: global
      placement:
        constraints: [node.platform.os == linux]

  portainer:
    image: portainer/portainer-ce:latest
    command: -H tcp://tasks.agent:9001 --tlsskipverify
    networks:
      - agent_network
    ports:
      - "9443:9443"
      - "9000:9000"
      - "8000:8000"
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock"
      - portainer_data:/data
    deploy:
      replicas: 1
      placement:
        constraints: [node.role == manager]

volumes:
  portainer_data:
    driver: local
    driver_opts:
      type: nfs
      o: nfsvers=4,addr=<nfs-server-ip>,nolock,soft,rw # optional to add on "nfsvers=4"
      device: ":<nfs-path-on-server>" # on synology ":/volume1/<storage>
```

Save the file.

## Step 2: Deploy Portainer Stack

Next, we can deploy the Portainer stack on our Swarm cluster. Run the following command:
```
docker stack deploy --compose-file portainer.yml portainer
```

This will deploy the Portainer service on the manager node of the Swarm.

## Step 3: Access Portainer Web Interface

To access the Portainer web interface, open a web browser and go to `http://<manager-ip>:9000`. You should see the Portainer login page.

## Step 4: Create Admin Account

Create an admin account by following the on-screen instructions.

## Step 5: Connect Portainer to Docker Swarm

Once you have logged in, you need to connect Portainer to the Docker Swarm cluster. Click on the `Endpoints` tab and then click the `Add endpoint` button.

Fill in the form with the following information:

- Name: Swarm
- Type: Docker Swarm
- URL: `tcp://<manager-ip>:2377`
- TLS: None

Click the `Add endpoint` button to save the endpoint.

## Step 6: Manage Docker Swarm with Portainer

Now that Portainer is connected to the Docker Swarm cluster, you can use it to manage your Docker containers, images, networks, and volumes. Click on the `Dashboard` tab to see an overview of your Swarm cluster.

That's it! You have successfully set up Portainer on Docker Swarm and can now use it to manage your Docker containers. 

## Additional if you are going to be using Traefik 

In this version, the labels section contains the Traefik labels that define the routing rules for HTTP and HTTPS traffic. The middlewares section defines some middleware functions to be applied to the requests, such as redirecting HTTP to HTTPS, and adding security headers to the HTTP response.

Replace <nfs-server-ip> with the IP address of your NFS server and <nfs-path> with the path to the NFS share you want to use for storage. Also, replace portainer.example.com with your own domain name or IP address that you want to use to access Portainer.

Save this as portainer.yml and then deploy it with the command docker stack deploy --compose-file portainer.yml portainer.

```yml
version: "3.9"

services:

  agent:
    image: portainer/agent:latest
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - /var/lib/docker/volumes:/var/lib/docker/volumes
    networks:
      - agent_network
    deploy:
      mode: global
      placement:
        constraints: [node.platform.os == linux]

  portainer:
    image: portainer/portainer-ce:latest
    command: -H tcp://tasks.agent:9001 --tlsskipverify
    networks:
      - agent_network
    ports:
      - "9443:9443"
      - "9000:9000"
      - "8000:8000"
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock"
      - "portainer_data:/data"
    deploy:
      replicas: 1
      placement:
        constraints: [node.role == manager]
    # Traefik labels for routing HTTP traffic
      labels:
        - "traefik.enable=true"
        - "traefik.http.routers.portainer.rule=Host(`portainer.example.com`)"
        - "traefik.http.routers.portainer.entrypoints=http"
        - "traefik.http.routers.portainer.middlewares=portainer-https"
        - "traefik.http.middlewares.portainer-https.redirectscheme.scheme=https"
        - "traefik.http.routers.portainer-secure.rule=Host(`portainer.example.com`)"
        - "traefik.http.routers.portainer-secure.entrypoints=https"
        - "traefik.http.routers.portainer-secure.tls=true"
        - "traefik.http.routers.portainer-secure.tls.certresolver=letsencrypt"
        - "traefik.http.routers.portainer-secure.service=portainer"
        - "traefik.http.routers.portainer-secure.middlewares=portainer-headers"
        - "traefik.http.services.portainer.loadbalancer.server.port=9000"
        - "traefik.http.middlewares.portainer-headers.headers.frameDeny=true"
        - "traefik.http.middlewares.portainer-headers.headers.sslRedirect=true"
        - "traefik.http.middlewares.portainer-headers.headers.STSIncludeSubdomains=true"
        - "traefik.http.middlewares.portainer-headers.headers.STSPreload=true"
        - "traefik.http.middlewares.portainer-headers.headers.STSSeconds=15552000"

volumes:
  portainer_data:
    driver: local
    driver_opts:
      type: nfs
      o: nfsvers=4,addr=<nfs-server-ip>,nolock,soft,rw # optional to add on "nfsvers=4"
      device: ":<nfs-path-on-server>" # on synology ":/volume1/<storage>
```

### **Explanation on the traefik labels**

- `traefik.enable=true`: Enables Traefik for this service.
- `traefik.http.routers.portainer.rule=Host(portainer.example.com)`: Defines the routing rule for HTTP traffic. Requests with the Host header set to portainer.example.com will be routed to this service.
- `traefik.http.routers.portainer.entrypoints=http`: Specifies the entry point for HTTP traffic. In this case, the default HTTP entry point is used.
- `traefik.http.routers.portainer.middlewares=portainer-https`: Specifies the middleware function to be applied to HTTP traffic before it is routed to this service. In this case, the portainer-https middleware is used to redirect HTTP traffic to HTTPS.
- `traefik.http.middlewares.portainer-https.redirectscheme.scheme=https`: The portainer-https middleware function that redirects HTTP traffic to HTTPS.
- `traefik.http.routers.portainer-secure.rule=Host(portainer.example.com)`: Defines the routing rule for HTTPS traffic. Requests with the Host header set to portainer.example.com will be routed to this service.
- `traefik.http.routers.portainer-secure.entrypoints=https`: Specifies the entry point for HTTPS traffic. In this case, the default HTTPS entry point is used.
- `traefik.http.routers.portainer-secure.tls=true`: Enables TLS encryption for this service.
- `traefik.http.routers.portainer-secure.tls.certresolver=letsencrypt`: Specifies the certificate resolver to be used for TLS encryption. In this case, the letsencrypt resolver is used, which obtains TLS certificates from Let's Encrypt.
- `traefik.http.routers.portainer-secure.service=portainer`: Specifies the service to be associated with this HTTPS router. In this case, the portainer service is used.
- `traefik.http.routers.portainer-secure.middlewares=portainer-headers`: Specifies the middleware function to be applied to HTTPS traffic before it is routed to this service. In this case, the portainer-headers middleware is used to add security headers to the response.
- `traefik.http.services.portainer.loadbalancer.server.port=9000`: Specifies the port to be used for load balancing traffic to this service. In this case, port 9000 is used.
- `traefik.http.middlewares.portainer-headers.headers.frameDeny=true`: Adds the X-Frame-Options header to the HTTP response with the value DENY, which prevents the page from being embedded in an iframe.
- `traefik.http.middlewares.portainer-headers.headers.sslRedirect=true`: Adds the Strict-Transport-Security header to the HTTP response, which instructs web browsers to always use HTTPS to access this site.
- `traefik.http.middlewares.portainer-headers.headers.STSIncludeSubdomains=true`: Adds the Strict-Transport-Security header to the HTTP response, which includes subdomains in the HTTPS enforcement policy.
- `traefik.http.middlewares.portainer-headers.headers.STSPreload=true`: Adds the Strict-Transport-Security header to the HTTP response, which enables HTTP Strict Transport Security (HSTS) preloading.
- `traefik.http.middlewares.portainer-headers.headers.STSSeconds=15552000`: Adds the Strict-Transport-Security header to the HTTP response, which specifies the duration (in seconds) for which the HSTS policy should be enforced. In this case, 6 months. 
