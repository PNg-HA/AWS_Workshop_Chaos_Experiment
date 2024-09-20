---
title : "Actions workflow to Inject CPU Stress & Remediate"
date : "`r Sys.Date()`"
weight : 5
chapter : false
pre : " <b> 1.5. </b> "
---

Reference: .github/workflows/inject-cpu-stress.yml from the repo mostlycloudysky/aws-chaos-experiments.

All Actions-related components use a new instance in us-east-2.

#### 1. Prerequisites

a. Create instance
![1.5](/images/1/1.5/Picture1.png)
b. Notification
![1.5](/images/1/1.5/Picture2.png)
c. Setup secrets

According to the reference YAML file, it is necessary to set up secrets for the action (based on the inject-cpu-stress.json file, you can determine the format of the secrets), including:

- AWS_ACCESS_KEY_ID: ***
- AWS_SECRET_ACCESS_KEY: ***
- AWS_REGION: us-east-2 (change of pace)
- AWS_ACCOUNT_ID: *****
- INSTANCE_ID: i-****
- IAM_ROLE: <role name after the “/” in the role ARN, e.g., HaiAnh-FIS>
- CLOUDWATCH_ALARM_NAME: CPUUtilizationAlarm
- GITHUB_TOKEN: Not needed because a temporary token will be created automatically each time the job runs

Step 1: Go to the repository settings
![1.5](/images/1/1.5/Picture3.png)
Step 2: In the sections on the left, below "Secrets and variables," select Actions.
![1.5](/images/1/1.5/Picture4.png)
Step 3: Select "New repository secret" and enter the secret name and value.
![1.5](/images/1/1.5/Picture5.png)
However, there will be a naming violation error due to the name containing special characters. Referring to this [GitHub discussion](https://github.com/orgs/community/discussions/11941), it seems they did not fix it, so we will use HashiCorp Vault (suggested by the GitHub Action course on KodeKloud). Steps 4 to 14 use HashiCorp, but there was still an error. It turns out that there was a special character in the YAML file that was not an underscore. Just pasting it into a place without font formatting, like a URL input field, will resolve the issue.

Step 4: Go to https://www.vaultproject.io/. Select "Try HCP Vault".
![1.5](/images/1/1.5/Picture6.png)
Step 5: Select Sign in with GitHub account
![1.5](/images/1/1.5/Picture7.png)
Step 6: If you don’t have an organization, create one. If you already have one, select that organization (currently, only one organization is allowed).
![1.5](/images/1/1.5/Picture8.png)
Step 7: Select Projects -> View project
![1.5](/images/1/1.5/Picture9.png)
Step 8: Scroll down or locate the left sidebar, and select Vault Secrets
![1.5](/images/1/1.5/Picture10.png)
Step 9: Select Create first app
![1.5](/images/1/1.5/Picture11.png)
Step 10: Name it “AWS-FIS-secrets” and select Create App
![1.5](/images/1/1.5/Picture12.png)
Step 11: Select Create new secret -> Static secret
![1.5](/images/1/1.5/Picture13.png)
Step 12: Enter the Secret and then Save.
![1.5](/images/1/1.5/Picture14.png)
Step 13: Select Integrations on the left sidebar -> GitHub Actions 
![1.5](/images/1/1.5/Picture15.png)
Step 14: Select Authorize HCP Vault Secrets
![1.5](/images/1/1.5/Picture16.png)
The above steps demonstrate how to store secrets on Vault. Now, let's return to storing secrets on GitHub Actions.

Step 15: Add the secrets to the repo as in step 3
![1.5](/images/1/1.5/Picture17.png)
#### 2. Set up

Push the fis-templates folder, the scripts folder, and the .github folder from the sample repo to the personal repo
![1.5](/images/1/1.5/Picture18.png)
#### 3. Run the workflow action

Step 1: Go to the Actions tab of your personal repository, select "Inject CPU Stress Experiment" from the left sidebar -> Click "Run workflow" -> Click the green button labeled "Run workflow".
![1.5](/images/1/1.5/Picture19.png)

Step 2: Observe the workflow
![1.5](/images/1/1.5/Picture20.png)
Observe the steps:
![1.5](/images/1/1.5/Picture21.png)
The following step to create an experiment template for CPU stress was observed to be successful:
![1.5](/images/1/1.5/Picture22.png)
Wait 20 minutes to proceed with the experiment (You can check in this Action log for more information: https://github.com/PNg-HA/AWS-Chaos-Experiments/actions/runs/10514440060/job/29132272786):
![1.5](/images/1/1.5/Picture23.png)
![1.5](/images/1/1.5/Picture24.png)
![1.5](/images/1/1.5/Picture25.png)
Observe the experiment's execution timeline. Check the FIS console:
![1.5](/images/1/1.5/Picture26.png)
Check alarm:
![1.5](/images/1/1.5/Picture27.png)
Check CPUUtilizationAlarm:
![1.5](/images/1/1.5/Picture28.png)
The alarm notifies via email:
![1.5](/images/1/1.5/Picture29.png)
Email CPU Utilization Alarm:
![1.5](/images/1/1.5/Picture30.png)
Everything is going according to the workflow! Once this step is completed, the following steps will finish very quickly, and the workflow will be completed:
![1.5](/images/1/1.5/Picture31.png)
Check workflow:
![1.5](/images/1/1.5/Picture32.png)
Not the expected flow due to CPU stress for 10 minutes and normal recovery before the experiment step completed:
![1.5](/images/1/1.5/Picture33.png)
Therefore, when running the step “Detect Issues,” the alarm does not trigger and will not create an issue on the personal repository; in other words, there is no issue to address, so no resolution is possible:
![1.5](/images/1/1.5/Picture34.png)
To temporarily resolve the issue, create a file named detect_cpu_stress_issues.yml (copy the content from inject-cpu-stress.yml, remove the steps related to the experiment, and keep only the Detect Issues step) for the detect cpu stress workflow and run it manually.

Proceed to rerun the experiment:
![1.5](/images/1/1.5/Picture35.png)
The workflow inject-cpu-stress.yml handles the scenario where the experiment template already exists by retrieving the existing template ID and saving it to $GITHUB_OUTPUT for the step 'Start FIS Experiment':
![1.5](/images/1/1.5/Picture36.png)
During the experiment, activate the Detect CPU Stress workflow:
![1.5](/images/1/1.5/Picture37.png)
Check the detailed workflow, the issue has been created:
![1.5](/images/1/1.5/Picture38.png)
Go to the issue just created from Action, and manually label it with 'cpu-stress':
![1.5](/images/1/1.5/Picture39.png)
Result:
![1.5](/images/1/1.5/Picture40.png)
Try closing the issue to see if the remediate workflow is triggered:
![1.5](/images/1/1.5/Picture41.png)
Check Workflow remediate:
![1.5](/images/1/1.5/Picture42.png)
Workflow details:
![1.5](/images/1/1.5/Picture43.png)
Monitor CloudWatch CPU alarm:
![1.5](/images/1/1.5/Picture44.png)