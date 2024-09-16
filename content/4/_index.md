---
title : "Clean up"
date : "`r Sys.Date()`"
weight : 4
chapter : false
pre : " <b> 4. </b> "
---

#### Delete GitHub repository
According to [GitHub Action billing](https://docs.github.com/en/billing/managing-billing-for-github-actions/about-billing-for-github-actions), usage is free for standard GitHub-hosted runners in public repositories, which is in this workshop senario. However, for private repositories, each GitHub account receives a certain amount of free minutes and storage for use with GitHub-hosted runners, depending on the account's plan. In summary, if you don't activate the GitHub Action flow, you are free of charge even for all workflow we have builded throughout this workshop.

1. Go to the Setting section.
![4](/images/4/s1a.png)

2. Scroll down to the end to click the "Delete this repository" button. 
![4](/images/4/s1b.png)

3. A pop-up show up. Click "I want to delete this repository."
![4](/images/4/s1c.png)

{{% notice info %}}
This workshop is conducted during August, 2024. At this time, I choose GitHub Nobile to verify for any action in GitHub console, including deleting GitHub repository.
{{% /notice %}}

#### Remove AWS FIS resources
According to [AWS Prcing](https://aws.amazon.com/fis/pricing/), AWS FIS just charges for the running experiment. So the templates you have configured are free of charge if you keep them without running the experiments. 

1. Navigate to [AWS FIS Experiment templates page](https://us-east-1.console.aws.amazon.com/fis/home?region=us-east-1#ExperimentTemplates).
2. Choose deployed templates -> Click Actions. 

![4](/images/4/s2.png)

3. Choose "Delete experiment template".
![4](/images/4/s3.png)

4. A pop-up shows up. Type "delete" in the input and click "Delete experiment template" button.
![4](/images/4/s4.png)


#### Remove AWS CloudWatch Alarms 
1. Navigate to [AWS CloudWatch Alarm page](https://us-east-1.console.aws.amazon.com/cloudwatch/home?region=us-east-1#alarmsV2:).

2. Choose the alarm you have created. Select Action buttion -> Select Delete.
![4](/images/4/s5.png)

3. A pop-up shows up. Click Delete button.
![4](/images/4/s6.png)

#### Remove AWS EC2 instance

1. Access the EC2 dashboard.


2. Navigate to Instances.

3. Choose the instances associated with the lab.


4. Click on Instance state.

5. Select Terminate instance.
![4](/images/4/s7.png)