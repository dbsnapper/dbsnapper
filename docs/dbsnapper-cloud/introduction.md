# Introduction

![DBSnapper Cloud](https://dbsnapper.com/blog/content/images/size/w2000/2022/11/dbs-turtle-cloud-padding.png)

The [DBSnapper Cloud](https://dbsnapper.com) has launched and extends the capabilities of our standalone CLI with new options for storing sanitized snapshots in private cloud storage and sharing them with your team. Up until now, each DBSnapper setup required creating a maintaining a local configuration file and snapshots were created on local storage, with no support for making them available in a centralized location.has launched and provides centralized configuration, storage, and sharing of your database snapshots.

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

The lightweight DBSnapper CLI works on your local system or in your private cloud infrastructure. Integrating with the DBSnapper Cloud provides centralized configuration, storage, and sharing of your database snapshots.
