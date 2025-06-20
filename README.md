# Mini Project: Configuring Auto Scaling with ALB using Launch Template

## Purpose:
In this mini project, you will learn how to configure Auto Scaling in AWS with an Application Load Balancer (ALB) using a Launch Template. The project aims to demonstrate the automatic scaling of EC2 instances based on demand, while leveraging the benefits of a Launch Template.

## Objectives:

Create Launch Template:

Learn how to create a Launch Template with the required specifications.
Set Up Auto Scaling Group:

Configure an Auto Scaling group to manage the desired number of EC2 instances using the Launch Template.
Configure Scaling Policies:

Set up scaling policies to adjust the number of instances based on demand.
Attach ALB to Auto Scaling Group:

Connect the Auto Scaling group to an existing ALB.
Test Auto Scaling:

Verify that the Auto Scaling group adjusts the number of instances in response to changes in demand.
Project Tasks:
## Task 1: Create Launch Template
1. Log in to the AWS Management Console.

2. Navigate to the EC2 service.

3. In the left navigation pane, click on "Launch Templates."

4. Click the "Create launch template" button.

5. Configure the launch template settings, including the AMI, instance type, and user data.

## Task 2: Set Up Auto Scaling Group
1. In the AWS Management Console, navigate to the EC2 service.

2. In the left navigation pane, click on "Auto Scaling Groups."

3. Click the "Create Auto Scaling group" button.

4. Choose "Use Launch Template" and select the Launch Template you created.

5. Configure the Auto Scaling group settings, including the group name, desired capacity, and initial instances.

6. Set up additional configurations such as network settings, subnets, and scaling policies.

## Task 3: Configure Scaling Policies
1. In the Auto Scaling group configuration, navigate to the "Scaling policies" section.

2. Click on "Create scaling policy" and configure policies for scaling in and scaling out based on demand.

## Task 4: Attach ALB to Auto Scaling Group
1. In the Auto Scaling group configuration, navigate to the "Load balancing" section.

2. Click on "Edit" and select the ALB to associate with the Auto Scaling group.

## Task 5: Test Auto Scaling
1. Generate traffic to trigger scaling policies.


```
sudo amazon-linux-extras install epel -y
sudo yum install stress -y

stress -c 4 
```
2. Monitor the Auto Scaling group and verify that the number of instances adjusts based on demand


## AWS CLI Scripts for all Task

### Create Launch Template
```
aws ec2 create-launch-template \
  --launch-template-name MyLaunchTemplate \
  --version-description "v1" \
  --launch-template-data '{
      "ImageId": "ami-0abcdef1234567890",
      "InstanceType": "t2.micro",
      "SecurityGroupIds": ["sg-0123456789abcdef0"],
      "UserData": "'$(base64 -w 0 user-data.sh)'",
      "KeyName": "my-key"
  }'

```

### Creating Auto Scaling Group
```
aws autoscaling create-auto-scaling-group \
  --auto-scaling-group-name MyAutoScalingGroup \
  --launch-template "LaunchTemplateName=MyLaunchTemplate,Version=1" \
  --min-size 1 --max-size 4 --desired-capacity 2 \
  --vpc-zone-identifier "subnet-0123456789abcdef0,subnet-abcdef0123456789" \
  --target-group-arns arn:aws:elasticloadbalancing:region:account-id:targetgroup/my-target-group/abc123 \
  --health-check-type EC2 \
  --health-check-grace-period 300

```

**Creating Scaling Policies
```
# Scale out policy
aws autoscaling put-scaling-policy \
  --auto-scaling-group-name MyAutoScalingGroup \
  --policy-name ScaleOutPolicy \
  --policy-type TargetTrackingScaling \
  --target-tracking-configuration file://scale-out.json

```

**scale-out.json**
```
{
  "PredefinedMetricSpecification": {
    "PredefinedMetricType": "ASGAverageCPUUtilization"
  },
  "TargetValue": 60.0,
  "DisableScaleIn": false
}

```

## Terraform Template
Create a `main.tf` file

```
provider "aws" {
  region = "us-east-1"
}

resource "aws_launch_template" "lt" {
  name_prefix   = "my-launch-template"
  image_id      = "ami-0abcdef1234567890"
  instance_type = "t2.micro"
  key_name      = "my-key"
  user_data     = base64encode(file("user-data.sh"))
  vpc_security_group_ids = ["sg-0123456789abcdef0"]
}

resource "aws_autoscaling_group" "asg" {
  desired_capacity     = 2
  max_size             = 4
  min_size             = 1
  vpc_zone_identifier  = ["subnet-0123456789abcdef0", "subnet-abcdef0123456789"]
  target_group_arns    = [aws_lb_target_group.target.arn]
  launch_template {
    id      = aws_launch_template.lt.id
    version = "$Latest"
  }
}

resource "aws_autoscaling_policy" "scale_out" {
  name                   = "scale-out-policy"
  policy_type            = "TargetTrackingScaling"
  autoscaling_group_name = aws_autoscaling_group.asg.name
  target_tracking_configuration {
    predefined_metric_specification {
      predefined_metric_type = "ASGAverageCPUUtilization"
    }
    target_value = 60.0
  }
}

```

## Cloudwatch Alarm Configuration

```
aws cloudwatch put-metric-alarm \
  --alarm-name "HighCPUUtilization" \
  --metric-name CPUUtilization \
  --namespace AWS/EC2 \
  --statistic Average \
  --period 60 \
  --threshold 60 \
  --comparison-operator GreaterThanThreshold \
  --evaluation-periods 2 \
  --dimensions Name=AutoScalingGroupName,Value=MyAutoScalingGroup \
  --alarm-actions arn:aws:autoscaling:region:account-id:scalingPolicy:policy-id:autoScalingGroupName/MyAutoScalingGroup:policyName/ScaleOutPolicy

```

## Testing + Stress Script with Logging

**user-data.sh (for launch template):**

```
#!/bin/bash
yum update -y
amazon-linux-extras install epel -y
yum install -y stress
stress -c 4 > /var/log/stress-test.log 2>&1 &

```

To run a test manually after deployment:

```
ssh ec2-user@<instance-ip>
sudo yum install -y stress
stress -c 4

```
Monitor instance activity with:

```
aws autoscaling describe-auto-scaling-groups --auto-scaling-group-names MyAutoScalingGroup
aws autoscaling describe-scaling-activities --auto-scaling-group-name MyAutoScalingGroup
aws cloudwatch get-metric-statistics \
  --namespace AWS/EC2 \
  --metric-name CPUUtilization \
  --start-time $(date -u -d '10 minutes ago' +%Y-%m-%dT%H:%M:%SZ) \
  --end-time $(date -u +%Y-%m-%dT%H:%M:%SZ) \
  --period 60 \
  --statistics Average \
  --dimensions Name=AutoScalingGroupName,Value=MyAutoScalingGroup

```
