# AWS Auto-Scaling Infrastructure Project

This project sets up an AWS infrastructure that automatically adds more servers when traffic gets heavy. It includes two web servers (one for students, one for faculty) with a load balancer that spreads traffic between them.

## Objective

- Creates a custom network (VPC) with two subnets in different zones
- Sets up two web servers running Apache
- Uses a load balancer to distribute traffic
- Automatically launches new servers when CPU usage goes above 65%
- Sends email alerts when scaling happens
- Everything is built using AWS CLI commands

## Why I Built This

I wanted to learn how to create a real production setup that can handle traffic spikes without going down. This project covers the main pieces you need: networking, servers, load balancing, and monitoring.

## What You Need

- An AWS account
- AWS CLI installed on your computer
- Some basic knowledge of AWS services

## Getting Started

### Step 1: Install AWS CLI

For Windows, download from: https://awscli.amazonaws.com/AWSCLIV2.msi

Open Command Prompt and check if it worked:
```cmd
aws --version
```

### Step 2: Set Up Your AWS Credentials

In Command Prompt, run:
```cmd
aws configure
```
Enter your access key, secret key, region (like ap-south-1), and set output to json.

### Step 3: Run the Commands

Open Command Prompt as Administrator and follow the commands step by step. All the infrastructure is built using Command Prompt - no clicking around in the AWS console needed!

## How It's Set Up

### Network Stuff
- **VPC**: Your own private network on AWS
- **Subnets**: Two public subnets in different availability zones
- **Internet Gateway**: So your servers can reach the internet
- **Route Tables**: Tell traffic where to go

### The Servers
- **EC2 Instances**: Small t2.micro servers (free tier)
- **Web Server**: Apache with PHP and MySQL installed
- **Custom Pages**: Simple HTML login pages for students and faculty

### Load Balancing
- **Application Load Balancer**: Sends traffic to healthy servers
- **Target Groups**: Groups your servers together
- **Health Checks**: Makes sure servers are working

### Auto Scaling
- **Auto Scaling Groups**: Manages how many servers you have
- **Scaling Rules**: Add server if CPU > 65%, remove if CPU < 30%
- **Launch Templates**: Settings for new servers

### Monitoring
- **CloudWatch**: Watches your server performance
- **SNS**: Sends you emails when things happen

## How the Scaling Works

| What Happens | When | Action |
|-------------|------|---------|
| High CPU | Over 65% for 2 minutes | Add 1 more server |
| Low CPU | Under 30% for 2 minutes | Remove 1 server |
| Minimum | Always | Keep at least 1 server |
| Maximum | Always | Never more than 2 servers |

## Main Commands I Used (All in Command Prompt)

Creating the network:
```cmd
aws ec2 create-vpc --cidr-block 192.168.0.0/16 --tag-specifications ResourceType=vpc,Tags=[{Key=Name,Value=Kavya-vpc}]
set VPC_ID=vpc-xxxxxxxxxx
aws ec2 create-subnet --vpc-id %VPC_ID% --cidr-block 192.168.0.0/24 --availability-zone ap-south-1a
```

Launching servers:
```cmd
aws ec2 run-instances --image-id ami-0e35ddab05955cf57 --instance-type t2.micro --subnet-id %SUBNET_ID_1% --security-group-ids %SG_ID% --associate-public-ip-address
```

Setting up the load balancer:
```cmd
aws elbv2 create-load-balancer --name Kavya-ALB --subnets %SUBNET_ID_1% %SUBNET_ID_2% --security-groups %SG_ID%
```

The cool thing is everything gets saved using Windows environment variables like %VPC_ID% and %SUBNET_ID_1% so you can reference them throughout the build process.

## Testing It Out

To test if scaling works, I used Git Bash (since SSH doesn't work well in regular Command Prompt) and ran:
```bash
ssh -i keyname.pem ubuntu@public-ip
sudo apt install stress -y
stress --cpu 1 --timeout 120s
```

But all the AWS infrastructure commands were done in Command Prompt. This maxes out the CPU and should trigger auto scaling after a couple minutes.

## What I Learned

- How to design a network from scratch on AWS using Command Prompt
- Setting up load balancers and auto scaling with CLI commands
- Using Windows environment variables to manage AWS resource IDs
- Writing infrastructure with Command Prompt instead of clicking around the AWS console
- How all these AWS services work together when controlled via CLI
- The power of automating everything through command line

## Files in This Project

- `Student-login.html` - Simple login page for students
- `Faculty-login.html` - Login page for faculty  
- `launch-template1.json` - Settings for student server instances
- `launch-template2.json` - Settings for faculty server instances
- `keyname.pem` - SSH key to connect to servers

## Costs

This uses mostly free tier resources:
- t2.micro instances are free for 750 hours/month
- Load balancer costs about $16/month
- Data transfer has some free allowance
- CloudWatch monitoring is mostly free

## Common Problems I Ran Into

**Servers won't start**: Check your security groups allow the right ports

**Load balancer shows unhealthy**: Make sure your web server is actually running and security groups allow port 80

**Auto scaling not working**: Double-check your CloudWatch alarms are set up right

**Can't SSH to servers**: Your security group might not allow port 22 from your IP

## Cleaning Up

Don't forget to delete everything when you're done testing. Open Command Prompt and run:
```cmd
aws autoscaling delete-auto-scaling-group --auto-scaling-group-name Kavya-ASG --force-delete
aws elbv2 delete-load-balancer --load-balancer-arn %ALB_ARN%
aws ec2 terminate-instances --instance-ids %STUDENT_LOGIN_ID% %FACULTY_LOGIN_ID%
```

## What's Next

Some ideas to make this better:
- Add a database server
- Set up HTTPS with certificates
- Use private subnets for better security
- Add more monitoring and alerts
- Try different instance types

This project taught me a lot about how real AWS infrastructure works and how powerful the command line can be. Building everything through Command Prompt gave me way more control than using the web console, and I can now script and automate the entire process.
