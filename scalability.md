# Scalability

1. A service is said to be scalable if when we increase the resources in a system, it results in increased performance in a manner proportional to resources added. Generally, increasing performance means serving more units of work, but it can also be to handle larger units of work, such as when datasets grow.

1. In distributed systems there are other reasons for adding resources to a system
   * For example to improve the reliability of the offered service. Introducing redundancy is an important first line of defense against failures. 
   * An always-on service is said to be scalable if adding resources to facilitate redundancy does not result in a loss of performance.
1. For the systems we build we must carefully inspect along which axis we expect the system to grow, where redundancy is required, and how one should handle heterogeneity in this system


## Types of Scaling
### Vertical Scaling
1. Increase the power of CPU, storage of Disk (PATA, SATA-7200rmp,SAS-15000rpm, SSD), capacity of Memory
2. Constrained by real-world limits

### Horizontal Scaling
1. Increase the number of servers, each may still has a unique IP address
2. HTTP requests now need to be distributed across the servers through a load balancer
3. DNS instead of returning the IP address of the server 1 or 2 etc., now returns the IP address of the 'load balancer'. So the load balancer now has the public IP address.
   * The back-end servers now have private IP addresses, which can't be seen by public 
   * Load balancer can now send requests to the servers using TCP/IP, 
4. It's not a good idea to put DB server on the same web server, as a user can connect to another server, which doesn't have his/her information there.

## Rules for Scalibility
1. The first golden rule for scalability: Every server contains exactly the same codebase and does not store any user-related data, like sessions or profile pictures, on local disc or memory. 
   * A user expects to get the same results back regardless of which server he "lands on" 

## Load Balancer
## Caching
## Replication
## Partition
## High Availability

## Security
1. Types of traffic allowed to come in
   * TCP:80
   * TCP:443: default port for SSL for HTTP based urls
   * TCP:22, for SSH or SSL based VPN to connect to the DC remotely
3. Types of traffic from Load Balancer to web servers (TCP:80)
   * It's common to offload your SSL to the load balancer or some special device to keep everything else unencrypted as you control it.
   * So everything from LB down stream can go as HTTP unencrypted, so you don't need put your SSL certificates on all the web servers. You can just put them on the load balancers.
4. Traffic from web servers to databases
   * TCP:3306, default MySQL port

# Performance vs Scalability

1. If you have a performance problem, your system is slow for a single user.
1. If you have a scalability problem, your system is fast for a single user but slow under heavy load.



# Resources:
1. [CS75 (Summer 2012) Lecture 9 Scalability Harvard Web Development David Malan](https://youtu.be/-W9F__D3oY4)
1. [Scalability for Dummies](https://www.lecloud.net/tagged/scalability)
1. [A Word on Scalability](https://www.allthingsdistributed.com/2006/03/a_word_on_scalability.html)
