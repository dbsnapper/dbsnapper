# Quick Start

In this example we will create and restore a database snapshots.
Run the `config init` command to create an example configuration at `~/.config/dbsnapper/dbsnapper.yml`

```sh
dbsnapper config init
```
Add one or more databse `targets` to configuration file. Here we define an `app` target with a `src_url` specifying the source database and a `dst_url` specifying the destination database

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