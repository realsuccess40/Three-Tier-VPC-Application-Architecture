# Three-Tier-VPC-Application-Architecture
This repository will build the entire VPC networking infrastructure supporting a classic three-tier web application in the us-east-1 region using Availability Zones A and B (us-east-1a and us-east-1b).


This exercise focuses purely on VPC design, high availability, security isolation, and connectivity validation — core DevOps skills for production-grade infrastructure.

Expected Architecture  -

- VPC CIDR: 10.0.0.0/16   
- Public Subnets (Web Tier + Bastion):  
  - Web-Public-A (10.0.1.0/24, us-east-1a)  
  - Web-Public-B (10.0.2.0/24, us-east-1b)  
- Private Subnets (App Tier):  
  - App-Private-A (10.0.11.0/24, us-east-1a)  
  - App-Private-B (10.0.12.0/24, us-east-1b)  
- Private Subnets (DB Tier):  
  - DB-Private-A (10.0.21.0/24, us-east-1a)  
  - DB-Private-B (10.0.22.0/24, us-east-1b)  

- Internet Gateway (1)
-  
- NAT Gateways (2 — one per AZ for true high availability)  
- Route Tables (1 public + 2 private)  
- Security Groups (tier-specific with strict ingress rules)  
- EC2 VMs (4 VMs): Bastion (public), Web (public), App (private), DB (private)  


# Step-by-Step Implementation Guide

Step 1: I Created the VPC    
Name: `Borderless-ThreeTier-VPC`  
IPv4 CIDR: `10.0.0.0/16`   


Step 2: I Created Subnets (6 total)  

- **Web-Public-A**: AZ us-east-1a, CIDR 10.0.1.0/24  
- **Web-Public-B**: AZ us-east-1b, CIDR 10.0.2.0/24  
- **App-Private-A**: AZ us-east-1a, CIDR 10.0.11.0/24  
- **App-Private-B**: AZ us-east-1b, CIDR 10.0.12.0/24  
- **DB-Private-A**: AZ us-east-1a, CIDR 10.0.21.0/24  
- **DB-Private-B**: AZ us-east-1b, CIDR 10.0.22.0/24  


Step 3:I Created and Attached Internet Gateway  
Name: ThreeTier-IGW


Step 4: I Created Elastic IP for NAT Gateways 


Step 5: I Created NAT Gateways    
**NAT-A**: Subnet = Web-Public-A, Elastic IP = first EIP, Name = `NAT-A`.  
**NAT-B**: Subnet = Web-Public-B, Elastic IP = second EIP, Name = `NAT-B`. 

Step 6: I Created Route Tables 
# Public Route Table

# Private Route Tables 
.

Step 7: Created Security Groups (Tier Isolation) 
 

- **Web-SG** (Web Tier + Bastion):  
  Inbound: SSH (22) from 0.0.0.0/0 , HTTP (80) from 0.0.0.0/0, ICMP (All) from 0.0.0.0/0.  

- **App-SG** (Application Tier):  
  Inbound: SSH (22) + ICMP from **Web-SG**, Custom TCP 8080 from **Web-SG**.  

- **DB-SG** (Database Tier):  
  Inbound: SSH (22) + ICMP from **App-SG**, Custom TCP 3306 from **App-SG**.  

Step 8: Created Key Pair  

Step 9: Launch Placeholder EC2 VMs

- **Bastion**: Subnet = Web-Public-A, Security Group = Web-SG (public IP auto).  
- **Web-VM**: Subnet = Web-Public-B, Security Group = Web-SG.  
- **App-VM**: Subnet = App-Private-A, Security Group = App-SG (no public IP).  
- **DB-VM**: Subnet = DB-Private-B, Security Group = DB-SG (no public IP).  

Step 10: Test Connectivity (The Most Important Part!)  

1. Internet Access 
   - SSH to Bastion and Web-VM (public IPs).  
   - Run: `curl ifconfig.me` and `ping 8.8.8.8`   

2. Private Subnet Outbound (NAT Test)  
   SSH to App-VM and DB-VM via Bastion (or jump through Web-VM) and repeat the curl/ping test.  

3 Tier-to-Tier Connectivity 
   - From Web-VM: `ping <App-VM-private-IP>` and `ping <DB-VM-private-IP>` (must succeed).  
   - From App-VM: `ping <DB-VM-private-IP>`.  
   - Test port reachability (no listener, so expect "Connection refused" — this proves SG allows traffic):  
       

4. Bastion Jump Test 
   From your laptop → SSH to Bastion → Then from Bastion:  
   `ssh -i /path/to/key ec2-user@<App-private-IP>` and same for DB-VM (works because of security group rules).

# Success Criteria  
- All VMs can reach the internet.  
- Web tier can communicate with App tier.  
- App tier can communicate with DB tier.  
- Private instances have no direct inbound internet access.  
- Architecture spans two AZs with proper routing and isolation
   - Try to SSH directly to App-VM or DB-VM private IPs from your laptop → Must fail (timeout).  
   - Confirm you cannot reach private instances from the internet.
