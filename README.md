# Mesh.Party Reverse Proxy Playbook

This playbook assists with managing a reverse proxy setup for hosting services from home.

## Goals

* Make selfhosted services available on both clearnet and Yggdrasil mesh
* Take advantage of multi-wan to load balance traffic
* Provide redundancy with regard to cloud providers
* Provide self-healing mesh failover to backend when clearnet is disrupted
* Make it easier to bring additional ingress nodes online

## Design

* Each ingress node runs haproxy and yggdrasil
    * Haproxy load balances requests and routes them to the local reverse proxy using multiple WAN connections
    * Yggdrasil provides a backup mesh connection to the local proxy
* The local proxy also runs haproxy and yggdrasil as well as certbot
    * Haproxy terminates SSL here
    * Yggdrasil serves the backup connection as well as provides access to services to users with a mesh connection
    * A firewall blocks connections to the clearnet WANs unless the source IP is from an ingress node (using auto updating aliases via DNS)

From the perspective of a user, they receive DNS records with A and AAAA records to all the ingress nodes which will relay to the backend. If the clearnet WAN is down, the user will be able to connect to the backup mesh provided that an alternate connection is meshed with the local proxy. Users with Yggdrasil access will use the mesh and not require a clearnet connection provided that a path through the Yggdrasil mesh is available.
