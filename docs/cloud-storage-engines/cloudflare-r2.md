# Cloudflare R2 Storage Engine

<p align="center">
  <img alt="Cloudflare R2 Logo" src="/static/cloudflare-r2-icon.png"  />
</p>

## Overview

The Cloudflare R2 Storage allows developers to store large amounts of unstructured data without the costly egress bandwidth fees associated with typical cloud storage services. It uses an Amazon S3 compatible API to store data with high durability and availability.

## Requirements

To use the Cloudflare R2 Storage engine, you must have a Cloudflare R2 account. You can sign up for a free Cloudflare account at [https://dash.cloudflare.com/sign-up](https://dash.cloudflare.com/sign-up). You will then need to generate an API key that consists of an __access key__ and __secret key__. Using these credentials and your cloudflare __account id__.

Create a bucket in your Cloudflare R2 account and note the __bucket name__. You can specify an optional bucket prefix that you can use to organize your snapshots.

