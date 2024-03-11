---
title: Philosophy
description: Why we made DBSnapper and reasons around some of the architectural decisions.
---

## Why We Built DBSnapper

Having worked with dozens of early stage tech startups over the last 20 years, I found that development teams often struggled with using or replicating their production data for development and testing. Either the teams created brittle test fixtures, or they hacked together a process to create a snapshot of their production database. This process was often manual, error-prone, and time-consuming, and not compliant with data privacy regulations. Often times they did neither, and these were the teams that had difficulty in identifying and fixing bugs in their applications, and a low quality product.

I decided, back in 2021 to fix this problem by creating a tool that would automate the process of creating de-identified datbase snapshots. The goal was a tool that was simple, easy to use, and could be used in a variety of environments and CI/CD automation pipelines.

## Design Philosophy

At DBSnapper, we hold a strong belief that the security and privacy of data should be a top priority. Our design philosophy reflects this commitment, focusing on keeping your data safe and under your control.

<!-- prettier-ignore-start -->

### Data Provenance: Your Data, Your Control

We built DBSnapper with a key principle in mind – you should have full control over your data at all times. Here's how we ensure this:

  - **Data Security**: We've designed the DBSnapper Agent to give you full control over your data. We don't require access to your database outside of your approved infrastructure or keep copies of your data. When using the DBSnapper Cloud, we ensure sensitive information is encrypted at rest and in transit, and provide features like connection string templating to retrieve credentials from environment variables as an alternative to providing them to our service.

  - **Bring Your Own Cloud Storage**: Rather than forcing you to use our cloud object storage, we allow you to use your own provider, ensuring your data stays within your approved PaaS vendor, and reducing the overall cost of the solution.

  - **On Prem First**: The DBSnapper Agent is equally useful in on-premises environments as it is on your local machine. We've designed the agent to be easy to install and use, making it a useful tool for your development teams.


### Simplicity

Simplicity was another key design goal, ensuring the ease of installation and use, regardless of the user's environment. To achieve this, we've consciously limited dependencies, focusing on creating a tool that's as lightweight and flexible as possible.

- **Minimized Dependencies**: We've chosen Go as the development language for its ability to create statically linked binaries that can be easily distributed and run on different operating systems and platforms.

- **Database Tools**: While we rely on database tools for some of the operations, we're careful to minimize the areas in which these tools are used.

- **Database Engines and Docker**: We provide the option to specify a Docker-enabled database engine. By using Docker, we run the necessary database tools within containers. This means you don't need to install these tools on your machine, and simplifies the support for different database versions and platforms.

<!-- prettier-ignore-end -->
