# No Touching! Touch-Free Database Snapshots with DBSnapper

_Your data is your most important asset so you must exercise caution when providing access to other applications and third parties. Here's how to create touch-free, sanitized database snapshots with DBSnapper._

![No Touching](/static/no-touching-bluth.jpg)

## Overview

You're taking the steps to sanitize your production data because you don't want PII and sensive data floating around on unsecured developer laptops. But have you considered the mechanisms behind the tools you're using to create these snapshots?  Understanding how these tools work is just as important as taking the step to sanitize your data. You want to make sure that the tools you use to create the sanitized database snapshots are secure and don't introduce any additional risk to your data.

## Risks to Consider

While it may not be possible to eliminate all the risks associated with creating sanitized database snapshots, you can take steps to miniimize the attack surface. Here are some of things to consider:

**Where does your data reside?** - Many cloud services can completely automate the task of creating database snapshots. In addition to having privilged access to your data, you should understand the intermediate and final storage locations of your snapshots. How are these secured (encrypted at rest?) Retention policies? What access controls and monitoring systems are in place to prevent unauthorized access? Likewise, if you're in a country subject to GDPR, it may be required that your data be stored datacenters that reside in the EU.

**Database Credentials** - To perform a proper snapshot, priviledged database credentials are required to access all tables and create a complete database dump. These credentials should be stored encrypted, and dedicated set of credentials should be provided for use by the snapshotting tool with restricted access (i.e. to read-only replicas).

**IP Addresses** - By default, databases are configured to disallow access from the public internet, a means to prevent unauthorized access. Some tools will require you to whitelist the IP addresses of the machines they use to create the snapshots. 

**Data in Transit** - When creating a database snapshot, the data is transferred over the network from the database to the snapshotting tool. This data should be encrypted in transit to prevent unauthorized access.

## Touch-Free Database Snapshots with DBSnapper

The DBSnapper architecture and agent provides an option to create touch-free sanitized database snapshots. Running the DBSnapper agent locally or in your private cloud environment eliminates the need to **whitelist IP addresses** or open up your database to the public internet. Sensitive **Database credentials** are encrypted in the configuration file or can be provided via environment variables to the agent process.

In this scenario, the DBSnapper agent can build the sanitized snapshot locally and provides options for storing the generated snapshots, giving you full control over the **storage location** and **retention policies** of your data:

1. **Unsanitized snapshot:** - You can choose to store the unsanitized snapshot locally, upload it to cloud storage location, or delete it entirely. 
2. **Sanitized snapshot** - Likewise, you can store the sanitized snapshot locally or in cloud storage. 
3. **DBSnapper Cloud** - Since the sanitized snapshot has lower risk profile and is intended to be shared with developers, the **DBSnapper cloud application** can manage the distribution of the sanitized database snapshots for your development team from your own cloud storage or ours.

## Get Your Touch-Free Database Snapshots Today

The tool you use choose for database snapshots should have the flexibility to meet your risk profile and desired need for control and automation. DBSnapper has the ability to operate in fully touch-free mode, automated, or a hybrid of the two. 

**Interested in getting started with DBSnapper?** [Install the DBSnapper Agent](https://docs.dbsnapper.com/installation/) and get started now, or [contact us](mailto:contact@dbsnapper.com) to learn more about how DBSnapper can help you create touch-free sanitized database snapshots.