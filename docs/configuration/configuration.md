# Configuration File

The config file specifies your targets along with system settings such as working directory and secret encryption key.

Supported configurations

- `secret_key` (`string[32]`) - 16 byte hexadecimal string (32 characters) that acts as your encryption key. Created on `init`. Can be provided via `DBSNAPPER_SECRET_KEY` environment variable instead of in configuration file.
- `working_directory` (`string` optional) - Working directory used to store database snapshots. Defaults to `~/.dbsnapper` 
- `docker`
    - `images`
        - `<image_name>` - specifies the docker image for a given database application i.e. `postgres:latest`
- `targets` - One or more entries specifying a snapshot workflow
    - `<target_name>` - specifies the name of the target definition
        - `src_url` - connection string of source database
        - `dst_url` - connection string of source database: __THIS WILL BE OVERWRITTEN ON LOAD!__
        - `query_file` - specifies the filename of the query to be used for sanitization. This query should be located in the `working_directory` specified in the configuration.

!!! danger
    A database specified on the `dst_url` will be DROPPED and RECREATED when the `load` command is used

