## rclone
```bash
rclone config # enter an interactive configuration session
rclone listremotes # list all remotes in config file
```

```bash
rclone check source:path dest:path --size-only # Compare the size of all files in the source and destination, skip hash check
rclone size remote:path --json # print the total size and number of object in JSON format
rclone md5sum remote:path # generate an md5sum file for all the objects in the path
```

```bash
rclone ls remote:path # list size and path of all objects
rclone lsd remote:path # list all directories
rclone lsjson remote:path # list all objects and directories in JSON format
```

```bash
rclone copy source:path dest:path -P # Download files from source to destination skipping files already copied, show progress
rclone sync source:path dest:path # Sync source and destination, modify destination only
```
