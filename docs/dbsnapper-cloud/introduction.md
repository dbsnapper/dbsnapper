---
title: DBSnapper Cloud Platform - Introduction
description: The DBSnapper Cloud Platform provides additional capabilities for building, managing, and sharing sanitized database snapshots.
---

# Introduction

![DBSnapper Cloud](/static/dbsnapper-cloud.jpeg)

The [DBSnapper Cloud](https://dbsnapper.com) extends the capabilities of our standalone CLI with new options for storing sanitized snapshots in private cloud storage and sharing them with your team. Up until now, each DBSnapper setup required creating a maintaining a local configuration file and snapshots were created on local storage, with no support for making them available in a centralized location. With the DBSnapper Cloud, you can speed up your workflow with a centralized configuration and make snapshots accessible to your team or development pipelines.

## DBSnapper Cloud Processing Pipeline

<p align="center">
  <img alt="DBSnapper Cloud Flow" src="/static/flow.png"  />
</p>

The DBSnapper Cloud brings everything together and provides a processing pipeline to create, store, and share sanitized database snapshots.

## Database Engines

DBSnapper provides a library of Database Engines that ensure compatibility between different database vendors.

Currently supported engines include:

- PostgreSQL Local installation
- MySQL Local installation
- PostgreSQL Docker Image
- Mysql Docker Image

More Database Engines are being created and will be added to the application in the future.

## Cloud Storage Engines

DBSnapper provides a library of Cloud Storage Engines that allow you to use your own private cloud storage provider to store your database snapshots.

Currently supported Cloud Storage engines include:

- Cloudflare R2
- Amazon S3

## On-prem and Private Cloud Compatible

The lightweight DBSnapper Agent works on your local system or in your private cloud infrastructure. Integrating with the DBSnapper Cloud provides centralized configuration, storage, and sharing of your database snapshots.
