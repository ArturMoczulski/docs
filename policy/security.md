# Rollbar Security Policy

Last updated: December 15, 2014

<!-- Sub:[TOC] -->

## Disclosure and audit

Data is sent to the Rollbar API via SDKs that are installed and run on customer machines or embedded in web pages and run on customer's user's browsers. We provide and maintain official open-source libraries for several popular programming languages and platforms, including [Ruby](https://github.com/rollbar/rollbar-gem), [Python](https://github.com/rollbar/pyrollbar), [Node.js](https://github.com/rollbar/node_rollbar), [JavaScript](https://github.com/rollbar/rollbar.js), and [PHP](https://github.com/rollbar/rollbar-php). 

SDKs are small pieces of code that gather and report data to Rollbar over HTTPS. They do not generate any dynamic code, so you do not need to worry about them interfering with your codebase once installed. SDKs are also designed to run in an asynchornous manner (where available), and should not have a noticeable impact on performance.

Since all of the Rollbar SDK code is open-source, the community is able and encouraged to audit the source code. All code is hosted at [GitHub - Rollbar](http://github.com/rollbar/).

## Secured content

In order to help us improve Rollbar and user experience, Rollbar may use third-party analytics and customer service tools to better understand user behavior on our site. Any data that Rollbar collects is used solely by Rollbar and is not shared, sold or rented to any third-parties.

Data reported to the Rollbar API is never shared with any third-party service without the express consent of our customers. Rollbar integrates with many complimentary tools, including GitHub, JIRA, HipChat, PagerDuty, and many others. Each of these tools must be configured and enabled by a user before any data will be sent to them.

See our [privacy policy](http://rollbar.com/privacy) for more information.

## Data collection

Our [privacy policy](http://rollbar.com/privacy) details the type of data we collect via our website and SDKs.

All data collected from our SDKs is stored in raw and aggregated forms. Both the aggregated and raw data are available via our website as well as our API. More information about our API can be found [here](http://rollbar.com/docs/api/).

## Data transmission

### HTTPS

All data transmitted to Rollbar from our SDKs is done over SSL by default. Users can choose to configure HTTPS or HTTP in each SDK by providing a configuration option or editing the source code directly. We do not perform host authentication over HTTPS nor do we store any passwords in our SDKs. HTTPS is used solely for communication encryption.

### SSL certificates

SDKs send data to [https://api.rollbar.com/](https://api.rollbar.com/). We have an extended validation SSL certificate from [DigiCert](https://www.digicert.com) which our API servers use to ensure trusted communication between our customers and Rollbar.

### CORS (cross-origin resource sharing)
 
Rollbar's API servers are configured to allow [CORS](http://en.wikipedia.org/wiki/Cross-origin_resource_sharing) requests for API endpoints that are used by our JavaScript SDK. This enables our customers to send data from their users' browsers to Rollbar for processing. Our CORS enabled endpoints only allow POST requests.

### Serialization formats

Data from our SDKs is either serialized into JSON or multipart-mime format before being sent. Our API servers respond with application/json content-type.  

## Proxies

None of the SDKs currently support proxies. Work on this is not yet scheduled. Please contact [support@rollbar.com](mailto:support@rollbar.com) if this would be useful for you.

## Data scrubbing

Data scrubbing is the process of removing sensitive information from the data sent to Rollbar by the SDK. All of the SDKs support data scrubbing. Refer to their individual READMEs on github for information regarding configuration.

### SDKs

By default, SDKs will attempt to remove request parameters that look like sensitive information. For example, the rollbar-gem SDK scrubs the following request parameters by default:

- passwd
- password
- password_confirmation
- secret
- confirm_password
- password_confirmation
- secret_token

More info can be found in the [README](https://github.com/rollbar/rollbar-gem/blob/master/README.md#data-sanitization-scrubbing).

### Server-side

Currently we do not support data scrubbing on the server-side. If this is important to you, please email [support@rollbar.com](mailto:support@rollbar.com).

## Infrastructure and data storage

### Hosting

All of our servers are hosted on [SoftLayer](http://softlayer.com/). We have a mix of "cloud" and physical hosts which enables us to quickly scale up or down.

### DNS and load balancers

Rollbar's API and web servers use a cluster of load balancers that are hosted around the globe for fast response times. We use [DynECT Managed DNS](http://dyn.com/dynect-managed-dns) for domain name resolution and failover. DynECT automatically fails over to usable API servers in case of an unexpected error or interruption in service from our API tier.

We have load balancers in the following locations:

- Dallas
- San Jose
- Singapore
- Amsterdam

### Internal network

All data transmitted between Rollbar hosts is done over SoftLayer's internal, private network. Our main database cluster is in Dallas, with redundant hosts in Seattle and San Jose. The SoftLayer private network is only accessible by machines running within its infrastructure.

### Public network

The only hosts that are accessible from the public internet are our load balancers and development machines. The hosts run with strict firewall rules that only allow HTTP and HTTPS traffic (as well as SSH for maintenance and development operations.)

We follow best practices for securing SSH and used industry-standard tools such as fail2ban to enforce strict access policies.

### Storage

#### Raw data

The data we collect from our SDKs is referred to as "raw" data. We store this data in temporary files that have a very short lifetime (less than a few seconds) before loading them into a MySQL cluster and Memcache. The data stored in MySQL and Memcache is not encrypted, although it is compressed.

#### Uploaded data

Data that is uploaded to Rollbar by our customers is temporarily stored on disk before it is compressed, encrypted and stored in MySQL alongside our raw data. The data is encrypted with a different secret key per project.

Uploaded data includes:
- iOS dSYM files
- Javascript source maps
- Java Proguard files

Files uploaded to Rollbar are available for download by any user that has access to the project.

#### Aggregate data

We process all of the raw data into an aggregate form to present to customers. This data is stored in MySQL and Memcache.

#### Retention

Data retention period varies based on account type: 30 days for the Free plan, 90 days for Starter, and 180 days for Small and above. After that period, data is eligible for deletion. The deletion process runs periodically and includes deleting the raw data from all locations (Memcache, MySQL, and Object Storage). Aggregate data is not typically deleted.

Data can also be deleted individually on a one-off basis. To delete an individual occurrence, press the "Delete" button at the bottom of the occurrence detail page. If you need a large amount of occurrences or other data to be deleted immediately, please contact [support@rollbar.com](support@rollbar.com).

#### Logical Partitioning and Data Access

All customer raw and aggregate data is stored as part of a Project. Individual user access to a particular Project is configurable in the "Teams" section of the Rollbar interface.

To ensure that customer data cannot be accessed by other customers, customer data is always stored with its Project ID and always queried using a Project ID.

#### Data Centers
Our primary data center, where data is stored at rest, is located in Dallas, Texas. In addition, we operate points-of-presence in California (San Jose), Amsterdam, and Singapore, as well as backups and other auxiliary services at additional locations in the United States.

## Passwords

### Rollbar.com

Customer passwords for Rollbar.com are never stored. We save a secure hash of a customer's password using bcrypt. If a user forgets their password, a password-reset email will be sent to their confirmed email address.

As of July 26, 2013, we use the [py-bcrypt](http://www.mindrot.org/projects/py-bcrypt/) implementation of [bcrypt](http://en.wikipedia.org/wiki/Bcrypt) to hash passwords. Password hashes in our legacy format (salted MD5s) are replaced with bcrypt hashes upon login.

### Third-party integrations

In most cases, we use access tokens for integrations with third-party services. These access tokens are stored encrypted in our database. 

Most of the third-party integrations that Rollbar includes do not require a user's login credentials and we therefore do not store any. The only exception is JIRA. The JIRA API does not currently provide a way to authenticate a user without their username and password. We store both in MySQL using AES encryption with a different secret key per user.

## Data access

All Rollbar employees have full access to all data. Each has signed confidentiality and IP agreements as part of their employment.

Customers can only access data associated with projects that their team is allowed to access. Teams are configured by account admins (users in the "Owners" team) and can be modified at any time.

## For more help

- [Privacy policy](http://rollbar.com/privacy/)
- Email [support@rollbar.com](mailto:support@rollbar.com) for more help and information
