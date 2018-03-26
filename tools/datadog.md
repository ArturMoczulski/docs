---
title: Integrate Rollbar with Datadog
---

# Connecting Rollbar to Datadog

Rollbar users can syndicate exceptions, errors and code deployments as 'Events' in [Datadog](https://www.datadoghq.com/).

### Setup

Configuration is per-project in Rollbar.

1. In Datadog, go to **Integrations → APIs** and copy your API key.

2. In Rollbar, go to the Notification settings page for a project: **Dashboard → Settings →
   Notifications → Datadog**.
  
3. Add your Datadog API key.

4. Click **Enable Datadog Integration**

Congrats! Datadog is now integrated with Rollbar. Once integrated, the default notification 'Rules'
will be activated and sent to your Datadog account.

### Customizing Datadog Notification Rules

The Datadog event created by each notification rule can be customized using [notification variables](/docs/notification-variables/) and filters.  Additionally, you may specify a different API key for each notification rule if you wish to send the events to a different Datadog account.

### Automatic Event Priority
When creating an event in Datadog from Rollbar, you can explicitly set the priority (`low`, `normal`) or allow Rollbar to automatically determine the priority.  Here's how priority is automatically determined:

Rollbar Notification Rule | Datadog Priority
-------------|-----------------
Item resolved | `low`
_Everything else_ |	`normal`

### Automatic Event Type
When creating an event in Datadog from Rollbar, you can explicitly set the type (`error`, `warning`, `info`, `success`) or allow Rollbar to automatically determine the type.  Here's how type is automatically determined:

Rollbar Notification Rule | Datadog Event Type
--------------------------|-------------------
Deploy | `info`
Item resolved | `success`
_Everything else_ | _determined by Rollbar item level_

Rollbar item levels are mapped to Datadog event types as follows:

Rollbar Item Level |	Datadog Event Type
-------------------|--------------------
`critical`, `error` | `error`
`warning` |	`warning`
`info`, `debug` |	`info`
