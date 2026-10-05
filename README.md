<img src="https://cdn.prod.website-files.com/677c400686e724409a5a7409/6790ad949cf622dc8dcd9fe4_nextwork-logo-leather.svg" alt="NextWork" width="300" />

# Creating a Private Subnet

**Project Link:** [View Project](https://nextwork.ai/projects/4e96e29d-98b8-551e-ae93-da39c36e30fd)

**Author:** Maria Jasmin Ahorro  
**Email:** jasminahorro23@gmail.com

---

![Image](https://nextwork.ai/overjoyed_magenta_proud_yak/uploads/4e96e29d-98b8-551e-ae93-da39c36e30fd_afe1fdbd)

## Introducing Today's Project!

### What is Amazon VPC?

Amazon VPC is a virtual network environment inside AWS that provides private, custom-defined networking infrastructure isolated from other AWS tenants. It is useful because it allows you to customize your network architecture—such as creating public and private subnets, managing traffic routing via gateways, and enforcing multi-layered security with Network ACLs and Security Groups—ensuring sensitive resources remain secure while supporting seamless cloud connectivity.

### How I used Amazon VPC in this project

In today's project, I used Amazon VPC to design and deploy a multi-tier network environment (NextWork VPC). I established network segmentation by pairing a public subnet (10.0.0.0/24) with an Internet Gateway and creating an isolated private subnet (10.0.1.0/24). By enforcing strict routing via a dedicated private route table and layering stateless packet filtering with a custom Network ACL, I successfully protected backend resources from direct public internet exposure while mastering AWS VPC security controls.

### One thing I didn't expect in this project was...

One thing I didn't expect in this project is that subnets within a VPC remain associated with the default main route table unless explicitly attached to a new one. Creating a dedicated private route table without an Internet Gateway target was crucial to ensuring complete isolation for our private subnet.

### This project took me...

This project took me approximately 30 minutes to set up and configure, including provisioning the private subnet, route table, and Network ACL rules in AWS.

## Private vs Public Subnets

The difference between public and private subnets is that public subnets are associated with a route table that routes internet traffic directly through an Internet Gateway, allowing resources within them to send and receive traffic from the public internet. Private subnets are isolated from direct internet access because their route tables do not contain a route to an Internet Gateway.

Having private subnets is useful because they enforce strict defense-in-depth network security by isolating sensitive AWS resources from direct internet access. While public subnets handle external-facing traffic (like web servers), private subnets keep backend components (like database clusters and application servers) safely hidden from public scanning and unauthorized access, ensuring traffic can only reach them through controlled internal routing or explicit security boundaries.

My private and public subnets cannot have the same CIDR block (or overlapping IP address range). Within a single Virtual Private Cloud (VPC), each subnet requires a unique set of IP addresses so that network traffic can be routed unambiguously to the correct destination without collisions.

![Image](https://nextwork.ai/overjoyed_magenta_proud_yak/uploads/4e96e29d-98b8-551e-ae93-da39c36e30fd_afe1fdbd)

## A dedicated route table

By default, my private subnet is associated with the main (default) route table automatically created for NextWork VPC. Unless an explicit subnet association is made with a custom route table, any new subnet inherits this main route table's rules.

I had to set up a new route table because our VPC's original route table (NextWork Public Route Table) has an active target route (0.0.0.0/0) pointing directly to an Internet Gateway. Associating the private subnet with that existing route table would make its resources publicly accessible; creating a dedicated route table guarantees that private subnet traffic is restricted strictly to local VPC targets (10.0.0.0/16).

My private subnet's dedicated route table only has a local target route (10.0.0.0/16) that allows communication between resources located within the VPC. It does not include an outbound target route (0.0.0.0/0) to an Internet Gateway, ensuring that all incoming and outgoing internet traffic is blocked at the routing layer.

![Image](https://nextwork.ai/overjoyed_magenta_proud_yak/uploads/4e96e29d-98b8-551e-ae93-da39c36e30fd_b4b904b5)

## A new network ACL

By default, my private subnet is associated with the VPC's main default Network ACL (NACL). When a new subnet is created without specifying a Network ACL, AWS automatically attaches this default NACL—which contains pre-configured rules allowing all inbound (0.0.0.0/0) and outbound (0.0.0.0/0) traffic—until you explicitly associate a custom NACL.

I set up a dedicated network ACL for my private subnet because having distinct, subnet-level stateless firewalls ensures independent access control for public vs. private environments. By explicitly isolating NextWork Private Subnet under its own Network ACL, I can configure strict custom packet-filtering rules to protect internal backend databases while keeping public subnet rules completely separate.

My new network ACL has two simple rules—Rule 100 for Inbound traffic and Rule 100 for Outbound traffic—which explicitly allow all IPv4 traffic (0.0.0.0/0) across all ports and protocols. Any traffic that is not evaluated by Rule 100 falls through to the asterisk (*) catch-all rule, which denies all remaining traffic by default.

![Image](https://nextwork.ai/overjoyed_magenta_proud_yak/uploads/4e96e29d-98b8-551e-ae93-da39c36e30fd_1ed2cb07)

---

*Built with [NextWork](https://nextwork.ai) - [View this project](https://nextwork.ai/projects/4e96e29d-98b8-551e-ae93-da39c36e30fd)*
