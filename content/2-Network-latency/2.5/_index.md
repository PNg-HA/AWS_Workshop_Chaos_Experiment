---
title : "Actions workflow to Inject Network Latency & Remediate"
date : "`r Sys.Date()`"
weight : 5
chapter : false
pre : " <b> 2.5. </b> "
---

Reference: .github/workflows/ inject-network-latency.yml và .github/workflows/remediate-network-latency.yml của repo mostlycloudysky/aws-chaos-experiments. 


#### 1. Prerequisites

The EC2 instance, SNS, GitHub repo, and CloudWatch Alarm resources were created in the CPU Stress section.

Setup secrets

According to the reference YAML file, it's necessary to set up the secrets for the action. However, since these were already added in the CPU Stress section, you only need to change CLOUDWATCH_ALARM_NAME to the value NetworkInAlarm.

Step 1: Go to the settings of the repository.
![2.5](/images/2/2.5/Picture1.png)
Step 2: On the left-hand side sections, under "Secrets and variables," select Actions.
![2.5](/images/2/2.5/Picture2.png)
Step 3: Select "New repository secret" and set the secret name and value.
![2.5](/images/2/2.5/Picture3.png)
Step 4: Add the secrets to the repo

#### 2. Set up

Push the fis-templates folder, scripts folder, and .github folder from the sample repo to your personal repo (already done in the CPU Stress section).
![2.5](/images/2/2.5/Picture4.png)
#### 3. Run the workflow action

a. Run the experiment

Step 1: Go to Actions in your personal repository, select "Inject Network Latency Experiment" from the left sidebar -> Select Run workflow -> Click the green button labeled "Run workflow."
![2.5](/images/2/2.5/Picture5.png)
Received notification:
![2.5](/images/2/2.5/Picture6.png)
Step 2: Observe the workflow
![2.5](/images/2/2.5/Picture7.png)
Observe the steps:
![2.5](/images/2/2.5/Picture8.png)
Observe the step to create an experiment template for Inject Network Latency, it was created successfully:
![2.5](/images/2/2.5/Picture9.png)
Wait a few minutes to proceed with the experiment:
![2.5](/images/2/2.5/Picture10.png)
Observe the timeline for running the experiment. Check the FIS console:
![2.5](/images/2/2.5/Picture11.png)
Check details of the running Experiment:
![2.5](/images/2/2.5/Picture12.png)
Check NetworkInAlarm:
![2.5](/images/2/2.5/Picture13.png)
Alarm notification sent to email:
![2.5](/images/2/2.5/Picture14.png)
Mail NetworkInAlarm:
![2.5](/images/2/2.5/Picture15.png)
Everything is still proceeding according to the workflow! However, once this step is completed, there will be no more alarms like in the CPU Stress test part.

b. Manually run detect

Therefore, I created a file named detect-network-latency.yml for the workflow to detect network latency (removing the steps for creating and running experiments, and keeping the detect step from the inject-network-latency.yml file):

Manually run detect:
![2.5](/images/2/2.5/Picture16.png)
Detect alarms and create issues on personal GitHub repo:
![2.5](/images/2/2.5/Picture17.png)
Observe the issue in detail:
![2.5](/images/2/2.5/Picture18.png)
c. Remediation

Step 1: Close the issue by selecting “Close issue” to see if it triggers the remediate workflow.
![2.5](/images/2/2.5/Picture19.png)
Step 2: Check workflow remediation:
![2.5](/images/2/2.5/Picture20.png)
However, the remediation did not run:
![2.5](/images/2/2.5/Picture21.png)
Although there are still pending alarms:
![2.5](/images/2/2.5/Picture22.png)
Reviewing the workflow of inject-network-latency.yaml, the job only runs when there is a network-latency label:
![2.5](/images/2/2.5/Picture23.png)
However, none of the files mention labeling issues; it turns out that labeling needs to be done manually.
![2.5](/images/2/2.5/Picture24.png)
Step 3: To create a label for an issue on your personal repo, go to the "Issue" tab and select the "Labels" button next to the green "New issue" button:
![2.5](/images/2/2.5/Picture25.png)
Step 4: Create the labels "network-latency", "remediate", and "cpu-stress":
![2.5](/images/2/2.5/Picture26.png)
Step 5: To add a label to an issue, select the issue -> Choose the setting next to "Labels" -> Select the label network-latency:
![2.5](/images/2/2.5/Picture27.png)
Result:
![2.5](/images/2/2.5/Picture28.png)
Step 6: Attempt to remediate by closing this issue by selecting "Close issue."

The Remediate Workflow is triggered:
![2.5](/images/2/2.5/Picture29.png)
Bước 7: Check workflow details:
![2.5](/images/2/2.5/Picture30.png)
Step 8: Check the instance on the EC2 page. Initially, the EC2Rescue instance is created:
![2.5](/images/2/2.5/Picture31.png)
Then, the instance for the experiment was stopped completely:
![2.5](/images/2/2.5/Picture32.png)
The volume of the instance in the issue that was shut down has been transferred to the EC2Rescue instance:
![2.5](/images/2/2.5/Picture33.png)
A few minutes later, the instance shuts down, the instance for the experiment is restarted, and note that the volume has been reattached:
![2.5](/images/2/2.5/Picture34.png)
Montior NetworkIn via CloudWatch:
![2.5](/images/2/2.5/Picture35.png)

Thus, although it is not automated because the detect part has to be run manually (due to the workflow logic where detection occurs after the experiment step), it generally resolves the issue successfully.