**AWS Asymmetric VPC Architecture – Manual Setup Guide (eu-north-1)**

**Project:** asymmetric-vpc-build
**Environment:** production
**Owner:** network-team
**CostCenter:** AWS-Networking

---

# 🏗️ **Architecture Overview**

This project manually builds a **production-grade AWS VPC** in the **Europe (Stockholm)** region using **variable-size subnets**, **public and private routing**, **Internet Gateway**, **multiple NAT Gateways**, and **dedicated route tables per subnet**.

The goal is to simulate a real enterprise network with different IP capacity requirements per subnet.

---

# 📡 **Final Architecture Diagram
<img width="1920" height="1080" alt="VPC-ResourceMap" src="https://github.com/user-attachments/assets/4fbd71e9-e719-48a3-8bc4-9e182ee71427" />

---

# 📂 **TABLE OF CONTENTS**

1. Create VPC
2. Create Six Subnets (Variable Sizes)
3. Enable Auto-Assign Public IPs for Public Subnets
4. Create Internet Gateway
5. Create NAT Gateways (Highly Available)
6. Create Route Tables
7. Associate Subnets with Route Tables
8. Tagging Standards
9. Validation

---

# ------------------------------------

# 🧱 **1. Create VPC**

# ------------------------------------

### **Steps**

1. AWS Console → **VPC → Create VPC**
2. Choose: **VPC Only**
3. Fill details:

   * **Name:** `asymmetric-vpc`
   * **IPv4 CIDR:** `10.0.0.0/16`
   * **Enable DNS Resolution:** ✔ Yes
   * **Enable DNS Hostnames:** ✔ Yes
4. Add standard tags
   `Environment = production`
   `Owner       = network-team`
   `Project     = asymmetric-vpc-build`
   `CostCenter  = AWS-Networking`

### **Screenshot Placeholder**

<img width="1920" height="1080" alt="VPC-DETAILS" src="https://github.com/user-attachments/assets/96a34412-af11-49be-b3e1-ac1bf0ff8dc3" />
<img width="1920" height="1080" alt="VPC-CIDR" src="https://github.com/user-attachments/assets/7f3b3b88-ba97-4d89-aca6-7d46ab1dd577" />
<img width="1920" height="1080" alt="VPC-TAGS" src="https://github.com/user-attachments/assets/ceeb8319-c835-41a3-b53f-a92d6b3ba155" />


---

# ------------------------------------

# 🗺️ **2. Create Six Subnets (Variable Sizes)**

# ------------------------------------

### **Subnet Design Table**

| Subnet Name      | CIDR         | Capacity | AZ          |
| ---------------- | ------------ | -------- | ----------- |
| public-subnet-a  | 10.0.0.0/24  | ~256     | eu-north-1a |
| public-subnet-b  | 10.0.16.0/20 | ~4096    | eu-north-1b |
| public-subnet-c  | 10.0.32.0/19 | ~8192    | eu-north-1c |
| private-subnet-a | 10.0.64.0/22 | ~1024    | eu-north-1a |
| private-subnet-b | 10.0.68.0/23 | ~512     | eu-north-1b |
| private-subnet-c | 10.0.96.0/19 | ~8192    | eu-north-1c |

### **Steps**

1. Go to **VPC → Subnets → Create Subnet**
2. Select **asymmetric-vpc**
3. Provide name, AZ, CIDR
4. Add tags:

   * Public → `Tier = public`
   * Private → `Tier = private`
   `Environment = production`
   `Owner       = network-team`
   `Project     = asymmetric-vpc-build`
   `CostCenter  = AWS-Networking`

### **Screenshot Placeholder**

<img width="1920" height="1080" alt="Pub-Subnet-Details" src="https://github.com/user-attachments/assets/86222e25-6590-436b-b26e-9b253e206be8" />
<img width="1920" height="1080" alt="Pub-Subnet-Routes" src="https://github.com/user-attachments/assets/414d32c1-9fb6-4b79-b9f5-4decb584adff" />
<img width="1920" height="1080" alt="Pub-Subnet-Tags" src="https://github.com/user-attachments/assets/ceaec11a-c553-45b6-8c8e-dfd55c52be84" />
<img width="1920" height="1080" alt="Pvt-Subnet-Details" src="https://github.com/user-attachments/assets/c0a4267e-419b-4f52-8bda-6cd8a348cfa8" />
<img width="1920" height="1080" alt="Pvt-Subnet-Routes" src="https://github.com/user-attachments/assets/6e0a3426-74e1-4210-88b4-7fdf0df66471" />
<img width="1920" height="1080" alt="Pvt-Subnet-Tags" src="https://github.com/user-attachments/assets/247e1a48-94fc-4c0f-9711-4ce2de49e03e" />


---

# ------------------------------------

# 🌐 **3. Enable Auto-Assign Public IPs for Public Subnets**

# ------------------------------------

### **Steps**

1. Select subnet: `public-subnet-a`
2. Click **Actions → Edit Subnet Settings**
3. Enable:

   ```
   [x] Auto-assign public IPv4 address
   ```
4. Repeat for:

   * public-subnet-b
   * public-subnet-c

### **Screenshot Placeholder**

<img width="1920" height="1080" alt="AutoAssign-PublicIP" src="https://github.com/user-attachments/assets/bb0977af-0831-4d91-8752-21733ca751b9" />


---

# ------------------------------------

# 🚪 **4. Create Internet Gateway**

# ------------------------------------

### **Steps**

1. VPC → **Internet Gateways → Create**
2. Name: `asym-igw`
3. Attach to **asymmetric-vpc**
4. Add standard tags.

### **Screenshot Placeholder**

<img width="1920" height="1080" alt="IGW-Details" src="https://github.com/user-attachments/assets/8ef68d68-3b8f-4355-a3ab-39cb5d9a4bff" />
<img width="1920" height="1080" alt="IGW-Tags" src="https://github.com/user-attachments/assets/d1acd13e-d1e3-4447-b9fa-8b1c764527f6" />


---

# ------------------------------------

# 🚀 **5. Create NAT Gateways (Highly Available)**

# ------------------------------------

### **Why Public Subnet?**

NAT Gateway **must** be placed in **public subnets** because it requires:

* Elastic IP
* Internet Gateway Route

### **Create One NAT Per AZ**

| NAT Gateway | Subnet          |
| ----------- | --------------- |
| nat-gw-a    | public-subnet-a |
| nat-gw-b    | public-subnet-b |
| nat-gw-c    | public-subnet-c |

### **Steps**

1. VPC → NAT Gateways → **Create NAT Gateway**
2. Choose:

   * Subnet = corresponding public subnet
   * Connectivity Type = **Public**
   * Elastic IP = **Allocate New**
3. Add tags.

### **Screenshot Placeholder**

<img width="1920" height="1080" alt="NAT-Details" src="https://github.com/user-attachments/assets/e79202ee-f8c0-4a09-b190-dd44a0edc500" />
<img width="1920" height="1080" alt="NAT-Tags" src="https://github.com/user-attachments/assets/7cf700ec-1108-4590-99f9-628df9dd3097" />


---

# ------------------------------------

# 🛣️ **6. Create Route Tables**

# ------------------------------------

## **Public Route Tables**

| Route Table | Subnet          |
| ----------- | --------------- |
| rt-public-a | public-subnet-a |
| rt-public-b | public-subnet-b |
| rt-public-c | public-subnet-c |

### **Steps**

1. VPC → Route Tables → Create Route Table
2. Add route:

   ```
   Destination: 0.0.0.0/0
   Target: Internet Gateway
   ```

### **Screenshot Placeholder**

<img width="1920" height="1080" alt="rt-public-routes" src="https://github.com/user-attachments/assets/5d58336d-864c-48c4-881e-f0abd6926a53" />
<img width="1920" height="1080" alt="rt-tags" src="https://github.com/user-attachments/assets/9a5892ed-1168-4c49-86fb-91486e1899d7" />



---

## **Private Route Tables**

| Route Table  | NAT Gateway |
| ------------ | ----------- |
| rt-private-a | nat-gw-a    |
| rt-private-b | nat-gw-b    |
| rt-private-c | nat-gw-c    |

### **Steps**

1. Create a route table
2. Add route:

   ```
   Destination: 0.0.0.0/0
   Target: NAT Gateway (same AZ)
   ```

### **Screenshot Placeholder**

<img width="1920" height="1080" alt="rt-private-routes" src="https://github.com/user-attachments/assets/09e574bc-02e7-4b0f-9b3c-35b15adb6820" />
<img width="1920" height="1080" alt="rt-pvt-Tags" src="https://github.com/user-attachments/assets/9d22d8b4-acdc-4e4a-af1f-45792929dd8b" />

---

# ------------------------------------

# 🔗 **7. Associate Subnets with Route Tables**

# ------------------------------------

### **Public Associations**

| Subnet          | Route Table |
| --------------- | ----------- |
| public-subnet-a | rt-public-a |
| public-subnet-b | rt-public-b |
| public-subnet-c | rt-public-c |

### **Private Associations**

| Subnet           | Route Table  |
| ---------------- | ------------ |
| private-subnet-a | rt-private-a |
| private-subnet-b | rt-private-b |
| private-subnet-c | rt-private-c |

### **Screenshot Placeholder**

<img width="1920" height="1080" alt="rt-SubnetAssociation" src="https://github.com/user-attachments/assets/0d74d88a-f49c-414a-953f-4ff2551ae450" />
<img width="1920" height="1080" alt="rt-private-SubnetAssociation" src="https://github.com/user-attachments/assets/8605199a-5665-4480-baea-1a6d5a8ae9b5" />

---

# ------------------------------------

# 🏷️ **8. Tagging Standards**

# ------------------------------------

### **Apply these tags to ALL resources**

```
Environment = production
Owner       = network-team
Project     = asymmetric-vpc-build
CostCenter  = AWS-Networking
```

### **Additional Tier Tag**

* Public resources: `Tier = public`
* Private resources: `Tier = private`

---

# 🎉 **Setup Completed Successfully!**

Your asymmetric VPC is now ready for:

* ALBs/NLBs
* Public EC2 instances
* Private EC2 + RDS
* Microservices
* High availability architecture

---
