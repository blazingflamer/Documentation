# How to Install Docker and Docker Swarm on Ubuntu

Docker is a popular containerization platform that allows you to easily deploy and manage applications in isolated environments. Docker Swarm is a clustering and scheduling tool for Docker containers. In this tutorial, we will guide you through the process of installing Docker and Docker Swarm on Ubuntu.

## Prerequisites

Before you start, you will need:

- An Ubuntu server with sudo privileges
- Access to a terminal or command-line interface

---
## Step 1: Remove Old Versions of Docker

First, we need to remove any old versions of Docker that may be installed on the system. Run the following command:



```
sudo apt-get remove docker docker-engine docker.io containerd runc
```

## Step 2: Install Dependencies

Next, we need to install the necessary dependencies for Docker. Run the following command:

```
sudo apt-get update
```
```
sudo apt-get install \
ca-certificates \
curl \
gnupg 
```

## Step 3: Add Docker GPG Key

To verify the authenticity of the Docker packages, we need to add the Docker GPG key to our system. Run the following command:

```
sudo install -m 0755 -d /etc/apt/keyrings 
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
```

## Step 4: Add Docker Repository

Next, we need to add the Docker repository to our system. Run the following command:

```
echo \
"deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
$(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

## Step 5: Install Docker

Finally, we can install Docker on our system. Run the following command:

```
sudo apt update
```

```
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
```



## Step 6: Set Up Docker Swarm

Now that we have Docker installed, we can set up Docker Swarm. First, we need to initialize the Swarm. Run the following command on the master node:

```
sudo docker swarm init
```

This will output a command that you can use to add worker nodes to the Swarm.

## Step 7: Add Worker Nodes

To add worker nodes to the Swarm, run the command that was output by the `docker swarm init` command on each worker node.
```
sudo docker swarm join --token <TOKEN> <MASTER-IP>:<PORT>
```


## Step 8: Verify the Swarm

To verify that the Swarm is running correctly, run the following command on the master node:

```
sudo docker node ls
```


This should output information about the nodes in the Swarm.

That's it! You have successfully installed Docker and Docker Swarm on your Ubuntu server. You can now start using Docker to create and manage containers in a Swarm environment.




