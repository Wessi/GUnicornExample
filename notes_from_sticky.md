https://www.ibexautoauction.com/en

/etc/ssh/sshd_config:
````
PermitRootLogin yes
PasswordAuthentication yes
````
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

**Calculating "Real" Speed**

**Keep in mind that 2 Mbps (Megabits) is not 2 MB/s (Megabytes)**
* 1 Mbps = ~125 KB/s download speed.
* 8 Mbps = ~1 MB/s download speed.

If you buy 1–10 Mbps: Your Outbound (sending data to users) will be exactly what you bought (e.g., 2 Mbps). However, your Inbound (uploading your code or updates to the server) is automatically boosted to 10 Mbps for free. If you buy > 10 Mbps: Your Inbound and Outbound speeds will be equal to the amount you purchased.

The Math of /8 or /16 (w.x.y.z)\
Locked Bits: 8\
Free Bits: 32 - 8 = 24 bits\
Calculation: 2^24 = 16,777,216 addresses

Telecloud asks for a /16 for VPC as Telecloud is built on Huawei Cloud Stack infrastructure\
192.168...-private range\
Requirement: The subnet CIDR must be a subset of the VPC CIDR. If your VPC is 192.168.0.0/16, your subnet will be something like 192.168.1.0/24 (but what worked for me is same IP for both)

**Can I put data on the System Disk?**\
Yes. When you spin up an ECS, the system disk will have free space available (e.g., if you choose an 80GB system disk, the OS and system files might only take up 20GB, leaving 60GB of open space). You can absolutely store personal files or databases there.\
**Why You Should Use a Data Disk**\
Independent Recovery: If your OS is corrupted, you can wipe and reformat the system disk without losing any files on your data disk.\

**spin up an ECS** = launching/initializing new virtual server from scratch. So **spin up** is just when you click `Create/Launch`, the cloud software allocates CPU, memory, and storage, then installs the operating system, and boots it into an active state.

ECS - Elastic Cloud Server\
VPC - Virtual Private Cluod\
CFW - Cloud edge Firewall\
NACL - Network Access Control List\
SG - Security Group


root: P@ss4Telecloud\
wessi: Wessi

#### Bandwidth split example
5TB=5Tx8b=40Tb[as there are exactly 8 bits(b) in 1 Byte (B)]=40x1,000,000Mb=40,000,000Mb

To find the speed per second, we must calculate exactly how many seconds are in a standard 30-day month: 30days = 30x1day=30x24hrs=30x24x60mins=30x24x60x60secs=2,592,000secs

Speed(Mb/s)=40,000,000Mb/2,592,000s=15.43Mb/s=15.43Mbps

for 14 regions it is 15.43Mbps/14=1.1Mbps

**https://httparchive.org/reports/page-weight**
3000KB=3MB = 3x8Mb = 24 Mb of total data required just to display one page.\
So, if my internet bandwidth is 1Mbps, then it will take 24Mb devided by 1Mb/s = 24Mbx(s/1Mb) = 24s.\
if 2 users access at the same time it becomes 2x24s=48s. so too slow and need to increase **bandwidth**.

