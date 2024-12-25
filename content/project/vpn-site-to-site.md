---
title: "VPN site-to-site setup."
description: "How setup a VPN connection on AWS."
dateString: December 2024
date: 2024-12-25T00:40:04-07:00
draft: false
tags: ["AWS","ECS","VPC"]
weight: 101
---
## Kiến trúc dự án:
![vpc](/images/vpn/VPN.png)
- Welcome mọi người đến với bài lab của tôi, lại là một bài hands-on liên quan tới networking.
- **Topic:** VPN site-to-site setup
- **Time to complete:** 30 minutes
- **Region:** Singapore
## Thông tin dự án:
- Hiện tại do mình không có môi trường on-premise nên mình sẽ mô phỏng một môi trường ảo trên cloud đóng vai trò là on-premise.
- Thì dự án của mình bao gồm 2 phần:
  - **Phần 1:** Khởi tạo 2 VPCs & launch EC2 instances theo kiến trúc.
  - **Phần 2:** Thiết lập VPN connection.
## Các bước thực hiện:
#### Create AWS-VPC
- **VPC settings:**
  - **Resources to create:** VPC and more
  - **Name:** AWS-VPC
  - **IPv4 CIDR block:** 10.0.0.0/16
   ![vpc](/images/vpn/aws-vpc/vpc1.png)
  - **Number of Availability Zones (AZs):** 2
  - **Number of public subnets:** 2
  - **Number of private subnets:** 2
   ![vpc](/images/vpn/aws-vpc/vpc2.png)
  - **NAT gateways:** In 1 AZ
  - **VPC endpoints:** None
   ![vpc](/images/vpn/aws-vpc/vpc3.png)
#### Create VPC-On-premise
- **VPC settings:**
  -  **Resources to create:** VPC and more
  -  **Name:** VPC-ON-PREMISE
  -  **IPv4 CIDR block:** 192.168.1.0/24
    ![vpc](/images/vpn/aws-vpc/vpc4.png)
  -  **Number of Availability Zones (AZs):** 1
  -  **Number of public subnets:** 1
  -  **Number of private subnets:** 1
    ![vpc](/images/vpn/aws-vpc/vpc5.png)
  -  **NAT gateways:** None
  -  **VPC endpoints:** None
    ![vpc](/images/vpn/aws-vpc/vpc6.png)
- Confirm 2 VPCs
#### Create Security group AWS-VPC
- Click "Create security group"
 ![sg](/images/vpn/aws-vpc/sg.png)
- **Security group name:** SecurityGroup-AWS-VPC
- **Description:** Allows traffic to connection VPN
- **VPC:** AWS-VPC-vpc
 ![sg](/images/vpn/aws-vpc/sg1.png)
- **Inbound rules:**
  - **Type:** All ICMP - IPv4
  - **Sources:** Anywhere IPv4
   ![sg](/images/vpn/aws-vpc/sg2.png)
 ![sg](/images/vpn/aws-vpc/sg3.png)
#### Create Security group VPC-On-premise
- Click "Create security group"
- **Security group name:** SecurityGroup-VPC-On-Premise
- **Description:** Allows traffic to connect VPN
- **VPC:** VPC-ON-PREMISE-vpc
 ![sg](/images/vpn/aws-vpc/sg4.png)
- **Inbound rules:**
  - **Type:** All ICMP - IPv4
  - **Sources:** Anywhere IPv4
  - **Type:** SSH
  - **Sources:** MyIP
   ![sg](/images/vpn/aws-vpc/sg5.png)
 ![sg](/images/vpn/aws-vpc/sg6.png)
#### Launch EC2 instance "OpenSwan"
- **Name and tags:** OpenSwan
 ![ec2](/images/vpn/aws-on-premise/ec2.png)
- **AMI:** Amazon Linux 2 AMI
 ![ec2](/images/vpn/aws-on-premise/ec2-1.png)
- **Instance type:** t2.micro
 ![ec2](/images/vpn/aws-on-premise/ec2-2.png)
- **Key pair:** Keypair-singapore
 ![ec2](/images/vpn/aws-on-premise/ec2-3.png)
- **Network settings:**
  - **VPC - required:** VPC-ON-PREMISE-vpc
  - **Subnet:** Public Subnet
  - **Auto-assign public IP:** Enable
  - **Security group:** SecurityGroup-VPC-On-premise
   ![ec2](/images/vpn/aws-on-premise/ec2-4.png)
#### Launch EC2 instance "Internal Server"
- **Name and tags:** Internal Server
 ![ec2](/images/vpn/aws-on-premise/internal-sv.png)
- **AMI:** Amazon Linux 2023
 ![ec2](/images/vpn/aws-on-premise/internal-sv1.png)
- **Instance type:** t2.micro
 ![ec2](/images/vpn/aws-on-premise/internal-sv2.png)
- **Key pair:** Keypair-singapore
 ![ec2](/images/vpn/aws-on-premise/internal-sv3.png)
- **Network settings:**
  - **VPC - required:** VPC-ON-PREMISE-vpc
  - **Subnet:** Private subnet
  - **Auto-assign public IP:** Disable
  - **Security group:** SecurityGroup-VPC-On-Premise
   ![ec2](/images/vpn/aws-on-premise/internal-sv4.png)
#### Launch EC2 instance "EC2-Public"
- **Name and tags:** EC2-Public
 ![ec2](/images/vpn/aws-vpc/ec2.png)
- **AMI:** Amazon Linux 2023
 ![ec2](/images/vpn/aws-vpc/ec2-1.png)
- **Instance type:** t2.micro
 ![ec2](/images/vpn/aws-vpc/ec2-2.png)
- **Key pair:** Keypair-singapore
 ![ec2](/images/vpn/aws-vpc/ec2-3.png)
- **Network settings:**
  - **VPC - required:** AWS-VPC-vpc
  - **Subnet:** Public subnet
  - **Auto-assign public IP:** Enable
  - **Security group:** SecurityGroup-AWS-VPC
   ![ec2](/images/vpn/aws-vpc/ec2-4.png)
#### Tiếp theo Disable-destination của instance "OpenSwan"
 ![ec2](/images/vpn/aws-on-premise/Disable-Destination.png)
 ![ec2](/images/vpn/aws-on-premise/Disable-Destination1.png)
#### Thiết lập Customer gateways.
- Click "Create customer gateway"
 ![cgw](/images/vpn/aws-vpc/cgw.png)
- **Name tag:** CGW-ON-PREMISE
- **BGP ASN:** 65000
- **IP address:** 52.221.239.121 (Copy IP public "OpenSwan")
 ![cgw](/images/vpn/aws-vpc/cgw1.png)
 ![cgw](/images/vpn/aws-vpc/cgw2.png)
- Confirm customer gateway
 ![cgw](/images/vpn/aws-vpc/cgw3.png)
#### Thiết lập Virtual private gateways.
- Click "Create virtual private gateway"
 ![vgw](/images/vpn/aws-vpc/vgw.png)
- **Name tag:** VGW-AWS-VPC
 ![vgw](/images/vpn/aws-vpc/vgw1.png)
- **Attach to VPC**
- **Availability VPCs:** AWS-VPC-vpc
 ![vgw](/images/vpn/aws-vpc/vgw2.png)
- => Attach to VPC
 ![vgw](/images/vpn/aws-vpc/vgw3.png)
- Confirm Virtual private gateway
 ![vgw](/images/vpn/aws-vpc/vgw4.png)
- Edit Route table "VPC-AWS-rtb-public"
  ![rtb](/images/vpn/aws-vpc/rtb.png)
  ![rtb](/images/vpn/aws-vpc/rtb1.png)
#### Thiết lập VPN connection.
- Click "Create VPN connection"
 ![vpn](/images/vpn/aws-vpc/vpn.png)
- **Name tag:** VPN-ON-PREMISE-TO-AWS-VPC
- **Virtual private gateway:** Specify VGW
- **Customer gateway ID:** Specify CGW
- **Routing options:** Static
- **Static IP prefixes:**
  - 192.168.1.0/24
  - 10.0.0.0/16
 ![vpn](/images/vpn/aws-vpc/vpn1.png)
- **Local IPv4 network CIDR - optional:** 192.168.1.0/24
- **Remote IPv4 network CIDR - optional:** 10.0.0.0/16
 ![vpn](/images/vpn/aws-vpc/vpn2.png)
 ![vpn](/images/vpn/aws-vpc/vpn3.png)
- Confirm VPN connection
 ![vpn](/images/vpn/aws-vpc/vpn4.png)
- Tiếp theo download configuration file
 ![vpn](/images/vpn/aws-vpc/file.png)
- Open file bằng notepad
 ![vpn](/images/vpn/aws-vpc/file1.png)
- Thực hiện các bước bên trong file
 ![vpn](/images/vpn/aws-vpc/file2.png)
- SSH vào ec2 instance "OpenSwan" để tiến hành config vpn
 ![ssh](/images/vpn/aws-vpc/ssh.png)
- **cli:** nano /etc/sysctl.conf
 ![ssh](/images/vpn/aws-vpc/ssh1.png)
- **cli:** sysctl -p
 ![ssh](/images/vpn/aws-vpc/ssh2.png)
- **cli:** yum install openswan -y
 ![ssh](/images/vpn/aws-vpc/ssh3.png)
- **cli:** nano /etc/ipsec.d/aws.conf
 ![ssh](/images/vpn/aws-vpc/ssh4.png)
- **cli:** nano /etc/ipsec.d/aws.secrets
 ![ssh](/images/vpn/aws-vpc/ssh5.png)
- **cli:** systemctl start ipsec
- **cli:** systemctl status ipsec
 ![ssh](/images/vpn/aws-vpc/ssh6.png)
 ![ssh](/images/vpn/aws-vpc/vpn5.png)
- Ping ip private instance "EC2-Public"
 ![ssh](/images/vpn/aws-vpc/ssh7.png)
- Tiếp theo ssh vào instance "Internal Server" để ping ip private instance "EC2-public"
- Edit Route table private
 ![rtb](/images/vpn/aws-on-premise/rtb.png)
 ![rtb](/images/vpn/aws-on-premise/rtb1.png)
 ![rtb](/images/vpn/aws-on-premise/rtb2.png)
- Ping ip private instance "EC2-public"
 ![rtb](/images/vpn/aws-on-premise/ping-success.png)
#### Clean up resources
1.  Terminate all instances
2.  Delete the VPN connection
3.  Detach VPC from Virtual private gateways => Delete Virtual private gateways
4.  Delete Customer gateways
5.  Delete Nat gateway
6.  Release Elastic IP
7.  Delete 2 VPCs

=> Recheck lại một lần nuk, để tránh phát sinh chi phí.
- Vừa rồi là một bài hands-on cũng hơi dài và cũng có nhiều công đoạn để mà setup vpn connection, hy vọng rằng bài lab này có thể giúp các bạn setup thành công.
- Thanks all mọi người.
