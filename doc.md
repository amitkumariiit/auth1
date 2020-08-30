## Context

For the slack-github integration, the slack app is currently on legacy app. Slack is deprecating the legacy app support and we need to build the new app and start using that and slowly get rid of the legacy app.


## GOAL
1. Remove legacy slack app from slack marketplace (by end of 31-Dec-2020)
2. Release new slack app in slack marketplace
3. Deprecate old slack app functionality (migrate users from old to new) (tentatively by end of 30-Jun-2021)

ChatOps encompasses two applications one on Slack end and the other on GitHub side. As part of this document, app available in the Slack marketplace will be called Slack app whereas app available in the GitHub marketplace will be referred to as GitHub app.

## GitHub App
### Option - 1 Use existing github app
#### Pros
1. No need to auther and maintain two apps
#### Cons
1. Current chatOpsGithubUser table don't have chatops clientAppId column (slackAppId in this case). So, only one secret can be stored per github app per github user. 
2. To handle point no-1 need to **alter** chatOpsGithubUser table to add clientAppId.
3. Common endpoints for signIn and installations callback need to be modified to handle both the legacy and new app.
4. Risk of regression in legacy support.

### Option - 2 Build new github app for new slack app
#### Pros
1. Better segregation of APIs.
2. Elimination of regression risk as old APIs will not to be changed.
#### Cons
1. Author and maintain two gitHub app.


Authoring a new GitHub app is a trivial work and considering the benefits of Option-2, we will go ahead with option-2

## Slack App

### Option-1: New Slack app + current Slack backend + Feature Parity with current Slack app
 
1. New slack app with current javascript slack code only.
2. Feature parity with current slack app.
3. At a later point of time after releasing, migrate to chatops code base.

#### Pros
1. Easy way to use the new app with minimum code changes
2. We can roll out the feature at the earliest of all the other options.
3. Will be rolling out all the features in one go.

#### Cons
1. Multiple testing. 1st on releasing the app, 2nd on moving to the chatops code base
2. Current Slack tables like (slackUsers, githubUsers) were designed to support storage of information for one app. There will be a need to alter existing tables to accommodate support for storing data for 2 apps. This brings in **regression risk**.
 
### Option-2:  New Slack app + ChatOps backend + Feature Parity with Teams + Scheduled Reminder

1. New slack app using new chatops code base. (typecript codebase and new chatops tables)
2. Feature parity with teams app + schedule reminder (schedule reminder only for slack app)

#### Pros-
1. New slack app will be on chatops backend
2. No migration work for moving users of new slack app to chatops backend
3. Users will get most used features including Scheduled Reminder
4. Regression risk eliminated as there will be no change to current Slack code base and tables

#### Cons-
1. New slack app with no advanced features like deploy, create issue, snooze, open/reopen/close pr/issue.
2. New slack app will not be in parity with the current slack app, so existing customers might hesitate to move to the new slack app unless we develop all the features.


### Option-3: New Slack app + ChatOps backend + Feature Parity with current Slack app 

1. ChatOps backend to support all the features supported by Slack
2. Develop new slack app on ChatOps backend leveaging the common code functionality of chatops codebase.

#### Pros
1. We can ship new app with full feature parity with the legacy app
2. Customer’s don’t see any feature difference and hence no complaint from them.
3. Regression risk eliminated as there will be no change to current Slack code base and tables

#### Cons
1. Teams client lag the feature, but Teams app can catch up in next release

**Option to go with is option-3**


### Following are the actions that we would like to do as part of the activity either on short or long term
1. Registering the new slack app with current name only
2. Exploring what changed in new slack app and what changes are need in code to supprt that
6. Renaming the old slack app (before releasing the new slack app)
7. Renaming the old github app (before releasing the new slack app)
8. Registering new github app with current name only
9. Hosting new slack app on slack.github.com and opening a hidden url for legacy slack.github.com for debugging
3. Move the slack features to the new chatops code base
4. Deprecating the old slack app

