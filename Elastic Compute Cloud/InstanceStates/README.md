# AWS EC2 Instance States

Amazon EC2 instances go through different **states during their lifecycle**. These states indicate the current condition of the instance such as whether it is starting, running, stopping, or terminated.

Understanding instance states is important for **managing infrastructure, troubleshooting issues, and optimizing costs**.

---

# 1. What is an EC2 Instance State?

An **instance state** represents the current status of a virtual server running in Amazon EC2.

Each instance moves through a sequence of states from the moment it is launched until it is terminated.

Example lifecycle:

```text
Pending → Running → Stopping → Stopped → Terminated
```

---

# 2. EC2 Instance States Overview

AWS EC2 instances have the following main states:

| State         | Description                        |
| ------------- | ---------------------------------- |
| Pending       | Instance is launching              |
| Running       | Instance is active and operational |
| Stopping      | Instance is shutting down          |
| Stopped       | Instance is powered off            |
| Shutting-down | Instance is being terminated       |
| Terminated    | Instance is permanently deleted    |

---

# 3. Pending State

The **Pending state** occurs immediately after launching an EC2 instance.

During this stage:

* AWS allocates hardware resources
* Networking is configured
* The operating system is initialized
* Instance boot process starts

Characteristics:

* Temporary state
* Automatically transitions to **Running**

Example:

```text
Launch Instance → Pending
```

---

# 4. Running State

The **Running state** means the EC2 instance is fully operational.

During this stage:

* The operating system is running
* Applications can execute
* SSH or RDP connections are allowed
* Network traffic can flow to and from the instance

Characteristics:

* Full compute charges apply
* Instance is available for use

Example:

```text
Pending → Running
```

---

# 5. Stopping State

The **Stopping state** occurs when a user stops the instance.

During this stage:

* The operating system begins shutting down
* AWS prepares to stop the virtual machine

Characteristics:

* Temporary state
* Automatically transitions to **Stopped**

Example:

```text
Running → Stopping
```

---

# 6. Stopped State

The **Stopped state** means the instance is powered off but still exists.

During this stage:

* The instance is not running
* Compute charges stop
* Storage charges (EBS) still apply

Characteristics:

* Instance can be restarted
* Public IP may change after restart unless using Elastic IP

Example:

```text
Stopping → Stopped
```

---

# 7. Shutting-down State

The **Shutting-down state** occurs when an instance is being terminated.

During this stage:

* AWS prepares to permanently delete the instance
* The shutdown process begins

Characteristics:

* Temporary state
* Transitions to **Terminated**

Example:

```text
Running → Shutting-down
```

---

# 8. Terminated State

The **Terminated state** indicates that the instance has been permanently deleted.

During this stage:

* Instance resources are released
* Instance cannot be restarted
* Instance ID is no longer usable

Characteristics:

* Final state
* No further charges for compute

Example:

```text
Shutting-down → Terminated
```

---

# 9. EC2 Instance Lifecycle Diagram

```text
        Launch
          │
          ▼
       Pending
          │
          ▼
       Running
       │     │
       │     ▼
       │   Stopping
       │     │
       │     ▼
       │   Stopped
       │     │
       ▼     │
   Shutting-down
          │
          ▼
      Terminated
```

---

# 10. Instance State Transitions

| Current State | Possible Next State      |
| ------------- | ------------------------ |
| Pending       | Running                  |
| Running       | Stopping / Shutting-down |
| Stopping      | Stopped                  |
| Stopped       | Pending                  |
| Shutting-down | Terminated               |
| Terminated    | None                     |

---

# 11. Important Notes

* You are **charged for compute resources only in the Running state**.
* When an instance is **Stopped**, you only pay for attached storage (EBS).
* When an instance is **Terminated**, the compute resource is permanently removed.
* Instances cannot be restarted once they reach the **Terminated state**.

---

# 12. Checking Instance State

Using AWS CLI:

```bash
aws ec2 describe-instances
```

Using AWS Console:

1. Open **EC2 Dashboard**
2. Navigate to **Instances**
3. View the **Instance State** column

---

# Summary

| State         | Meaning                         |
| ------------- | ------------------------------- |
| Pending       | Instance is starting            |
| Running       | Instance is active              |
| Stopping      | Instance is shutting down       |
| Stopped       | Instance is powered off         |
| Shutting-down | Instance is being terminated    |
| Terminated    | Instance is permanently deleted |

---

# Conclusion

EC2 instance states represent the lifecycle of a virtual machine in AWS. Understanding these states helps administrators manage infrastructure efficiently and control operational costs.


# AWS EC2 Instance Actions

Amazon EC2 provides several instance actions that allow users to control the lifecycle and behavior of virtual machines. These actions help administrators manage compute resources efficiently and control infrastructure costs.

Common EC2 instance actions include:

* Start
* Stop
* Reboot
* Terminate
* Hibernate

Each action affects the instance differently in terms of **availability, billing, and resource allocation**.

---

# 1. Start Instance

The **Start** action is used to power on an EC2 instance that is currently in the **Stopped state**.

When an instance starts:

* The operating system boots
* The instance becomes available for use
* Applications and services start running

Characteristics:

* Instance transitions from **Stopped → Pending → Running**
* Compute charges begin once the instance enters the Running state

Example use case:

```text
Restarting a development server after maintenance
```

---

# 2. Stop Instance

The **Stop** action shuts down the EC2 instance but does not delete it.

When an instance is stopped:

* The operating system shuts down
* Compute resources are released
* The instance enters the **Stopped state**

Characteristics:

* No compute charges while stopped
* Storage charges (EBS) still apply
* Instance can be started again later

State transition:

```text
Running → Stopping → Stopped
```

Important notes:

* Public IP address may change when the instance starts again
* Elastic IP addresses remain unchanged

Example use case:

```text
Stopping development servers during non-working hours to save costs
```

---

# 3. Reboot Instance

The **Reboot** action restarts the operating system of the EC2 instance.

It is similar to restarting a computer.

Characteristics:

* Instance remains on the same host machine
* Instance does not move to another physical server
* No change to instance state

State transition:

```text
Running → Reboot → Running
```

Billing:

* Instance continues to incur charges

Example use case:

```text
Applying system updates that require a restart
```

---

# 4. Terminate Instance

The **Terminate** action permanently deletes the EC2 instance.

When an instance is terminated:

* Compute resources are released
* Instance cannot be restarted
* Instance enters the **Terminated state**

State transition:

```text
Running → Shutting-down → Terminated
```

Important notes:

* Root EBS volume may be deleted depending on configuration
* All instance store data is permanently lost
* Instance ID cannot be reused

Example use case:

```text
Deleting temporary infrastructure after a project is completed
```

---

# 5. Hibernate Instance

The **Hibernate** action saves the current memory (RAM) state of the instance to the root EBS volume before stopping it.

When the instance starts again, the previous state is restored.

Characteristics:

* RAM state is preserved
* Applications continue from the same state
* Boot time is faster

State transition:

```text
Running → Stopping (Hibernate) → Stopped
```

Requirements:

* Instance must support hibernation
* Root volume must be encrypted
* Sufficient EBS storage for RAM snapshot

Example use case:

```text
Long-running computations that need to resume quickly
```

---

# 6. Comparison of Instance Actions

| Action    | Instance State After | Data in RAM | Instance Restartable | Billing            |
| --------- | -------------------- | ----------- | -------------------- | ------------------ |
| Start     | Running              | No          | Yes                  | Yes                |
| Stop      | Stopped              | Lost        | Yes                  | No compute charges |
| Reboot    | Running              | Lost        | Yes                  | Charged            |
| Terminate | Terminated           | Lost        | No                   | No further charges |
| Hibernate | Stopped              | Preserved   | Yes                  | No compute charges |

---

# 7. Important Considerations

Public IP behavior:

* **Stop / Start** may assign a new public IP
* **Elastic IP** remains unchanged

Storage behavior:

* **EBS volumes persist** unless configured to delete on termination
* **Instance store volumes are lost** when instance stops or terminates

---

# 8. Performing Instance Actions

Using AWS Console:

1. Open **EC2 Dashboard**
2. Select the instance
3. Click **Instance State**
4. Choose the required action

Available options:

```text
Start
Stop
Reboot
Terminate
Hibernate
```

---

# 9. AWS CLI Commands

Start instance:

```bash
aws ec2 start-instances --instance-ids i-123456
```

Stop instance:

```bash
aws ec2 stop-instances --instance-ids i-123456
```

Reboot instance:

```bash
aws ec2 reboot-instances --instance-ids i-123456
```

Terminate instance:

```bash
aws ec2 terminate-instances --instance-ids i-123456
```

---

# Summary

| Action    | Purpose                            |
| --------- | ---------------------------------- |
| Start     | Power on a stopped instance        |
| Stop      | Shut down instance temporarily     |
| Reboot    | Restart operating system           |
| Terminate | Permanently delete instance        |
| Hibernate | Stop instance while preserving RAM |

---

# Conclusion

EC2 instance actions allow administrators to manage compute resources effectively. Understanding these options helps optimize **performance, availability, and cost management** when operating workloads in AWS.
