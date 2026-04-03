
# AWS Auto Scaling

## 1. What is Auto Scaling?

**Auto Scaling** is a cloud feature that automatically **adjusts the number of compute resources (EC2 instances)** based on demand.

### Why it is used:

* Maintain **high availability**
* Improve **fault tolerance**
* Optimize **cost** (scale in when not needed)
* Handle **traffic spikes automatically**

---

## 2. What is an Auto Scaling Group (ASG)?

An **Auto Scaling Group (ASG)** is a service in **Amazon Web Services** that manages a group of EC2 instances.

### Key features:

* **Min Size** → Minimum number of instances
* **Max Size** → Maximum limit
* **Desired Capacity** → Target number of instances
* **Health Checks** → Replaces unhealthy instances automatically
* **Multi-AZ Support** → Distributes instances across Availability Zones

👉 ASG ensures:

* Required number of instances are always running
* Failed instances are automatically replaced

---

## 3. Types of Scaling

### 1. Dynamic Scaling

* Adjusts capacity based on **real-time metrics**
* Uses metrics like:

  * CPU utilization
  * Network traffic

**Example:**

* Add instances if CPU > 70%
* Remove instances if CPU < 30%

---

### 2. Predictive Scaling

* Uses **machine learning** to forecast future demand
* Scales **before traffic increases**

**Best for:**

* Applications with predictable patterns (daily/weekly spikes)

---

### 3. Scheduled Scaling

* Scales resources at **specific times**
* Does not depend on metrics

**Example:**

* Scale up at 9 AM (peak hours)
* Scale down at 9 PM

---

## 4. Scaling Actions (Basic Concepts)

* **Scale Out (Horizontal Scaling):** Add more instances
* **Scale In:** Remove instances

---

## 5. Auto Scaling Policies

### 1. Target Tracking Scaling Policy

* Maintains a **specific target metric**
* Automatically adjusts capacity to meet the target

**Example:**

* Keep CPU at **50%**
* If CPU increases → scale out
* If CPU decreases → scale in

👉 **Most commonly used policy**

---

### 2. Step Scaling Policy

* Scales in **steps based on thresholds**
* Different actions for different metric ranges

**Example:**

* CPU 60–70% → add 1 instance
* CPU 70–80% → add 2 instances

👉 More **controlled and flexible**

---

### 3. Simple Scaling Policy

* Adds/removes a **fixed number of instances**
* Triggered by an alarm

👉 Less flexible, mostly **deprecated**

---

## 6. Important Concepts

### Cooldown Period

* Time to wait after a scaling activity before another action
* Prevents **rapid scaling fluctuations**

---

### Health Checks

* ASG continuously monitors instance health
* Replaces unhealthy instances automatically

---

### Load Distribution

* Usually used with a Load Balancer (optional but common)
* Distributes traffic across instances

---

## Quick Summary

* **Auto Scaling** → Automatically adjusts resources
* **ASG** → Manages EC2 instances
* **Scaling Types** → Dynamic, Predictive, Scheduled
* **Policies** → Target Tracking, Step, Simple
* **Goal** → High availability + Cost optimization

