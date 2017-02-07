# Deploy Tracking

If you notify Rollbar every time you deploy or release your app, you'll unlock several features that will help your debugging process.

[Instructions for reporting deploys](/docs/deploys/) are available for [Bash](/docs/deploys/bash/), [Capistrano](/docs/deploys/capistrano), [Engine Yard](/docs/deploys/engineyard/), [Fabric](/docs/deploys/fabric/), [Heroku](/docs/deploys/heroku), [MSBuild](/docs/deploys/msbuild), [Powershell](/docs/deploys/powershell), and [other tools and languages](/docs/api/deploys/#record-create-a-deploy).

## Deploys Screen

The deploy screen shows a detailed view of all deployments made to a particular environment.

![](../images/guides/deploys/deploy-screen.png)

## Suspect Deploy

When deploys are reported to Rollbar, then every detected error will have an entry in its 'Suspect Deploy' tab.

![](../images/guides/deploys/suspect-deploy.png)

The suspect deploy is one of the following:

* The last deploy prior to the first occurrence of the error _(if the item has never been resolved)_.
* The last deploy prior to the reactivation of the error _(if the item was previously resolved)_.

## Deploys in Item Feed
TODO
