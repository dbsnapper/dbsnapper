# Secrets Encryption

The configuration file encryption routines are run on every command invocation. Any new unencrypted sensitive data will be automatically encrypted. Our `app` target defined above will be encrypted similar to the following:

!!! example "`app` target after encryption"

    ```yml  
    targets:
      app:
        src_url: postgresql://postgres:$aes$bf7b67ddbf88c9becd6c208e40ce127cc2164afe74f6c7347a6bce367e1a582b09ff88c6@localhost:15432/app?sslmode=disable
        dst_url: postgresql://postgres:$aes$f24821b81b7e96942450a6735c4fbdb7e2ad23981bc61be7d3a0280679e0411eae512bd7@localhost:15432/app_snap?sslmode=disable
        snap: ""
        query: ""
    ```