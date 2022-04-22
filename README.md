# DBSnapper - Database snapshots made useful <!-- omit in toc -->

`dbsnapper` is a developers friend when it comes to creating and using database snapshots for development and testing purposes. 

- [Features](#features)
- [Requirements](#requirements)
- [Limitations](#limitations)
- [Installation](#installation)
- [Usage](#usage)
  - [Quick Start](#quick-start)
  - [Initialization](#initialization)
- [Configuration file](#configuration-file)
  - [Target definitions](#target-definitions)
  - [Secrets encryption](#secrets-encryption)
  - [IMPORTANT NOTE REGARDING `dst_url`](#important-note-regarding-dst_url)
- [Commands](#commands)
  - [build](#build)
  - [targets](#targets)
  - [target](#target)
  - [config init](#config-init)
## Features
- Create database snapshots with the `build` command
- Load database snapshots to a target database
- Specify multiple target configurations to represent your workflow
- Sensitive data in configuration file is encrypted
## Requirements
- Docker - Used to access datbase dump and restore commands (pgdump / pgsql )
- PostgreSQL - Currently works with PostgreSQL databases, support for other databases will come in future releases
## Limitations
Builds are available for the following OSes, not all have been tested yet:
- ✅ MacOS universal
- ✅ Linux - Tested on Ubuntu
- 🤷 Windows - Untested

Only works with PostgreSQL databases at the moment, but more will be added soon.

## Installation
__Mac:__ The easiest way to install `dbsnapper` is via [Homebrew](https://brew.sh/)

```sh
brew install dbsnapper/tap/dbsnapper
```

__Linux:__ (.deb):

```sh
# Download the release
TAG=0.9.6
wget https://github.com/dbsnapper/dbsnapper/releases/download/v"$TAG"/dbsnapper_"$TAG"_Linux_amd64.deb 

## Install with dpkg
dpkg -i dbsnapper_"$TAG"_Linux_amd64.deb
```

## Usage

Execute dbsnapper with:
```
$ dbsnapper [command] 
```

Provide secret on command line with:

```
$ DBSNAPPER_SECRET_KEY=<your_secret_key> dbsnapper [command]
```

### Quick Start

1. Initialize the configuration `dbsnapper config init`
2. Add one or more databse `targets` to configuration at `~/.config/dbsnapper/dbsnapper.yml` see [Target definitions](#target-definitions)
3. List all targets, size, and status with `dbsnapper targets`
4. Build a snapshot for target `app` with `dbsnapper build app`
5. List all snapshots for target `app` with `dbsnapper target app`

### Initialization
`dbsnapper` uses a configuration file and working directory which can be created with the init command:

`dbsnapper config init`

This will create:

- Configuration file at: `~/.config/dbsnapper/dbsnapper.yml`
- Working directory at `~/.dbsnapper`

The initial configuration file will have values for the `secret_key` and `working_directory`

Example: 

```yml
# dbsnapper.yml
secret_key: c614a689a559d1b517c28a5e4fcdc059
working_directory: /Users/joescharf/.dbsnapper
```

## Configuration file
The config file specifies your targets along with system settings such as working directory and secret encryption key.

Supported configurations
- `secret_key` (`string[32]`) - 16 byte hexadecimal string (32 characters) that acts as your encryption key. Created on `init`. Can be provided via `DBSNAPPER_SECRET_KEY` environment variable instead of in configuration file.
- `working_directory` (`string` optional) - Working directory used to store database snapshots. Defaults to `~/.dbsnapper` 
- `targets` - One or more entries specifying a snapshot workflow
  - `<target_name>` - specifies the name of the target definition
    - `src_url` - connection string of source database
    - `dst_url` - connection string of source database: __THIS WILL BE OVERWRITTEN ON LOAD!__
    - `snap` -
    - `query` -

### Target definitions
Add multiple target definitions that represent the source, destination, and sanitization query files

Example - Adding configuration for `app` target
```yml
targets:
  app:
    src_url: postgresql://postgres:postgres@localhost:15432/app?sslmode=disable
    dst_url: postgresql://postgres:postgres@localhost:15432/app_snap?sslmode=disable
```

You can now list the known targets with the command `dbsnapper targets`
### Secrets encryption

The configuration file encryption routines are run on every command invocation. Any new unencrypted sensitive data will be automatically encrypted. Our `app` target defined above will be encrypted similar to the following:

Example - `app` target after encryption
```yml  app:
    src_url: postgresql://postgres:$aes$bf7b67ddbf88c9becd6c208e40ce127cc2164afe74f6c7347a6bce367e1a582b09ff88c6@localhost:15432/app?sslmode=disable
    dst_url: postgresql://postgres:$aes$f24821b81b7e96942450a6735c4fbdb7e2ad23981bc61be7d3a0280679e0411eae512bd7@localhost:15432/app_snap?sslmode=disable
    snap: ""
    query: ""
```

---

### IMPORTANT NOTE REGARDING `dst_url` 

To prevent potential data loss, we want to __STRESS__ the fact that a database definition specified in the `dst_url` of a target definition will be __DROPPED__ and __RECREATED__ when using the `load` command!

---

## Commands

### build
Creates a snapshot of a database per the target definition. Uses `src_url` to specify the source database

Usage:
```sh
$ dbsnapper build <target_name>
```

### targets
List all targets and properties such as their size and status (whether they can be reached)

Usage:
```sh
$ dbsnapper targets
```
### target
List all snapshots for a target

Usage:
```sh
$ dbsnapper target <target_name>
```

### config init
Create and initialize the default configuration. Configuration file is stored in `~/.config/dbsnapper/dbsnapper.yml`

Usage:
```sh
$ dbsnapper config init
```