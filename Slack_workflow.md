Slack Notifications

YAML File 

Slack Configuration
Requirements:
Be logged in your Slack workspace.


In Slack we need one channel to receive notifications and a Slack app with one incoming webhook URL to be used for our GH Actions.


It is assumed that the Slack channel already exists, and it does not matter whether it is public or private, so let's create the App:


Go to https://api.slack.com/messaging/webhooks and click on the Create you Slack app button.
Click on the Create New App button and select “From scratch” option.
Choose a name for the App and select the workspace where the channel is.
Then go to Incoming Webhooks and enable that option.
Once Incoming webhooks are enabled you can Add New Webhook to Workspace.
Select your Channel in the list and click in Allow.


You should see something like this:

GitHub Actions Configuration
In this last step we will use three  actions already created:


To get notifications about PRs created by Dependabot:
https://github.com/actions/checkout
https://github.com/kv109/action-ready-for-review



To get notifications about vulnerabilities detected by Dependabot:
https://github.com/kunalnagarco/action-cve
Since not all vulnerabilities can be resolved with automatic PRs, it is good to get notifications of all detected vulnerabilities.


Now we need to create two workflows by adding the following YAML files in .github/workflows in the repository.


dependabot-pr-to-slack.yaml


name: Notify about PR ready for review

on:
 pull_request:
   branches: ["main"]


 # Allows you to run this workflow manually from the Actions tab
 workflow_dispatch:


jobs:
 slackNotification:
   name: Slack Notification
   if: startsWith(github.head_ref, 'dependabot/') # This step only runs when PR has dependabot/ HEAD
   runs-on: ubuntu-latest
   steps:
   # Latest version available at: https://github.com/actions/checkout/releases
   - uses: actions/checkout@v2.5.0
   - name: Slack Notification
     # Latest version available at: https://github.com/kv109/action-ready-for-review/releases
     uses: kv109/action-ready-for-review@0.2
     env:
       SLACK_CHANNEL: dependabot-notifications
       SLACK_WEBHOOK: ${{ secrets.SLACK_WEBHOOK }}


This workflow runs every time that Dependabot creates a new PR.




dependabot-vulns-to-slack.yaml
name: 'Dependabot vulerabilities notification to Slack'


on:
 schedule:
   - cron: '0 10 * * 1' # Cron
 # Allows you to run this workflow manually from the Actions tab
 workflow_dispatch:


jobs:
 Notify-Vulnerabilites:
   runs-on: ubuntu-latest
   steps:
     # Latest version available at: https://github.com/kunalnagarco/action-cve/releases
     - name: Notify Vulnerabilities
       uses: kunalnagarco/action-cve@v1.7.15
       with:
         token: ${{ secrets.PERSONAL_ACCESS_TOKEN }} # This secret need to be created
         slack_webhook: ${{ secrets.SLACK_WEBHOOK }} # This secret need to be created


This workflow runs periodically based on cron expression.




As is commented in the code, we need to add two secrets in our repository to be used in these workflows: PERSONAL_ACCESS_TOKEN and SLACK_WEBHOOK.


For adding both secrets follow this steps:
Go to the Setting tab in the repository.
Go to Secret → Actions in the ‘Security’ section.
Click in New repository secret and add the followings:



The names chosen are used in workflows, so if they are modified, then change them also in the YAML files.


Also, we need to add SLACK_WEBHOOK secret in Secret → Dependabot in the same way that it did before.


SLACK_WEBHOOK value is the URL created previously.


PERSONAL_ACCESS_TOKEN could be created following these steps:
Click on your profile and select Setting.
Go to Developer settings.
Click on Personal access token and choose Tokens (classic).
Click on Generate new token (classic).
Select the following permissions:




Click on Generate token and copy the generated token. This token can’t be visible later, so be sure to copy it at this time.
