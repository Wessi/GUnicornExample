https://www.ibexautoauction.com/en
/etc/ssh/sshd_config:
PermitRootLogin yes
PasswordAuthentication yes

#### ECS:

```
Billing Mode: Prepaid
Region: ET-CLOUD-AA1
Resource Pool: OpenStack_et-global-1
Availability Zones: KVM_AZ
Specifications: telecloudECS1|2
Type: General Purpose
vCPUs: 1
Memory(GB): 2
Image: Linux
System Disk: KVM_Volume_SAS | 40 GB > Upgraded to SSD
Data Disk: KVM_Volume_SAS | 0GB
```
#### EIP:
```
Billing Mode: Prepaid
Region: ET-CLOUD-AA1
Resource Pool: OpenStack_et-global-1
Bandwidth: 2 Mbit/s
```	
EIP/Public IP: 196.189.51.93

P@ss4Master

Calculating "Real" Speed

Keep in mind that 2 Mbps (Megabits) is not 2 MB/s (Megabytes).
1 Mbps = ~125 KB/s download speed.
8 Mbps = ~1 MB/s download speed.

If you buy 1–10 Mbps: Your Outbound (sending data to users) will be exactly what you bought (e.g., 2 Mbps). However, your Inbound (uploading your code or updates to the server) is automatically boosted to 10 Mbps for free. If you buy > 10 Mbps: Your Inbound and Outbound speeds will be equal to the amount you purchased.

The Math of /8 or /16 (w.x.y.z)
Locked Bits: 8
Free Bits: 32 - 8 = 24 bits
Calculation: 2^24 = 16,777,216 addresses

Telecloud asks for a /16 for VPC as Telecloud is built on Huawei Cloud Stack infrastructure
192.168...-private range
Requirement: The subnet CIDR must be a subset of the VPC CIDR. If your VPC is 192.168.0.0/16, your subnet will be something like 192.168.1.0/24 (but what worked for me is same IP for both)

ECS - Elastic Cloud Server
VPC - Virtual Private Cluod
CFW - Cloud edge Firewall
NACL - Network Access Control List
SG - Security Group


root: P@ss4Telecloud
wessi: Wessi
