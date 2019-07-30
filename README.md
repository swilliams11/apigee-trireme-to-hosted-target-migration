# apigee-trireme-to-hosted-target-migration

## Summary
This repository represents the best practices to migrate from Apigee Trireme to Hosted Targets.  

## Hosted targets
There is a cost to using Hosted Targets (HTs) and there is also a limit - 30 HTs maximum deployed across all organizations or 20 HTs per organization.  However, this cost is comparable to what an organization would pay if it were to host the Node.js code on another platform.  If you don't want to pay the additional cost, then identify a provider to host your Node.js code (i.e. Google Cloud Platform GKE, or app engine, etc.).

## Best practices
Identify all the proxies to obtain an accurate count and estimate the level of effort and time to complete the migration.

### Process Overview
1. Identify migration team members (product owner, developers, testers)
2. Identify all Trireme proxies (org/env) and the owners of these proxies (Business Unit owners or same BU different team).
3. Identify all test cases/test scripts/automated tests for Trireme proxies.
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
6. Establish CI/CD pipeline to promote code across environments.
7. Convert Trireme proxy to HT
8. Test: Unit test, automated test, performance test

### Agile development and Team Structure
Follow Agile best practices and establish an Agile team with committed team members to complete the migration.

### Identification - TODO
* Identify all the Trireme proxies across all organizations that need to be migrated.
  This is important to establish how many proxies will be migrated and also how many HTs you will need to pay above the free tier.  

### Migration TODO
#### a127-magic
#### apigee-cache
#### apigee-access KVMs
#### apigee-access

### Initialization of Apigee proxy with apigee-access KVMs
If you initiate variables or environment variables in your Trireme proxy with apigee-access (KVM), then you will need to take the following approach to use within HTs.  


* TODO - document environment variable naming best practices
  * do not include periods (.) in your environment variable names.  This will not be populated correctly when you deploy the proxy as an HT.  

### Testing
The best case scenario is you already have automated test and unit test script available.  You should be able execute the unit test before Trireme migration and then after it and receive the same results.
