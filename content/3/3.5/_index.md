---
title : "Actions workflow to Stop EC2 Instances & Remediate"
date : "`r Sys.Date()`"
weight : 5
chapter : false
pre : " <b> 3.5. </b> "
---

Reference: .github/workflows/stop-instances.yml and .github/workflows/remediate-stop-instances.yml from the repo mostlycloudysky/aws-chaos-experiments

#### 1. Prerequisites

The resources for EC2 instance, SNS, and GitHub repo have been created in the CPU Stress section.

#### 2. Set up

Push the fis-templates folder, scripts folder, and .github folder from the sample repo to the personal repo (already completed in the CPU Stress section).
![3.5](/images/3/3.5/Picture1.png)
#### 3. Run the workflow action

a. Run the experiment

Step 1: Go to the Actions of your personal repo, select "Stop Instances Experiment" from the left sidebar -> Choose "Run workflow" -> Click the green button labeled "Run workflow".
![3.5](/images/3/3.5/Picture2.png)
Step 2: Observe the workflow
![3.5](/images/3/3.5/Picture3.png)
Observe the steps:
![3.5](/images/3/3.5/Picture4.png)
However, it failed to run. The step “Retrieve the latest FIS Experiment Template ID” is supposed to retrieve the template ID if it exists; if not, it returns “new” and proceeds to the “Replace” and “Create” steps. However, the log shows that these two steps did not run, which means the condition of the if statement on line template_id = new in the Retrieve step is incorrect. Upon checking, it turns out to be incorrect, so it needs to be fixed:
![3.5](/images/3/3.5/Picture5.png)
There is still one incorrect JSON name on line 54. Please correct it:
![3.5](/images/3/3.5/Picture6.png)
Push code to the repository:

Observe the steps of the job:
![3.5](/images/3/3.5/Picture7.png)
Observe the step to create an experiment template, it shows successful creation:
![3.5](/images/3/3.5/Picture8.png)
Observe the experiment timeline. Check the FIS console:
![3.5](/images/3/3.5/Picture9.png)
Check the details of the running Experiment:
![3.5](/images/3/3.5/Picture10.png)
However, the Detect issues step did not detect:
![3.5](/images/3/3.5/Picture11.png)
Diagnosis: Due to the instance stopping slower than the script execution process.

I ran it again (added a print(response) for easier debugging) and created the issue:
![3.5](/images/3/3.5/Picture12.png)
b. Remediation

Step 1: To add a label to the issue, select the issue -> Click the settings icon next to "Labels" -> Choose the "remediate" label. Result:
![3.5](/images/3/3.5/Picture13.png)
Step 2: Try to resolve it by closing this issue by selecting “Close issue”. 

The Remediate Workflow is triggered:
![3.5](/images/3/3.5/Picture14.png)
Step 3: Check workflow details
![3.5](/images/3/3.5/Picture15.png)
Step 4: Check the output of the "Remediate stopped Instances" step:
![3.5](/images/3/3.5/Picture16.png)
The current state is pending and the previous state is stopped, indicating that the instance is being restarted.

Step 5: Check the EC2 console:
![3.5](/images/3/3.5/Picture17.png)
Step 6: Check the CloudTrail logs:
![3.5](/images/3/3.5/Picture18.png)
Thus, although it is not automated because the detection part needs to be run manually (due to the workflow logic where detection occurs after the experiment step), it generally resolves the issue successfully.