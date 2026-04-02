# AWS Auto Scaling Lab

## Lab Objective

This lab demonstrates **AWS Auto Scaling** using **Auto Scaling Groups (ASG)**.
You will learn how to automatically scale EC2 instances based on demand using:

* **Simple Scaling**
* **Step Scaling**

> **Note:** This lab does not include Load Balancers or SNS notifications — the focus is purely on Auto Scaling.

---

## Prerequisites

1. **AWS Account** with permissions to manage EC2 and Auto Scaling.
2. **SSH Key Pair** for connecting to EC2 instances.
3. **Basic AWS CLI or Console knowledge.**
4. **Amazon Linux 2 or Ubuntu AMI** for EC2 instances.

---

## Lab Steps

### Step 1: Launch a Base EC2 Instance

1. Open the **AWS Management Console** → EC2 → Launch Instance.
2. Choose an **AMI** (e.g., Amazon Linux 2).
3. Choose an **instance type** (t2.micro is sufficient for lab).
4. Configure **network settings** and select a **key pair**.
5. Launch and connect via SSH to ensure it’s working.

---

### Step 2: Create a Launch Template or Configuration

1. Go to **EC2 → Launch Templates** or **Launch Configurations**.
2. Configure the following:

   * AMI ID (same as Step 1)
   * Instance Type
   * Key Pair
   * Security Group (allow SSH: 22)
3. Name it `ASG-Lab-Template`.

---

### Step 3: Create an Auto Scaling Group (ASG)

1. Go to **EC2 → Auto Scaling Groups → Create ASG**.
2. Choose the **launch template/configuration** created in Step 2.
3. Configure:

   * **Group Name:** `ASG-Lab`
   * **VPC & Subnet:** Select one subnet (or multiple for advanced lab)
   * **Desired Capacity:** 1
   * **Minimum Capacity:** 1
   * **Maximum Capacity:** 3
4. Skip load balancer integration.
5. Review and **create ASG**.

---

### Step 4: Configure Scaling Policies

#### 4.1 Simple Scaling

1. In the ASG, go to **Automatic scaling → Add policy → Simple scaling**.
2. Configure:

   * Metric: **CPUUtilization**
   * Threshold: e.g., `> 70%`
   * Action: **Add 1 instance**
3. Configure cooldown: 300 seconds.

#### 4.2 Step Scaling

1. Add a **step scaling policy**.
2. Example:

   * Metric: **CPUUtilization**
   * Step adjustments:

     * 70-80% → +1 instance
     * 80-90% → +2 instances
     * > 90% → +3 instances
3. Cooldown: 300 seconds.

---

### Step 5: Testing Auto Scaling

1. Connect to the instance and **generate load**:

```bash
# Example CPU load generator
yes > /dev/null &
```

2. Monitor ASG in the console → **Instances tab**.
3. Verify new instances launch when thresholds are exceeded.
4. Stop the load and confirm instances terminate according to scaling policies.

---

## Step 6: Cleanup

1. Terminate all instances in ASG.
2. Delete the **Auto Scaling Group**.
3. Delete the **launch template/configuration** if not needed.

---

## Lab Notes

* Auto Scaling **increases fault tolerance** by maintaining a desired number of instances.
* Scaling policies help **optimize costs** and respond to traffic spikes.
* This lab focuses on **core scaling concepts** — no load balancers or SNS.

---

## References

* [AWS Auto Scaling Documentation](https://docs.aws.amazon.com/autoscaling/ec2/userguide/what-is-amazon-ec2-auto-scaling.html)
* [Scaling Policies](https://docs.aws.amazon.com/autoscaling/ec2/userguide/as-scale-based-on-demand.html)

---
