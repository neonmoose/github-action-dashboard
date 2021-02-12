# GitHub Action Dashboard

When our current CI/CD provider shutdown I found myself evaluating GitHub actions as an alternative. Great solution with one problem. There was no single pane of glass to see the status of all the builds in our GitHub organization. Instead you had to go into each repo, check the action status, etc.

Looked around for solutions to the problem and found very few.  Meercode was a SaaS that was available but connecting it to my GitHub account at the time I tested it it required granting it permission to act on my behalf. I couldn't see a way that my employer would be cool with that.

A self hosted solution seemed like the way to go but I couldn't really find any.  Surprising given the popularity of GitHub.

## Limitations

* Single organization/username.  Currently the dashboard requires you to specify the organization or the username of the repositories which show on the dashboard. It doesn't support multiple organizations or usernames.
* Currently the dashboard doesn't support GitHub webhooks.  Supporting them in a self hosted solution in a secure fashion isn't simple.  MVP just uses the GitHub apis. 


## How it works

* Upon startup all repositories for the organization/username are iterated. 
* Each repository is checked for workflows.
* All repositories with workflows are cached.
* Each workflow has it's runs listed and the most recent run for each branch are returned.

Every 5 minutes the runs for the cached repositories/workflows are refreshed.  Five minutes was chosen so as to not hit GitHub API limits.

When you click the refresh button in the dashboard it refreshes all runs associated with that workflow across all branches.  This is refreshed server side so that other consumers of the dashboard also see the update prior to the refresh of all data.

The dashboard itself updates once a minute.  It will receive whatever data is currently cached on the server when it refreshes.

## Setup GitHub App

The dashboard runs as a GitHub App.  It does not automatically register itself as a GitHub app.  Automatic registration is difficult if the dashboard is private and not exposed on the internet.  Instead you need to manually setup a GitHub app for your personal account or an organization.

Steps:
* Go into the settings for your org or personal account
* Click Developer Settings
* Click GitHub Apps
* Click New GitHub App
** Add an app name and homepage url
** Repository Permissions:
*** Action: read-only
** Where can this GibHub App be installed: Only on this account
** Click Create GitHub App
* You should now be on the general settings page for the app
* Click Generate a new client secret and save off the client secret as it will disappear after you navigate off the page.
* Click Generate a private key
* Change to Install App page
* Click Install

## Start

The dashboard has all of it's parameters passed via environment variables.

### Variables

* GITHUB_USERNAME or GITHUB_ORG - Only one valid is allowed.  If both are specified GITHUB_ORG takes precedence and the GITHUB_USERNAME is ignored.
* GITHUB_APPID - The AppId from the GitHub App general settings page.
* GITHUB_APP_PRIVATEKEY - The private key from the GitHub App general settings page. All line feeds must be converted to \n characters so the environment variable and the entire setting should be surrounded with single quotes.
* GITHUB_APP_CLIENTID - The client id from the GitHub App general settings page.
* GITHUB_APP_CLIENTSECRET - The client secret from the GitHub App general settings page.
* GITHUB_APP_INSTALLATIONID - Installation id that can be retrieved using steps in the next section.
* DEBUG=action-dashboard:* - Optional setting to help in debugging

### Installation Id

For some reason GitHub doesn't expose the installation id anywhere in the UI. I have provided a utility to retrieve it.  You will need all the variables from the previous section with the exception of GITHUB_APP_INSTALLATIONID to do this.

#### Option 1

Requires nodejs installed locally.

~~~bash
GITHUB_USERNAME=XXXXXXX GITHUB_APPID=XXXXXX GITHUB_APP_PRIVATEKEY='-----BEGIN RSA PRIVATE KEY-----\nXXXXXX\nXXXXXXX\nXXXXXXX\n-----END RSA PRIVATE KEY-----' GITHUB_APP_CLIENTID=XXX.XXXXXXXXXXXXXXXX GITHUB_APP_CLIENTSECRET=XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX node getinstallationid.js
~~~

#### Option 2

Requires docker installed locally.

~~~bash