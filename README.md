## What is Inotify?

> GitHub repository: <https://github.com/inotify-tools/inotify-tools>

From the linux man pages (<https://man7.org/linux/man-pages/man7/inotify.7.html>):

*The inotify API provides a mechanism for monitoring filesystem events. Inotify can be used to monitor individual files, or to monitor directories. When a directory is monitored, inotify will return events for the directory itself, and for files inside the directory.*

Two command-line tools are distributed as part of the `inotify-tools` package (`inotifywait`, `inotifywatch`) and allows to interact with the Inotify API.

## How to use this image

`docker-inotify` provides an Alpine-based image that contains the `inotify-tools` package. It also includes a lightweight `inotifywait.sh` script that can watch files and/or directories and send events to a user-defined script. The script can be entirely configured through environment variables.

The main use-case for this image is to provide an easy way to trigger an action based on a configuration file change. All you need to do is mount a volume in the sidecar to be monitored, and provide a script to trigger when an event is received.

### How it works

An `inotifywait` process watches INOTIFY_TARGET, and runs INOTIFY_SCRIPT with the triggered event data as arguments.

When using the default configuration values, the script will receive:

| argv | name                                    |
| ---- | --------------------------------------- |
| `$1` | timestamp                               |
| `$2` | watched file/directory path             |
| `$3` | event names                             |
| `$4` | filenames (if a directory is monitored) |

#### Arguments examples

> Watched events: `modify delete delete_self`

A watched file being modified/deleted

```bash
22:48:36 /test MODIFY
22:48:36 /test DELETE_SELF
```

A file being modified/deleted in a watched directory

```bash
22:48:36 /test/ MODIFY a_file
22:48:36 /test/ DELETE a_file
```

The following section describes how to configure the watch process.

### Environment variables

#### Global

| Variable       | Required | Default value | Description                                |
| -------------- | -------- | ------------- | ------------------------------------------ |
| INOTIFY_TARGET | true     | empty         | The file or directory to watch for events  |
| INOTIFY_SCRIPT | true     | empty         | The script to run whenever an event occurs |
| INOTIFY_QUIET  | false    | true          | If set, suppress log messages              |

#### Watch configuration

> The default values should be fine in most cases.</br>
> See [inotify-tools](https://github.com/inotify-tools/inotify-tools) for more details about `inotifywait` available flags.
>
> Booleans can be set to any value to be considered true.

| Variable               | Default value               | Description                                                         |
| ---------------------- | --------------------------- | ------------------------------------------------------------------- |
| INOTIFY_CFG_CSV        | false                       | (bool) Output events using CSV format                               |
| INOTIFY_CFG_EVENTS     | `modify delete delete_self` | Space-separated list of events to watch                             |
| INOTIFY_CFG_EXCLUDE    | -                           | Exclude a subset of files using a POSIX regex pattern               |
| INOTIFY_CFG_EXCLUDEI   | -                           | Same as `INOTIFY_CFG_EXCLUDE` but case insensitive                  |
| INOTIFY_CFG_FORMAT     | `%T %w %e %f`               | The formatting pattern used to emit events                          |
| INOTIFY_CFG_INCLUDE    | -                           | Include a subset of files using a POSIX regex pattern               |
| INOTIFY_CFG_INCLUDEI   | -                           | Same as `INOTIFY_CFG_INCLUDE` but case insensitive                  |
| INOTIFY_CFG_NO_NEWLINE | false                       | (bool) Remove newline from the emmited event                        |
| INOTIFY_CFG_QUIET      | true                        | (bool) Suppress inotifywait logging                                 |
| INOTIFY_CFG_RECURSIVE  | false                       | (bool) Watch all subdirectories with unlimited depth                |
| INOTIFY_CFG_TIMEFMT    | `%H:%M:%S`                  | The strftime-compatible pattern used to display %T in emitted event |
| INOTIFY_CFG_TIMEOUT    | -                           | Timeout and re-setup watchers after X seconds of no event received  |

### Example

#### Docker-compose

```yaml
version: "3"
services:
  inotify:
    image:  'lihanov/inotify:latest'
    restart: unless-stopped
    container_name:   'inotify'
    volumes:
      - /data:/data
    environment:
      INOTIFY_TARGET: "/data/nginx"
      INOTIFY_SCRIPT: "/data/script.sh"
      INOTIFY_CFG_RECURSIVE: "true"
```
