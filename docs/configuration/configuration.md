# Configuration File

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

!!! example "Adding configuration for `app` target"

    ```yml
    targets:
      app:
        src_url: postgresql://postgres:postgres@localhost:15432/app?sslmode=disable
        dst_url: postgresql://postgres:postgres@localhost:15432/app_snap?sslmode=disable
    ```

You can now list the known targets with the command `dbsnapper targets`
