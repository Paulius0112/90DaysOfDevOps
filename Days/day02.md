# VPC

- Logically isolated part of AWS Cloud where you can define your own network
- the largerst CIDR can be with /16, the lowest with /28
    - Block sizes must be between a /16 and /28 netmask
- 1 subnet is always in one AZ
- Custom VPC creates only *Main route table* and *network access list*
- AWS reserves 5 IP adresses
    - 10.0.0.0 - network address
    - 10.0.0.1 - reserved by AWS for the VPC router
    - 10.0.0.2 - reserved by AWS. IP address of a DNS server
    - 10.0.0.3 - Reserved by AWS for future use
    - 10.0.0.255 - Network broadcast address. We do not support broadcast in a VPC, therefore we reserve this address
- Can only have 1 *internet gateway* per VPC


## Gateway terminology
- *Internet gateway* - AWS VPC side of the connectino to the public internet
- *Virtual private gateway* - VPC endpoint on the AWS side
- *Customer gateway* - representation of the customer end of the connection


## VPC Flow Logs
- Enables capture information about IP traffic within VPC
- Three different levels:
    1. VPC
    2. Subnet
    3. Network Interface level
- You can't enable flow logs for VPC's that are peered with your VPC unless the peer VPC is in your account
- You can't tag a flow log
-  You can't change the configuration of a flow log after it's been created (need to delete and re-create)


## NAT Gateway and NAT instances

| Topic               | **NAT gateway**                   | **NAT instance**                         |
|---------------------|-----------------------------------|------------------------------------------|
| **Managed**         | AWS managed                       | Managed by user                          |
| **Availability**    | Highly available within an AZ     | Not highly available                     |
| **Bandwidth**       | Up to 45 GPS                      | Depends on batdwidth of the EC2 instance |
| **Maintenance**     | Managed by AWS                    | Managed by you                           |
| **Performance**     | Optimized for NAT                 | Custom                                   |
| **Public IP**       | Elastic IP cannot be detached     | Elastic IP that can be detached          |
| **Security Groups** | Cannot associate a security group | Can associate with a security group      |
| **Bastion Host**    | Not supported                     | Can be used                              |
- Enables instanes in private subnet to connect to the internet or other AWS services while preventing the internet from initiating a connection with those instances
- Will always be in a public subnet
- Redundant inside the AZ
- For **Multi-AZ redundancy** of NAT gateway, create gateways in each AZ with routes for private subnets to use the local gateway


## Network ACL

- Optional layer of security for your VPC that acts as a firewall for controlling traffic in and out of one or more subnets
- *First layer of security*, after Network ACL comes security group
- *Default Network ACL* - your VPC automatically comes with a default network ACL, allows all outbound and inbound traffic
- *Custom network ACL* - By default, will deny all inbound and outbound traffic until you add rules
- *Subnet association* - each subnet must be associated with a network ACL. If you don't acssociate, the subnet is automatically associaed with the default network ACL
- *Allows to block IP addresses*, whereas security groups don't
- One ACL can have multiple subnets, byt a subnet can be associated with only 1 network ACL.
- Network ACL contains *numbered lust of rules* - are evaluated in order starting from lowest numbered rule
- Have separate inbound and outbound rules *(stateless)*
    - Each rule can either allow or deny traffic


## VPC Peering

- Allows to connect 1 VPC with another via a direct network route using private IP address
- Instances behave as if they were on the same private network
- You can peer VPCs with other AWS accounts as well as with other VPCs in the same account
- VPC peering can be in different regions (known as **inter-region VPC peering connection**)
    - Data sent between VPCs in different regions is encrypted (traffic charges apply)
- Can only have one peering connection between any two VPCs at a time
- Cannot have overlapping CIDR ranges
- Maximum 50 VPC peers per VPC
- Peering is in a *star* configuration 0 one cetral VPC peers with 4 others
    - *Transit peering* is not supported. Meaning every connection between VPCs goes through central VPC, cannot go directly


## PrivateLink

- Provides a private connectivity between VPCs, AWS services, and on-premises applications, securely on the Amazon network.
- To open our applications to othe VPCs, we can either
    1. *Open the VPC up to the internet*
        - Security considerations; everything in the public subnet is public
        - A lot more to manage
    2. *VPC peering*
        - Management overhead for different peering relationships
        - The whole network will be accessible. Isnt great if u have multiple applications within VOC
    3. *PrivateLink*
        - The best solution to expose a service VPC to customers
        - Doesn't have management overhead
        - Requires a *NLB* on the service VPC and an *ENI* on the customer VPC


## VPC endpoint

- Enables you to privately connect your VPC to supported AWS services and VPC endpoint services powered by **PrivateLink** without requiring an internet gateway, NAT devices, VPN connection, AWS Direct Connect connection
    - Traffic between your VPC and the other service **does not leave** the Amazon network
    - Instances in your VPC **do not require public IP addresses** to communicate with resources in the service
- Enpoints are *Virtual Devices*. They are horizontally scaled, redundant, and highly available VPC components that allow communication between instances in your VPC and services without imposing availability risks or bandwidth constraints on your network traffic (whereas NAT has)
- Two types
    1. *Interface endpoint* - an elastic network interface with a private IP address that serves as an entry point for traffic headed to a supported service. They support a large number of AWS services
    2. *Gateway endpoint* - Agateway that is a target for a specific route. It supports connection to **S3 and DynamoDB**
- You can restrict access to **s3** from specific **VPC endpoint**


## VPN CloudHub

- Connects locations in a hub and spoke manner using AWS **Virtual Private Gateway**
- Use cases: Link remote offices for backup or primary WAN access to AWS resources and each other
- Used for hardware-based VPNs
- Used to connect multiple sites with their own VPN together
- Operates over a public internet, but all traffic between the customer gateway and the AWS VPN CloudHub is encrypted
- Similar to VPC peering


## Direct Connect

- Cloud service solution that makes it easy to establish a dedicated network connection from **on-premise** to **AWS**
- Provides a *private connectivity* - establish private connectivity between AWS and your data center, office, etc..
- Provides better network experience than internet-based connections
- Traffic is not encrypted
- *WHEN?* - Requires a large network link into AWS
- Two types of connections
    1. *Dedicated connection* - a physical ethernet connection associated with a single customer. Customers can request a dedicated connection through the AWS Direct Connect console, CLI, API
    2. *Hosted Connection* - a physical ethernet connection that an AWS Direct Connect Partner provisions on behalf of a customer
- You can connect **VPN** with **Direct Connect** to provide an encrypted connecion
- Useful for high-throughput workloads
- Provides stable and reliable secure connection


## Transit gateway

- Connects VPCs and on-premises networks through a central hub.
- Simplifies your network and puts an end to complex peering relationships
- Acts as a cloud router - each new connection is only made once
- Works on a regional basis, but you can have it across multiple regions
- Can use it across multiple AWS accounts using RAM (Resource Access Manager)


## Global Accelerator

- Creates accelerators to improve availability and performance of your applications for local and global users by directing traffic to optimal endpoinds over AWS global network.
- By defaults gives you two statis IP addresses


## Shared Services VPCs

- Enables subnets to be shared with other AWS accounts within the same AWS organisation. 
- You can create separate Amazon VPCs for each account with the account owner being responsible for connectivity and security of each Amazon VPC


## AWS Managed VPN

- AWS provided network connectivity between two VPCs
- **Pros** - uses AWS backbone without traversing the public internet
- **Cons** - is slower than PrivateLink
- *Virtual Private Gateway* - is required on the AWS side and *Customer Gateway*
is required on the customer side.
- Two tunnels per connection must be configured for redundancy
    - Only uses private IP addresses, cannot access public or static address