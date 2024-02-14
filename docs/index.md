---
title: Home
icon: material/home
description: DBSnapper simplifies the process of creating database snapshots from production data that can be used for development and testing.
---

# DBSnapper: Streamline Development with Sanitized Production Data <!-- omit in toc -->

<!-- prettier-ignore-start -->
!!! tip ":new: DBSnapper 2.0 Released!"

    DBSnapper v2.0 has been released and is now available for download. This version brings a host of new improvements including **Database Subsetting**, which allows you to create smaller, more manageable relationally-complete snapshots of your database. [Download DBSnapper v2.0](https://github.com/dbsnapper/dbsnapper/releases)
<!-- prettier-ignore-end -->

<p align="center">
  <img src="static/dbs-turtle-tagline-light.png#only-light" width="60%">
  <img src="static/dbs-turtle-tagline-dark.png#only-dark" width="60%">
</p>

DBSnapper revolutionizes the way development teams handle database snapshotting, bringing de-identified production data into the heart of development and testing workflows. It stands as a robust alternative to the traditional, often cumbersome methods of creating development and test fixtures. With DBSnapper, you get to leverage real, production-grade data, stripped of its sensitive elements, to power your development and testing environments.

## Snapshotting - Simplified database backups

DBSnapper offers an efficient and powerful solution for snapshotting databases that simplifies the snapshotting process for different database platforms.

DBSnapper's snapshotting capability is not just about capturing data; it's a strategic tool that integrates into and enhances the entire software development lifecycle. From creating realistic test environments and aiding in AI model training to providing essential support in CI/CD pipelines, DBSnapper stands as an indispensable asset for any software development team aiming to streamline and improve their database management and utilization in a modern startup environment.

- **Real-world Test Cases**: Utilizing de-identified data snapshots, you can create more effective and realistic test cases. This helps in identifying potential issues in a more accurate production-like environment.

- **Seamless Integration with CI/CD Pipelines**: DBSnapper can be integrated into CI/CD pipelines, automating the process of generating snapshots for your team and ensuring the team is using the latest and most accurate data for testing.

- **Training AI Models**: For AI and machine learning initiatives, having access to diverse, real-world data sets is crucial. DBSnapper's ability to provide de-identified snapshots of real operational data can significantly enhance the training process of AI models, leading to more accurate responses.

## :new: Subsetting - Smaller snapshots, relationally complete

DBSnapper 2.0 introduces **Database Subsetting**. This feature allows you to create smaller, more manageable relationally-complete snapshots of your database. This is particularly useful when you want to create a snapshot of a large database, but only need a subset of the data for development or testing purposes.

- **Efficient Data Management**: Instead of working with the entire database, which can be cumbersome and resource-intensive, you can select just the relevant parts you need. This leads to quicker operations, lower resource usage, and more efficient data handling.

- **Maintains Data Integrity**: Despite working with a smaller dataset, the integrity and relational structure of the original database are preserved. This is crucial when developing or testing applications, as it ensures that any interactions with the subset reflect real-world scenarios.

- **Customizable to Specific Needs**: DBSnapper 2.0 provides flexibility in defining the criteria for subsetting. Whether it's specific tables, rows, or a particular set of data, you can tailor the subset to meet the exact needs of your project.

## Sanitize - De-identification and sensitive data removal

DBSnapper enables you to de-identify and sanitize your production data, removing sensitive information such as personal details, financial data, and other confidential information. This is crucial for ensuring compliance with data protection regulations and maintaining the privacy and security of your users.

- **Data Provenance During Sanitization**: The DBSnapper tools are designed to give you can full-control over your data, ensuring that no sensitive or proprietary data leaves your environment.

- **Adherence to GDPR and Other Regulations**: In the era of stringent data protection laws like the GDPR, CCPA, and others, DBSnapper's sanitization feature ensures that your data handling practices are compliant, reducing the risk of legal complications and hefty fines.

- **Maintaining Data Utility Post-Sanitization**: Despite the removal of sensitive data, the utility and integrity of the dataset are preserved, making it suitable for development, testing, and analysis without compromising privacy.

## Share - Securely distribute snapshots via DBSnapper Cloud

The sharing aspect of DBSnapper is made possible through the DBSnapper Cloud, a critical feature for secure storage and distribution of database snapshots. It's designed for seamless collaboration within your team or for integration with automated processes.

- **Flexibility of Storage Choices**: With DBSnapper, you have the flexibility to 'Bring Your Own Cloud Storage Provider'. This means you can choose the cloud storage that best aligns with your company’s policies and data management strategies, ensuring that your data remains within your approved PaaS vendor.

- **Easy Access for Team Members**: Shared snapshots are easily accessible to authorized team members. This facilitates collaboration, as team members can work with the same data sets in a synchronized manner.

- **Integration with Automated Processes**: The DBSnapper Cloud is designed for integration with automated processes, such as CI/CD pipelines, making it simpler to incorporate database snapshots into your development and deployment workflows.

## Get Started

Ready to get started with DBSnapper? Head over to the [Installation](installation.md) guide to install DBSnapper on your system. Once installed, you can follow the [Quick Start](quick-start.md) guide to create your first snapshot.
