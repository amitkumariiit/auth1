
# Context
As we are migrating from the legacy workspace model built github slack app to the new granular bot permission model app, we need to close on the migration options we have.
This doc talks about the available options.

Creating the new slack app is the only option as there is no option to migrate the workspace model slack app to the new granular bot permissions. This limitation is from Slack.

# Limitations in new app model
1. At a time in a workspace only one app (legacy or new) can exist because both apps will have the same slash command (i.e /github) and two apps can't have the same slash command in a workspace. (Practically it can be but there won’t be consistent behaviour on which app will pick the slack command)
2. While installing the new app, users can't choose channels for the app to be installed like workspace apps. So after installation, the app needs to be invited in the channels for the slash commands or notifications to work.

Documentation: https://api.slack.com/authentication/quickstart#public

[Note] Bot can gain abilities to post in public channels by asking for them explicitly by having the scops [chat:write.public](https://api.slack.com/scopes/chat:write.public)

# Communicating about app update
1. We can send ephemeral messages about the update every time the user runs the command from the legacy app. We can show the banner in the posted notifications also.
2. Sending the email communications to our customers.

# Post installation messages
We need to communicate the following information to the users once they installed the new app.

1. Channels on which legacy app was installed.

We can send the list of messages as a direct message to the installing user about the exhaustive list of channels along with the invite command like below that user can run to invite the new app.

/invite @github channelName1

/invite @github channelName2

/invite @github channelName3

We can also post this message (/invite @github) to invite new apps to each channel from the legacy app. Need to post from legacy app because new app can post to the private channels.

[Note] We can also automatically [invite](https://api.slack.com/methods/admin.users.invite) bot to channels from code, but this would need the user level token with [admin:users:write](https://api.slack.com/scopes/admin.users:write) scope and this scope opens up other privileges also. For this only use-case, taking depending on user token with admin scope doesn’t seem promising. 

2. List of active subscriptions along with the features.

We need to post this list to each channel from the legacy app so that users would know what all subscription commands to be run. (If we are migrating this data along with the new installation we don’t need to do this)


# Challenges with data-migration
To enable seamless migration for users so that notifications keep working as before after installing the new app, following needs to be done.
1. We need to use the same github app so that users don’t need to install it on all the subscribed repositories.
  1. Github app installation flow is triggered only from the subscribe command, so we don’t actually have any other flow to prompt users for this installation if we go with the different app approach.
2. Cons of using the same github app
  1. We need to support both apps (legacy and new) scenarios from the same endpoints for the following.
     1. Webhook url (Where we get receive the github events)
     2. Authorization callback url
     3. App installation setup callback url
  2. So we need to mess up with the legacy endpoints and risk of regression will be there, especially for supporting webhooks.
  3. To support webhooks for both apps
     1. We always need to fetch subscriptions from both old and legacy tables to know from which app notifications need to be posted. So making extra calls. So when the new app is installed, always making extra redundant calls considering we cleanup legacy tables on update.
     2. GitHub event listeners with the octokit library can only be registered in the legacy code and to use the new chatops codebase for event handlers, significant amount of changes needed in both legacy and chatops code base.
     3. So both sides of the feature **need to be tested**.
     4. Also, after retiring the legacy app, when we will delete the legacy javascript codebase, we need to redo to register github event listeners with octokit library with the chatops code base, so again will **need the testing effort**.
  4. We can’t have separate deployment (instances) supporting the new app because endpoints have to be the same, hence can’t have the benefit of abstracting new app with the legacy.

# Options for migration
## Options-1 Migrate without user data
This is how flow look like
1. User installs new app
2. Post list of subscriptions to the channels from legacy app on each channel.
3. DM the list of invite commands to the installing user.
4. Uninstall legacy app (automatically)
### Pros
1. No extra burden of data migration.
2. We can go with a different github app. Users would need to install this in the repo on running subscribe command for the first time.
3. Benefits of going with the different github app
   1. We can define all the endpoints other than the legacy app. like
      1. Webhook url
      2. Authorization callback url
      3. App installation setup callback url
   2. And hence no need to mess up with the legacy endpoints and no risk of regression there.
   3. We can have completely separate deployment (instances) supporting the new app (Till the EOL of legacy app)
   4. New app **focused development**. **No need to test legacy scenarios** because of this abstraction.
   5. When declaring an EOL of the legacy app, we can just disable the webhook or delete the github app and help lowering the events being delivered for the stale installations (scenario- github app installed on repo but no subscription).
      This could significantly reduce the load on the event queue on github.com side.
### Cons
1. Users would need to sign in and subscribe again. But if we are giving an exhaustive list subscription with features, users can easily follow through to recreate the subscription.

## Option-2 Migrate with user data
Data to migrate- user mappings and github access tokens and subscriptions.

This is how user flow look like
1. Install new app
2. DM the list of invite commands to the installing user.
3. Uninstall legacy app (automatically)

### Pros
1. Users have all the subscriptions. Just after inviting the app, notifications will start working as before. (On public channels even before inviting it will work because bot by default has the permission to post message to the public channels)
### Cons
2. To support this scenario we need to use the same github app and challenges of using the same github is mentioned in the above section.
