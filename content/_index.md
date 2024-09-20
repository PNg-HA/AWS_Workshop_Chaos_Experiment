---
title : "AWS Chaos Experiment"
date :  "`r Sys.Date()`" 
weight : 1 
chapter : false
---

# AWS Chaos Experiment

#### Introduction
In today's rapidly evolving digital landscape, system resilience is critical for businesses of all sizes. What happens when chaos hits your AWS EC2 Windows instances? Are you confident they’ll recover, or will everything crash down? This workshop is designed to equip you with a few ways to conduct chaos experiment in your EC2 instance, including: CPU-Stress, Network-latency and Stopping instances

#### AWS Fault Injection Simulator
AWS FIS is a fully managed service for running chaos engineering experiments to test and verify the resilience of your applications. FIS gives you the ability to inject faults into the underlying compute, network, and storage resources. This includes stopping Amazon Elastic Compute Cloud (Amazon EC2) instances, adding latency to network traffic and pausing I/O operations on Amazon Elastic Block Store (Amazon EBS) volumes.

Content:
1. [CPU-Stress](1-CPU-Stress)
2. [Network-latency](2-Network-latency)
3. [Stopping instances](3)
4. [Clean up](4)

#### Overall architecture
In this workshop, you wil create experiment templates in AWS FIS and conduct those experiments to EC2 instance. 3 CloudWatch Alarms are set up (CPUUltilaztion, NetworkIn, NetworkOut) to inform you whenever the experiment makes the EC2 instance overwhelmed. Besides, in this workshop, there are sections where you can use code and GitHub Action to automatically conduct, detect and remediate the chaos events.
![arch](/images/architecture.png)


{{% notice info %}}
This workshop is inspired by the repository in https://github.com/mostlycloudysky/aws-chaos-experiments. Besides, all the codes in this workshop are in https://github.com/PNg-HA/AWS-Chaos-Experiments.
{{% /notice %}}
