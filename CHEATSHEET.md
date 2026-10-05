# Cheat Sheet: Creating a Private Subnet

Lesson notes from the NextWork project, for quick review.

## Contents

1. [The big picture](#1-the-big-picture)
2. [Picking a CIDR block that doesn't overlap](#2-picking-a-cidr-block-that-doesnt-overlap)
3. [A dedicated route table](#3-a-dedicated-route-table)
4. [A dedicated network ACL](#4-a-dedicated-network-acl)
5. [What about security groups?](#5-what-about-security-groups)
6. [Defaults to remember](#6-defaults-to-remember)
7. [Self-quiz](#7-self-quiz)

---

## 1. The big picture

What I built, on top of the VPC from the previous project:

```
NextWork VPC (10.0.0.0/16)
├── Public 1 subnet      10.0.0.0/24 → NextWork route table (has 0.0.0.0/0 → Internet Gateway)
└── Private subnet       10.0.1.0/24 → Private route table (local route only)
                                       + Private network ACL (denies everything)
```

| AWS piece | Analogy |
|---|---|
| VPC | A gated neighborhood |
| Subnet | A street inside the neighborhood |
| Internet Gateway | The main gate out to the highway |
| Route table | The road signs on a street, telling traffic where it can go |
| Network ACL | A guard at the entrance of a street, checking everyone coming in **and** going out |
| Security group | The lock on each individual house's door |

A **private subnet** is a street with no road sign pointing to the main gate, plus a strict guard at its entrance.

## 2. Picking a CIDR block that doesn't overlap

| Subnet | CIDR block | IP range |
|---|---|---|
| Public 1 | `10.0.0.0/24` | `10.0.0.0` – `10.0.0.255` |
| Private | `10.0.1.0/24` | `10.0.1.0` – `10.0.1.255` |

- The only difference is the third number (`0` vs `1`), but that keeps the two ranges completely separate.
- Subnets in the same VPC **can't overlap**. Each IP address has to belong to one subnet so AWS knows where to route traffic.
- `/24` means the first 24 bits are fixed, which leaves 256 addresses. AWS reserves 5 in every subnet, so 251 are usable.
- Both ranges must sit inside the VPC's range (`10.0.0.0/16` = `10.0.0.0` – `10.0.255.255`).

**Analogy:** two streets can't share the same house numbers, or the mail carrier wouldn't know which house to deliver to.

## 3. A dedicated route table

**Why revisit route tables?**
- In the VPC project I renamed the VPC's **default (main) route table** to *NextWork route table*, and added a route to the Internet Gateway.
- Any subnet that isn't explicitly associated with another route table **automatically uses the main one**.
- So the new private subnet started out using the public route table, which made it **public** until I changed it.

**The fix:** create a new route table with only the local route, and associate the private subnet with it.

| Destination | Target | Meaning |
|---|---|---|
| `10.0.0.0/16` | local | Traffic can move between resources inside the VPC |
| ~~`0.0.0.0/0`~~ | ~~Internet Gateway~~ | Not added, so there's no path to the internet |

**What makes a subnet public or private?** Only its route table. A subnet is public if its route table has a route to an Internet Gateway, and private if it doesn't.

## 4. A dedicated network ACL

**Why revisit network ACLs?** Same story as route tables:
- Every VPC has a **default network ACL**, and it **allows all traffic** in and out.
- Subnets without an explicit association use the default NACL, so the private subnet was wide open.

**"I already removed the internet route. Isn't that enough?"**
No. Removing the route stops *direct* internet access, but relying on one layer is weak security. If the public subnet gets compromised, an attacker could reach the private subnet through the VPC's local route, and a permissive NACL wouldn't stop them. That's **defense in depth**: several layers, so one failure doesn't expose everything.

**Default NACL vs custom NACL**

| | Default NACL | Custom NACL (new) |
|---|---|---|
| Created | Automatically with the VPC | By you |
| Inbound rules | Rule 100: allow all, then `*` deny | Only `*` deny all |
| Outbound rules | Rule 100: allow all, then `*` deny | Only `*` deny all |
| Result | Everything allowed | **Everything blocked** |

- Rules are checked from the **lowest number up**. The first match wins.
- The `*` rule is the catch-all at the end. It always denies and can't be removed.
- For now, the private NACL denies everything. Allow rules get added later in the series, once it's clear which traffic needs in.

## 5. What about security groups?

- Security groups are set at the **resource level** (e.g. an EC2 instance), not the subnet level.
- No resources are in the private subnet yet, so there's nothing to attach one to.

| | Network ACL | Security group |
|---|---|---|
| Level | Subnet | Resource (e.g. EC2 instance) |
| Rules | Allow **and** deny | Allow only |
| State | **Stateless**: return traffic must be allowed separately | **Stateful**: replies are allowed automatically |
| Analogy | Guard at the street entrance | Lock on each house's door |

## 6. Defaults to remember

| When you create... | It automatically... |
|---|---|
| A VPC | Gets a main route table and a default NACL (allows all) |
| A subnet | Uses the main route table and the default NACL until you associate others |
| A custom route table | Has only the `local` route |
| A custom NACL | Denies all inbound and outbound traffic |

**Takeaway:** a new subnet copies whatever the VPC's defaults are. If the defaults are public or open, so is the new subnet, until you change the associations.

## 7. Self-quiz

<details>
<summary>1. Public 1 is 10.0.0.0/24. Why does 10.0.1.0/24 not overlap with it?</summary>

10.0.0.0/24 covers 10.0.0.0 – 10.0.0.255, and 10.0.1.0/24 covers 10.0.1.0 – 10.0.1.255. The ranges share no addresses.
</details>

<details>
<summary>2. Why can't two subnets in the same VPC have overlapping CIDR blocks?</summary>

Each IP address must belong to exactly one subnet, so traffic can be routed to the right place.
</details>

<details>
<summary>3. Right after I created the private subnet, which route table was it using, and why was that a problem?</summary>

The VPC's main route table (NextWork route table). It has a route to the Internet Gateway, so the "private" subnet was actually public.
</details>

<details>
<summary>4. What single thing decides whether a subnet is public or private?</summary>

Whether its route table has a route (usually 0.0.0.0/0) to an Internet Gateway.
</details>

<details>
<summary>5. What routes does the private route table have?</summary>

Only the local route (10.0.0.0/16 → local), so resources can talk within the VPC but not to the internet.
</details>

<details>
<summary>6. The internet route is already gone. Why also create a custom NACL?</summary>

Defense in depth. The default NACL allows all traffic, so if the public subnet were compromised, an attacker could reach the private subnet through the VPC. A strict NACL adds a second layer.
</details>

<details>
<summary>7. What rules does a brand-new custom NACL have?</summary>

Only the `*` rule on inbound and outbound, which denies all traffic.
</details>

<details>
<summary>8. How is that different from the default NACL?</summary>

The default NACL also has Rule 100 allowing all traffic in both directions, so everything is allowed.
</details>

<details>
<summary>9. NACL rules 100, 200 and * all match a packet. Which one applies?</summary>

Rule 100. Rules are evaluated from the lowest number up, and the first match wins.
</details>

<details>
<summary>10. Why didn't we create a security group in this project?</summary>

Security groups attach to resources like EC2 instances, and there are no resources in the private subnet yet.
</details>

<details>
<summary>11. NACLs are stateless and security groups are stateful. What does that mean?</summary>

A stateless NACL checks traffic in each direction separately, so replies need their own allow rule. A stateful security group remembers allowed requests and lets the replies back in automatically.
</details>
