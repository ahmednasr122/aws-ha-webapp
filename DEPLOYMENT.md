# Deployment Guide - AWS Console

## Step-by-Step Instructions

### Prerequisites
- AWS Account
- EC2 Key Pair (optional)

### Time Required: ~30 minutes

---

## 1️⃣ Create VPC

1. Go to **VPC Console** → **Create VPC**
2. Select **VPC and more**
3. Settings:
   - Name: `HA-WebApp-VPC`
   - IPv4 CIDR: `10.0.0.0/16`
   - AZs: `2`
   - Public subnets: `2`
   - Private subnets: `2`
   - NAT gateways: `In 1 AZ`
4. Click **Create VPC**

---

## 2️⃣ Create Security Groups

### ALB Security Group
1. **EC2 Console** → **Security Groups** → **Create**
2. Settings:
   - Name: `ALB-SG`
   - VPC: `HA-WebApp-VPC`
   - Inbound: HTTP (80) from 0.0.0.0/0
3. Click **Create**

### EC2 Security Group
1. **EC2 Console** → **Security Groups** → **Create**
2. Settings:
   - Name: `EC2-SG`
   - VPC: `HA-WebApp-VPC`
   - Inbound: HTTP (80) from `ALB-SG`
3. Click **Create**

---

## 3️⃣ Create Target Group

1. **EC2 Console** → **Target Groups** → **Create**
2. Settings:
   - Type: `Instances`
   - Name: `HA-WebApp-TG`
   - Protocol: HTTP, Port: 80
   - VPC: `HA-WebApp-VPC`
   - Health check path: `/`
3. Click **Next** → **Create** (don't register targets)

---

## 4️⃣ Create Load Balancer

1. **EC2 Console** → **Load Balancers** → **Create**
2. Select **Application Load Balancer**
3. Settings:
   - Name: `HA-WebApp-ALB`
   - Scheme: Internet-facing
   - VPC: `HA-WebApp-VPC`
   - Subnets: Select both PUBLIC subnets
   - Security group: `ALB-SG`
   - Listener: HTTP:80 → `HA-WebApp-TG`
4. Click **Create**
5. **Copy the DNS name!**

---

## 5️⃣ Create Launch Template

1. **EC2 Console** → **Launch Templates** → **Create**
2. Settings:
   - Name: `HA-WebApp-LT`
   - AMI: Amazon Linux 2023
   - Instance type: t3.micro
   - Security group: `EC2-SG`
3. **User data** (paste this):

```bash
#!/bin/bash
yum update -y
yum install -y httpd
systemctl start httpd
systemctl enable httpd

INSTANCE_ID=$(ec2-metadata --instance-id | cut -d " " -f 2)
AZ=$(ec2-metadata --availability-zone | cut -d " " -f 2)

cat > /var/www/html/index.html <<EOF
<!DOCTYPE html>
<html>
<head>
    <title>HA Web App</title>
    <style>
        body {
            font-family: Arial;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            color: white;
            text-align: center;
            padding: 50px;
        }
        .container {
            background: rgba(255,255,255,0.1);
            padding: 40px;
            border-radius: 15px;
            max-width: 600px;
            margin: 0 auto;
        }
        .info {
            background: rgba(255,255,255,0.2);
            padding: 20px;
            border-radius: 8px;
            margin: 15px 0;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>🚀 High Availability Web App</h1>
        <div class="info"><strong>Instance:</strong> $INSTANCE_ID</div>
        <div class="info"><strong>AZ:</strong> $AZ</div>
        <div class="info"><strong>Status:</strong> ✅ Healthy</div>
        <p>Running in private subnet, no public IP!</p>
    </div>
</body>
</html>
EOF
```

4. Click **Create**

---

## 6️⃣ Create Auto Scaling Group

1. **EC2 Console** → **Auto Scaling Groups** → **Create**
2. Settings:
   - Name: `HA-WebApp-ASG`
   - Launch template: `HA-WebApp-LT`
   - VPC: `HA-WebApp-VPC`
   - Subnets: Select both PRIVATE subnets
3. **Load balancing**:
   - Attach to existing: `HA-WebApp-TG`
   - Enable ELB health checks
   - Grace period: 300 seconds
4. **Group size**:
   - Desired: 2
   - Min: 2
   - Max: 4
5. **Scaling policy**:
   - Target tracking
   - Metric: Average CPU
   - Target: 70%
6. Click **Create**

---

## 7️⃣ Test Your Application

1. Wait 3-5 minutes for instances to be healthy
2. Go to **Target Groups** → `HA-WebApp-TG` → **Targets** tab
3. Wait for both targets to show **Healthy**
4. Open your ALB DNS name in browser:
   ```
   http://[YOUR-ALB-DNS-NAME]
   ```
5. Refresh multiple times to see different instances!

---

## ✅ Verification Checklist

- [ ] 2 instances running in private subnets
- [ ] Both instances show "Healthy" in target group
- [ ] No public IPs on EC2 instances
- [ ] Web page loads via ALB DNS
- [ ] Refresh shows different instance IDs

## 🎉 Success!

Your high availability web app is now running!
