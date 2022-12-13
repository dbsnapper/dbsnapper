# Configuration File

The config file specifies your targets along with system settings such as working directory and secret encryption key.

!!! example "Example configuration file"
    ```yaml
    authtoken: $aes$8fc00dcfef1798be9e77ac90f58b4d9259f059940ec45603cdbf252650cc322e4f347aa7eaf88c9d693efac709e3a5fc9568799eaae6f846f907055444fcb0593bcfd647bd701303d038c99c
    docker:
      images:
        mysql: mysql:8.0-oracle
        postgres: postgres:latest
    secret_key: 1234567890abcdef1234567890abcdef
    targets:
      example_app:
        name: example_app
        src_url: postgresql://postgres:$aes$c953f1e81c98cd6a5bd546ee53a56de2bb906ab6f53477b35c264021d6db6e62e68c6963@10.4.20.22:5432/example_app?sslmode=require
        dst_url: postgresql://postgres:$aes$a6b41b3e2eb78a24f66333d14d2ad0e739c1a84e6dc4cf6b58c30655fb2021190c6c12db@10.4.20.22:5432/example_app_snap?sslmode=require
        query_file: example_app.san.sql
    working_directory: /Users/snappy/.dbsnapper    
    ```

Supported configurations

- `authtoken` (`string` optional) - Auth token used to authenticate with the DBSnapper Cloud. Can also be provided via the `DBSNAPPER_AUTHTOKEN` environment variable..
- `secret_key` (`string[32]`) - 16 byte hexadecimal string (32 characters) that acts as your encryption key. Created on `init`. Can be provided via the `DBSNAPPER_SECRET_KEY` environment variable instead of in configuration file.
- `working_directory` (`string` optional) - Working directory used to store database snapshots. Defaults to `~/.dbsnapper` 
- `docker`
    - `images`
        - `<image_name>: <value>` - specifies the docker image to use for a given database engine. default is `postgres: postgres:latest` see [Database Engines](/database-engines/introduction) for more information.
- `targets` - One or more entries specifying a snapshot workflow
    - `<target_name>` - specifies the name of the target definition
        - `src_url` - connection string of source database
        - `dst_url` - connection string of source database: __THIS WILL BE OVERWRITTEN ON LOAD!__
        - `query_file` - specifies the filename of the query to be used for sanitization. This query should be located in the `working_directory` specified in the configuration.

!!! danger
    A database specified on the `dst_url` will be DROPPED and RECREATED when the `load` command is used

