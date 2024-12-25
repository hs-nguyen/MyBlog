---
title: "Setup a VPC Peering connections between two VPCs across regions on AWS."
description: "Setup a VPC Peering connections between two VPCs across regions on AWS."
dateString: December 2024
date: 2024-12-06T00:40:04-07:00
draft: false
tags: ["AWS","VPC"]
weight: 101
---
## Kiến trúc dự án
![vpc](/images/vpc-peering/VPC-peering.png)
-   Welcome mọi người đến project của tôi. Tiếp tục là một buổi hands-on để nâng cao kỹ năng trên AWS.
-   **Topic:** Setup a VPC Peering connections between two VPCs across regions on AWS.
-   **Time to complete:** 15 minutes
-   **Regions:** Singapore & Virginia
## Giới thiệu một chút:
#### VPC Peering là gì?
-   VPC Peering là một tính năng của VPC đóng vai trò xây dựng luồng kết nối giữa các VPC với nhau trong cùng region, khác region or khác account.
-   Vd:  Bạn đang có VPC A & VPC B trong cùng region với dải CIDR khác nhau, vậy làm thế nào chúng có thể giao tiếp với nhau?
-   VPC Peering sẽ giúp chúng ta giải quyết vấn đề này!
-   Hiểu như là VPC Peering là cây cầu bắt ngang 2 vị trí, từ điểm A có thể đi qa điểm B và ngược lại.
![vpc](/images/vpc-peering/VPC-peering-1.png)
#### Ưu điểm:
-   Xây dựng kết nối riêng giữa các VPCs.
-   Dễ dàng thiết lập và quản lý.
-   Độ trễ thấp.
-   Bảo mật.
-   Tiết kiệm chi phí.
#### Nhược điểm:
-   **Khả năng mở rộng:** Nếu bạn chỉ có từ 2 -> 3 VPCs thì việc thiết lập VPC Peering không thành vấn đề, nhưng trong tương lai nếu bạn có nhiều hơn 3 VPCs thì VPC peering không phải là giải pháp tối ưu. Thay vào đó chúng ta phải sử dụng Transit Gateway.
-   Thì trong bài này mình sẽ không đề cập về Transit Gateway.
-   **Tích hợp:** VPC Peering không hỗ trợ kết nối tới mạng, nếu bạn có những giải pháp kết nối bổ sung như VPN or AWS Direct Connect.
### Các bước thực hiện:
1.  Create 2 VPCs trên 2 regions Singapore & Virginia.
2.  Create VPC peering connection tại region Singapore.
3.  Update Route table cả 2 VPCs trên 2 regions.
4.  Create 2 Security group for 2 VPCs trên 2 regions.
5.  Launch EC2 instances trên 2 regions.
6.  Ping connection.
7.  Clean up resources.
## Let's get started
### Region: Singapore
#### VPC Setting (VPC, Subnets, Route table, IGW)
-   Click Create VPC
    -   **Resources to create:** VPC and more
    -   **Name:** vpc-singapore
    -   **IPv4 CIDR block:** 10.0.0.0/16
       ![vpc](/images/vpc-peering/vpc1.png)
    -   **Number of Availability Zones (AZs):** 1
    -   **Number of public subnets:** 1
    -   **Number of private subnets:** 1
        ![vpc](/images/vpc-peering/vpc2.png)
    -   **NAT gateways ($):** None
    -   **VPC endpoints:** None
       ![vpc](/images/vpc-peering/vpc3.png)
- Click Create VPC (hoàn tất việc tạo)
### Region: Virginia
#### VPC Setting (VPC, Subnets, Route table, IGW)
-   Click Create VPC
    -   **Resources to create:** VPC and more
    -   **Name:** vpc-virginia
    -   **IPv4 CIDR block:** 10.1.0.0/16
       ![vpc](/images/vpc-peering/vpc-vir-1.png)
    -   **Number of Availability Zones (AZs):** 1
    -   **Number of public subnets:** 1
    -   **Number of private subnets:** 1
       ![vpc](/images/vpc-peering/vpc-vir-2.png)
    -   **NAT gateways ($):** None
    -   **VPC endpoints:** None
       ![vpc](/images/vpc-peering/vpc-vir-3.png)
- Click Create VPC (hoàn tất việc tạo)
- Confirm VPC cả 2 regions
![vpc](/images/vpc-peering/confirm-vpc-sing.png)
![vpc](/images/vpc-peering/confirm-vpc-virginia.png)
### Region: Singapore
-   Click "Create Peering connection"
   ![vpc](/images/vpc-peering/vpc5.png)
    -   **Name:** Demo-VPC-Peering
    -   **Select a local VPC to peer with:**
        -   **VPC ID (Requester):** Specify VPC Singapore
    -   **Select another VPC to peer with:**
        -   **Account:** My account
        -   **Region:** Another Region => Specify VPC Virginia
        -   **VPC ID:** Copy VPC ID Region Virginia
          ![vpc](/images/vpc-peering/vpc6.png)
          ![vpc](/images/vpc-peering/vpc7.png)
          ![vpc](/images/vpc-peering/vpc8.png)
### Region: Virginia
-  Tại **Region Virginia** chúng ta cần phải **Accept request** từ VPC Peering Singapore.
![vpc](/images/vpc-peering/vpc9.png)
-  Confirm VPC peering ở cả 2 regions.
![vpc](/images/vpc-peering/vpc10.png)
![vpc](/images/vpc-peering/vpc11.png)
### Update Route table cả 2 VPCs trên 2 regions.
### Region: Singapore
-   Click Route tables => Click **vpc-singapore-rtb-public**
-   Routes => Edit routes
   ![vpc](/images/vpc-peering/rtb-sing.png)
-   **Destination:** 10.1.0.0/16 (CIDR block VPC Virginia)
-   **Target:** Peering Connection
   ![vpc](/images/vpc-peering/rtb-sing1.png)
-   Confirm Route table
   ![vpc](/images/vpc-peering/rtb-sing2.png)
### Region: Virginia
-   Click Route tables => Click **vpc-virginia-rtb-public**
-   Routes => Edit routes
   ![vpc](/images/vpc-peering/rtb-virgi.png)
-   **Destination:** 10.0.0.0/16 (CIDR block VPC Singapore)
-   **Target:** Peering Connection
   ![vpc](/images/vpc-peering/rtb-virgi-1.png)
-   Confirm Route table
   ![vpc](/images/vpc-peering/rtb-virgi-2.png)
### Create 2 Security group for 2 VPCs trên 2 regions.
### Region: Singapore
-   **Security group name:** SecurityGroup-VPC-Sing
-   **Description:** Allows connection between two VPC
-   **VPC:** Specify VPC Singapore
-   **Inbound rules:**
    -   **Type:** SSH
    -   **Port:** 22
    -   **Source:** Anywhere IPv4
    -   **Type:** ALL ICMP - IPv4
    -   **Port:** All
    -   **Source:** 10.1.0.0/16 (CIDR block VPC Virginia)
       ![security](/images/vpc-peering/sg-vpc-sing.png)
### Region: Virginia
-   **Security group name:** SecurityGroup-VPC-Virginia
-   **Description:** Allows connection between two VPC
-   **VPC:** Specify VPC Virginia
-   **Inbound rules:**
    -   **Type:** SSH
    -   **Port:** 22
    -   **Source:** Anywhere IPv4
    -   **Type:** ALL ICMP - IPv4
    -   **Port:** All
    -   **Source:** 10.0.0.0/16 (CIDR block VPC Singapore)
       ![security](/images/vpc-peering/sg-vpc-virginia.png)
- Confirm Security Group ở cả 2 regions.
 ![security](/images/vpc-peering/confirm-sg-sing.png)
 ![security](/images/vpc-peering/confirm-sg-virginia.png)
### Launch EC2 instances trên 2 regions.
### Region: Singapore
-   **Name and tags:** EC2-Singapore
   ![ec2](/images/vpc-peering/ec2.png)
-   **AMI:** Amazon Linux 2023
   ![ec2](/images/vpc-peering/ec2-1.png)
-   **Instance type:** t2.micro
   ![ec2](/images/vpc-peering/ec2-2.png)
-   **Keypair:** Keypair-singapore
   ![ec2](/images/vpc-peering/ec2-3.png)
-   **Network Setting:**
    -   **VPC - required:** vpc-singapore-vpc
    -   **Subnet:** Public Subnet
    -   **Auto-assign public IP:** Enable
    -   **Security Group:** SecurityGroup-VPC-Sing
       ![ec2](/images/vpc-peering/ec2-4.png)
- Click Launch instance
### Region: Virginia
-   **Name and tags:** EC2-Virginia
   ![ec2](/images/vpc-peering/ec2-virginia.png)
-   **AMI:** Amazon Linux 2023
   ![ec2](/images/vpc-peering/ec2-virginia-1.png)
-   **Instance type:** t2.micro
   ![ec2](/images/vpc-peering/ec2-virginia-2.png)
-   **Keypair:** admin.virginia
   ![ec2](/images/vpc-peering/ec2-virginia-3.png)
-   **Network Setting:**
    -   **VPC - required:** vpc-virginia-vpc
    -   **Subnet:** Public Subnet
    -   **Auto-assign public IP:** Enable
    -   **Security Group:** SecurityGroup-VPC-Virginia
       ![ec2](/images/vpc-peering/ec2-virginia-4.png)
- Click Launch instance.
- Confirm EC2 ở cả 2 regions.
- ![ec2](/images/vpc-peering/confirm-ec2-singapore.png)
- ![ec2](/images/vpc-peering/confirm-ec2-virginia.png)
### Ping connection.
-   Instance connect to EC2-Singapore (thực hiện lệnh ping IP private EC2-Virginia)
   ![ec2](/images/vpc-peering/ssh-ec2-singapore.png)
-   Instance connect to EC2-Virginia (thực hiện lệnh ping IP private EC2-Singapore)
   ![ec2](/images/vpc-peering/ssh-ec2-virginia.png)
### Clean up resources.
-   Terminate 2 EC2 instance on 2 regions
-   Delete Peering connections
-   Delete 2 VPCs on 2 regions
-   OK, cuối cùng mình đã làm xong project về VPC Peering, chỉ mất khoảng 15 phút cho bài lab lần này khá nhanh. Ngoài VPC peering ra các bạn có thể tìm hiểu và đọc thêm về Transit Gateway nha.
-   Chúc các bạn thành công.
