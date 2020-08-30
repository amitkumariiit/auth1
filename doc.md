Context

For the slack-github integration, the slack app is currently on legacy app. Slack is deprecating the legacy app support and we need to build the new app and start using that and slowly get rid of the legacy app.


GOAL
Remove legacy slack app from slack marketplace (by end of 31-Dec-2020)
Release new slack app in slack marketplace
Deprecate old slack app functionality (migrate users from old to new) (tentatively by end of 30-Jun-2021)

ChatOps encompasses two applications one on Slack end and the other on GitHub side. As part of this document, app available in the Slack marketplace will be called Slack app whereas app available in the GitHub marketplace will be referred to as GitHub app.

GitHub App
Option - 1 Use existing github slack app
Cons
Using so, the same github secret would be used by both. 
Difficulty on making the choice on cleaning up secrets on signout from any app.
Logic for processing all the endpoints for github app like callback/installations need to be modified to handle both the legacy and new app.
Risk of regression in legacy support.

In ChatopsGithubUsers table, we don’t have a column for clientAppId (slack app id). With this limitation, we can’t use the table to store the github user info that could be used for both slack apps because if someone signs out from one app, difficulty in deleting the shared data. This can be handled using adding clientAppId or the referenceCount etc but those are not so ideal solutions.

Option - 2 Build new github app for new slack app
Pros
Better segregation of APIs.
Elimination of regression risk as old APIs will not to be changed.
Cons
Author new gitHub app.


Authoring a new GitHub app is a trivial work and considering the benefits of Option-2, we will go ahead with option-2
Slack App

Option-1: New Slack app + current Slack backend + Feature Parity with current Slack app
 
New slack app with old code only.
Feature parity with current slack app.
At a later point of time after releasing, migrate to chatops code base.

Pros
Might be an easy way to use the new app with minimum code changes
We can roll out the feature at the earliest of all the other options.
Will be rolling out all the features in one go only.
Cons
Multiple testing. 1st on releasing the app, 2nd on moving to the chatops code base
Current Slack tables like (slackUsers) were designed to support storage of information for one slack app. There will be a need to alter existing tables to accommodate support for storing data for 2 slack app. This brings in regression risk.
 
Option-2:  New Slack app + ChatOps backend + Feature Parity with Teams + Scheduled Reminder


New slack app using new chatops code base.
Feature parity with teams app + schedule reminder (schedule reminder only for slack app)
Pros-
New slack app will be on chatops backend
No migration work for moving users of new slack app to chatops backend
Users will get most used features including Scheduled Reminder
Regression risk eliminated as there will be no change to current Slack code base and tables

Cons-
New slack app with no advanced features like deploy, create issue, snooze, open/reopen/close pr/issue.
New slack app will not be in parity with the current slack app, so existing customers might hesitate to move to the new slack app unless we develop all the features.



Option-3: New Slack app + ChatOps backend + Feature Parity with current Slack app 

ChatOps backend to support all the features supported by Slack
Develop new slack app on ChatOps backend

Pros
We can ship new app with full feature parity with the legacy app
Customer’s don’t see any feature difference and hence no complaint from them.
Regression risk eliminated as there will be no change to current Slack code base and tables
Cons
Teams client lag the feature, but Teams app can catch up in next release


Deployment
We should host the new slack app server on separate instances because of the following reasons.

Hosting slack.github.com
For a new slack app, we need to host a website similar to what we have now as slack.github.com. The new site can be slackV2.github.com. Now with the same deployment we can host two different sites.

Probot instance
We can’t have two probot instances running on the same machine or making it run will face difficulties.

No risk of regression
No risk of regression in legacy app with any change made for the new app.


Following are the actions that we would like to do as part of the activity either on short or long term.
Registering the new slack app
Making changes to the code to make compatibility with the new slack app
Move the codebase of slack features to the new chatops code base
Possibly need to build the delta features for the teams also to achieve point no 3
Deprecating the old slack app

