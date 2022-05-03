# Target Definitions

Add multiple target definitions that represent the source, destination, and sanitization query files

!!! example "Adding configuration for `app` target"

    ```yml
    targets:
      app:
        src_url: postgresql://postgres:postgres@localhost:15432/app?sslmode=disable
        dst_url: postgresql://postgres:postgres@localhost:15432/app_snap?sslmode=disable
        query_file: ""
    ```

You can now list the known targets with the command `dbsnapper targets`
