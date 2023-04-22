# **Running portainer agent on Synology NAS.**

If you want to manage Docker on your Synology NAS, from a Portainer instance running somewhere else, you need to deploy the Portainer agent on your Synology NAS.

The installation guideline for running the agent on a standalone Docker host, can be found at [Portainer.io](https://www.portainer.io/installation/)

A small correction is needed, as at the Synology DSM, Docker directories are not located at `/var/lib/docker`, but at `/volumeX/@docker`.

So, to run the Portainer agent, SSH into your Synology NAS and run the following command. Replace volume1 with whatever your volume name is.

```
sudo docker run -d -p 9001:9001 --name portainer_agent --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v /volume1/@docker/volumes:/var/lib/docker/volumes portainer/agent
```