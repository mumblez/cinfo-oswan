# cinfo-oswan

# Task
Imagine we need to setup a VPN connection between our AWS account to one of our partner's data centre so that services in our AWS account can call API's in our partner's data centre (IP address = 10.10.10.10) .

We would like to install two OpenSWAN instances (Private IP address = 20.20.20.20, 20.20.20.21), and setup two redundant IPSEC tunnels in a high availability configuration, however the services which run in our side are not aware of the redundant configuration and only knows the target IP address (10.10.10.10)

Design a high availability solution which support this need and describe in details  the architecture, network configuration and services that you use or develop. If you need to write scripts, you can use your favourite scripting language.

---

A non openSWAN alternative is to use the vpn gateway feature already available on AWS - [AWS VPN Gateway](http://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/VPC_VPN.html)
This assumes we have access to the customers router, if so we could add a vpn gateway and connection within AWS and avoid maintaining an openSWAN instance, we could create 2 connections for HA and use BGP as default to let it figure out the routes and it would provide automatic failover (assign unique BGP numbers for each VPN connection), also we would have to know the customers router IP address assuming 10.10.10.10 is a server (API service we're accessing)

---

An openSWAN solution, using [this guide](http://walkintocloud.com/index.php/2016/06/16/openswan-connecting-two-vpcs-different-regions-amazon-aws/) or the official [AWS VPN openSWAN guide](https://aws.amazon.com/articles/5472675506466066) allows us to connect multiple AWS regions (VPC's) together but in our situation we would like to connect to the customers router with our own openSWAN instances, for a rough outline we will assume the customer's router IP address is 10.10.10.1 with a public IP address of 12.12.12.12

# Settings (much of this info has been copied from the above mentioned guides!)

_Customer_

Router Public IP: 12.12.12.12

Router Private IP: 10.10.10.1

CIDR: 10.10.0.0

_Our Settings_

| CIDR | Subnet | Instance Name | Private IP | Public IP (EIP) |
| ---- | ------ | ------------- | ---------- | --------------- |
| 20.20.0.0/16 | 20.20.20.0/24 | VPN1 | 20.20.20.20 | 5.5.5.1 |
| 20.20.0.0/16 | 20.20.20.0/24 | VPN2 | 20.20.20.21 | 5.5.5.2 |

(Setup the Private IPs on ENI's so we can re-use if instance goes down and re-use same config)

## Security Groups (ICMP / Ports)

| Protocol | Port | Source |
| --- | --- | --- |
| TCP | 50   | 5.5.5.1/32 |
| TCP | 51   | 5.5.5.1/32 |
| UDP | 4500 | 5.5.5.1/32 |
| UDP | 500  | 5.5.5.1/32 |
| TCP | 50   | 5.5.5.2/32 |
| TCP | 51   | 5.5.5.2/32 |
| UDP | 4500 | 5.5.5.2/32 |
| UDP | 500  | 5.5.5.2/32 |

## Network ACLs

| Destination | Target |
| --- | --- |
| 10.10.0.0/16 | 20.20.20.20 |
| 10.10.0.0/16 | 20.20.20.21 |

*Need to test this but I assume if the first route fails the second one will kick in*

## openSWAN setup

Following this guide - [AWS VPN](https://aws.amazon.com/articles/5472675506466066) we will replace EIP1 and VPC1 with our own details for each instance and EIP2 and VPC2 with the customer's details.
In the configuration files, anything starting with *left* is us and *right* is the customer!

... work in progress ...
### Notes / Refs
http://www.briantobin.org/2013/02/20/ipsec-s2s-vpn-via-aws-openswan-and-cisco-ipsec-router
https://supportcenter.checkpoint.com/supportcenter/portal?eventSubmit_doGoviewsolutiondetails=&solutionid=sk100726 (native AWS vpn)
https://heitorlessa.com/working-with-amazon-aws-vpc-software-based-vpn-part-1-f540a70c60ab#.a8054rllt (seems ideal)

