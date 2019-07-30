# apigee-trireme-to-hosted-target-migration

## Summary
This repository represents the best practices to migrate from Apigee Trireme to Hosted Targets.  

## Hosted targets
There is a cost to using Hosted Targets (HTs) and there is also a limit - 30 HTs maximum deployed across all organizations or 20 HTs per organization.  However, this cost is comparable to what an organization would pay if it were to host the Node.js code on another platform.  If you don't want to pay the additional cost, then identify a provider to host your Node.js code (i.e. Google Cloud Platform GKE, or app engine).

## Best practices
Identify all the proxies to obtain an accurate count and estimate the level of effort and time to complete the migration.

### Process Overview
1. Identify migration team members (product owner, developers, testers)
2. Identify all Trieme proxies (org/env) and BU/team owners.
3. Identify all test cases/test scripts/automated tests for Trieme proxies.
4. Identify which features are used by Trieme proxies.
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
Follow Agile best practices and estabilish an Agile team with committed team members to complete the migration.

### Identification - TODO
* Identify all the Trieme proxies across all organizations that need to be migrated.
  This is important to estabilish how many proxies will be migrated and also how many HTs you will need to pay above the free tier.  

### Migration TODO
#### a127-magic
#### apigee-cache
#### apigee-access KVMs
#### apigee-access

### Testing
