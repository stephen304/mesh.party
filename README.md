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

From the perspective of a user, they receive DNS records with A and AAAA records to all the ingress nodes which will relay to the backend. If the clearnet WAN is down, the cloud ingress node will be able to route the request to the local proxy via the Yggdrasil mesh provided that an alternate internet connection provides a path across the mesh to the local proxy. Users with Yggdrasil access will use the mesh and not require a clearnet connection provided that a path through the Yggdrasil mesh is available, bypassing the cloud ingress nodes.

## Usage

* Create an ingress node in a cloud provider
* Create a local reverse proxy with ports forewarded to it
* Add both to the hosts inventory
* Define WAN IPs of the local network in `group_vars/ingress.yml`
* Run the playbook
* Add the yggdrasil IP of the local proxy as a backup server in `group_vars/ingress.yml`
* Run the playbook again
* Add the ipv4/6 adresses of the cloud ingress nodes as A/AAAA records for any domain you want to use
* Add the Yggdrasil IP of the local proxy as an AAAA record to allow mesh users to access the local proxy using mesh routing
* Confirm that visiting the configured domains returns either a cloud node IP or the local proxy mesh IP
* Configure certbot on the local proxy for SSL termination

## Status

- [x] Ingress Nodes
  - [x] Haproxy
    - [x] Load balancing
    - [x] Mesh backup
    - [ ] Parametric backend config
  - [x] Yggdrasil
- [ ] Local Proxy
  - [ ] Haproxy
    - [ ] Certbot
    - [ ] Parametric service config?
  - [x] Yggdrasil
- [ ] Bonus?
  - [ ] Automated DNS
