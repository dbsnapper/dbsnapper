# Quick Start

In this example we will create and restore a database snapshots.
Run the `config init` command to create an example configuration at `~/.config/dbsnapper/dbsnapper.yml`

```sh
dbsnapper config init
```

!!! example "Configuration file initialized to default values"
    ```yaml
    secret_key: d3d234bc83dd4efe7b7329855ba0acc2
    working_directory: /Users/joescharf/.dbsnapper
    docker:
      images:
        postgres: postgres:latest
    ```

Next, we can check our configuration and required dependencies. This run some checks to verify the configuration file is valid, Docker is installed, and the specified Docker images exist locally.

```sh
dbsnapper config check
```

!!! example "DBSnapper config check output"
    ```sh

    DBSnapper CLI v0.10.1+5a65398a.2022-04-28T14:35:53Z

    Checking DBSnapper Configuration
      ✅ Config file found and loaded
      ✅ Docker client connected
      ✅ docker.images set in config file
        ✅ Found Docker image:  postgres:latest

      ✅ Configuration OK
    ```


Add one or more databse `targets` to configuration file. Here we define an `app` target with a `src_url` specifying the source database and a `dst_url` specifying the destination database

!!! example "Defining an app target definition"
    ```yaml
    targets:
      app:
        src_url: postgresql://postgres:postgres@localhost:15432/app?sslmode=disable
        dst_url: postgresql://postgres:postgres@localhost:15432/app_snap?sslmode=disable
    ```

!!! danger
    A database specified on the `dst_url` will be DROPPED and RECREATED when the `load` command is used

Now that we have a target defined, we can list all targets with: 

```sh
dbsnapper targets
```

This command will also check the size and connectivity status for each target defined in the configuration file.

Now we're ready to create our first snapshot of the `app` target which can be done with the `build` command:

```sh
dbsnapper build app
```
 
This will connect to the database using the native dump utility and will create a dump of all data in the database.

When this is finished you can list all snapshots for the `app` target with:

```sh
dbsnapper target app
```

Finally, if a `dst_url` is defined in the target definition, you can load a snapshot to the destination using the index on the snapshot list.

```sh
dbsnapper load app 0
```

!!! danger
    Remember, the database specifid on `dst_url` will be DROPPED and a new empty database with the same name will be CREATED prior to loading the data from the snapshot!