# apigee-trireme-to-hosted-target-migration

## Summary
This repository represents the best practices to migrate from [Apigee Trireme](https://docs.apigee.com/api-platform/nodejs/overview-nodejs-apigee-edge) based proxies, which is [deprecated](https://docs.apigee.com/release/deprecated-features), to [Hosted Targets](https://docs.apigee.com/api-platform/hosted-targets/hosted-targets-overview).  

## Hosted targets
There is a cost to using Hosted Targets (HTs) and there is also a [limit](https://specs.apigee.com/files/ApigeeEdgeCloud_user_ComparisonMatrix.pdf) - 30 HTs maximum deployed across all organizations or 20 HTs per organization.  However, this cost is comparable to what an organization would pay if it were to host the Node.js code on another platform.  If you don't want to pay the additional cost, then identify a provider to host your Node.js code (i.e. Google Cloud Platform GKE, Google app engine, etc.).

## Best practices
Identify all the proxies to obtain an accurate count and estimate the level of effort and time to complete the migration.

### Process Overview
1. Identify migration team members (product owner, developers, testers)
2. Identify all Trireme proxies (org/env) and the owners of these proxies (Business Unit owners or same BU different team).
3. Identify all test cases/test scripts/automated tests for Trireme proxies.
   * create missing test cases and scripts
4. Identify which features are used by Trireme proxies.
   * a127-magic
   * cache
   * kvms
   * apigee-access
5. Determine replacements for these features
   * a127-magic ~ swagger tools
   * cache ~ Apigee cache proxy or alternative Node.js caching module
   * kvm ~ Apigee KVM proxy or alternative Node.js kvm module
   * apigee-access ~ could either include caching or KVM or Apigee flow variables
6. Establish CI/CD pipeline to promote HT code across environments.
7. Convert Trireme proxy to HT
8. Test: Unit test, automated test, performance test

### Agile development and Team Structure
Follow Agile best practices and establish an Agile team with committed team members to complete the migration.

### Identification
* Identify all the Trireme proxies across all organizations that need to be migrated.
  This is important to establish how many proxies will be migrated and also how many HTs you will need to pay for above the free tier.  

### Migration
#### Startup script
When you application first starts, make sure your code does not specify a port or IP address. Your code should look similar to what is shown below.
```
var port = process.env.PORT || 8080;
app.listen(port, () => {
   console.log(`Server listening on port ${port}`)
 })
```

Avoid writing code as shown below. This code assigns a static IP (localhost) as the IP address and Nginx will not be able to resolve the static IP.  If you receive a 502 Bad Gateway from Nginx, then this is likely the cause of the error.  Allow the Google/Apigee platform to assign the IP and port to avoid conflicts with other Hosted Targets.  

```
var ip = process.env.IP || 'localhost';
var port = process.env.PORT || 8080;
app.listen(port, ip);
```

#### a127-magic
* The a127-magic library is no longer supported and it is not actively under development.  Please use [apigee-tool](https://www.npmjs.com/package/apigeetool) as the next best alternative to deploy Hosted Targets to Apigee Edge.  
* However, you can use the a127-magic library for routing from the Swagger file to a controller.
  * https://github.com/apigee-127/a127-documentation/wiki/Configuration#configuring-arbitrary-values
#### apigee-cache
Apigee does not provide an alternative to apigee-cache, which uses Apigee Edge Cache.  Therefore, you should modify the existing apigee-cache library to make the API requests directly to your proxy or create a new library that sends API requests to your API proxy which using Apigee Caching.  
* https://docs.apigee.com/api-platform/nodejs/access-cache-nodejs

#### apigee-access KVMs
#### apigee-access
None of the functions on apigee-access will work in a Hosted Target environment.  
  * getVault()
    * Vault was replaced by encrypted KVMs and this method will not work in Hosted Targets.
    * In this case you will need to write another module to create, store and retrieve values from an Encrypted KVM.  
  * getQuota()
    * Use the [Apigee Quota policy](https://docs.apigee.com/api-platform/reference/policies/quota-policy) in the Hosted Target proxy to limit access to your API.  

#### volos-quota-apigee
This module will not function correctly in a Hosted Targets environment.  In this case you should use the Apigee [Quota policy](https://docs.apigee.com/api-platform/reference/policies/quota-policy) in the proxy to limit access to your API.

#### volos-cache-apigee
This module will not function correctly in a Hosted Targets environment.  Consider finding an alternative caching solution such as [node-cache](https://www.npmjs.com/package/node-cache).


### Initialization of Apigee proxy with apigee-access KVMs
If you initiate variables or environment variables in your Trireme proxy with apigee-access (KVM), then you will need to take the following approach to use within HTs.  

* Do not include periods (.) in your environment variable names [`manifest.yaml`](https://docs.apigee.com/api-platform/hosted-targets/hosted-targets-reference#manfiest-file-syntax) file, as these will not be populated correctly when you deploy the proxy as an HT and your application will silently fail.  

### Logging
Logging is going to be critical to determine what errors are occurring within your Hosted Target backend, so be sure to include logging in your Node.js code.

#### Request Message ID
The [Message ID](https://docs.apigee.com/api-platform/reference/variables-reference#messageid) property is a globally unique value that is created for every request and allows you to create correlate requests from Apigee Edge to your target servers.  Consider forwarding this property to your Hosted Target via a request header.

#### Logging in Node.js
Consider including the Apigee Request Message ID in your logging server so that you can quickly correlate errors in Apigee Edge and your target server.  

#### API Monitoring
Take advantage of [API Monitoring](https://docs.apigee.com/api-monitoring), which gives you the ability to view the log messages and the Request Message ID.  This can help you trace the failure from Apigee Edge to your Hosted Target backend.  

### Testing
The best case scenario is you already have automated tests and unit test scripts available.  You should execute the unit tests before Trireme migration and then execute them again during and after the migration and you should receive the same results.

You also should be execute your test cases as part of your CI/CD pipeline.  

#### No Automated tests
If you don't have test scripts or automated tests, then you will need to define this before you start migrating proxies and this could add a significant amount of time to complete the migration.  
