---
title: "attach"
description: "The attach command description and usage"
keywords: "attach, running, container"
---

# attach

```markdown
Usage: docker attach [OPTIONS] CONTAINER

Attach local standard input, output, and error streams to a running container

Aliases:
  docker container attach, docker attach

Options:
      --detach-keys string   Override the key sequence for detaching a container
      --help                 Print usage
      --no-stdin             Do not attach STDIN
      --sig-proxy            Proxy all received signals to the process (default true)
```

## Description

Use `docker attach` to attach your terminal's standard input, output, and error
(or any combination of the three) to a running container using the container's
ID or name. This allows you to view its ongoing output or to control it
interactively, as though the commands were running directly in your terminal.

> **Note:**
> The `attach` command will display the output of the `ENTRYPOINT/CMD` process.  This
> can appear as if the attach command is hung when in fact the process may simply
> not be interacting with the terminal at that time.

You can attach to the same contained process multiple times simultaneously,
from different sessions on the Docker host.

To stop a container, use `CTRL-c`. This key sequence sends `SIGKILL` to the
container. If `--sig-proxy` is true (the default),`CTRL-c` sends a `SIGINT` to
the container. If the container was run with `-i` and `-t`, you can detach from
a container and leave it running using the `CTRL-p CTRL-q` key sequence.

> **Note:**
> A process running as PID 1 inside a container is treated specially by
> Linux: it ignores any signal with the default action. So, the process
> will not terminate on `SIGINT` or `SIGTERM` unless it is coded to do
> so.

It is forbidden to redirect the standard input of a `docker attach` command
while attaching to a tty-enabled container (i.e.: launched with `-t`).

While a client is connected to container's stdio using `docker attach`, Docker
uses a ~1MB memory buffer to maximize the throughput of the application. If
this buffer is filled, the speed of the API connection will start to have an
effect on the process output writing speed. This is similar to other
applications like SSH. Because of this, it is not recommended to run
performance critical applications that generate a lot of output in the
foreground over a slow client connection. Instead, users should use the
`docker logs` command to get access to the logs.

### Override the detach sequence

If you want, you can configure an override the Docker key sequence for detach.
This is useful if the Docker default sequence conflicts with key sequence you
use for other applications. There are two ways to define your own detach key
sequence, as a per-container override or as a configuration property on  your
entire configuration.

To override the sequence for an individual container, use the
`--detach-keys="<sequence>"` flag with the `docker attach` command. The format of
the `<sequence>` is either a letter [a-Z], or the `ctrl-` combined with any of
the following:

* `a-z` (a single lowercase alpha character )
* `@` (at sign)
* `[` (left bracket)
* `\\` (two backward slashes)
*  `_` (underscore)
* `^` (caret)

These `a`, `ctrl-a`, `X`, or `ctrl-\\` values are all examples of valid key
sequences. To configure a different configuration default key sequence for all
containers, see [**Configuration file** section](cli.md#configuration-files).

## Examples

### Attach to and detach from a running container

```console
$ docker run -d --name topdemo ubuntu /usr/bin/top -b

$ docker attach topdemo

top - 02:05:52 up  3:05,  0 users,  load average: 0.01, 0.02, 0.05
Tasks:   1 total,   1 running,   0 sleeping,   0 stopped,   0 zombie
Cpu(s):  0.1%us,  0.2%sy,  0.0%ni, 99.7%id,  0.0%wa,  0.0%hi,  0.0%si,  0.0%st
Mem:    373572k total,   355560k used,    18012k free,    27872k buffers
Swap:   786428k total,        0k used,   786428k free,   221740k cached

PID USER      PR  NI  VIRT  RES  SHR S %CPU %MEM    TIME+  COMMAND
 1 root      20   0 17200 1116  912 R    0  0.3   0:00.03 top

 top - 02:05:55 up  3:05,  0 users,  load average: 0.01, 0.02, 0.05
 Tasks:   1 total,   1 running,   0 sleeping,   0 stopped,   0 zombie
 Cpu(s):  0.0%us,  0.2%sy,  0.0%ni, 99.8%id,  0.0%wa,  0.0%hi,  0.0%si,  0.0%st
 Mem:    373572k total,   355244k used,    18328k free,    27872k buffers
 Swap:   786428k total,        0k used,   786428k free,   221776k cached

   PID USER      PR  NI  VIRT  RES  SHR S %CPU %MEM    TIME+  COMMAND
       1 root      20   0 17208 1144  932 R    0  0.3   0:00.03 top


 top - 02:05:58 up  3:06,  0 users,  load average: 0.01, 0.02, 0.05
 Tasks:   1 total,   1 running,   0 sleeping,   0 stopped,   0 zombie
 Cpu(s):  0.2%us,  0.3%sy,  0.0%ni, 99.5%id,  0.0%wa,  0.0%hi,  0.0%si,  0.0%st
 Mem:    373572k total,   355780k used,    17792k free,    27880k buffers
 Swap:   786428k total,        0k used,   786428k free,   221776k cached

 PID USER      PR  NI  VIRT  RES  SHR S %CPU %MEM    TIME+  COMMAND
      1 root      20   0 17208 1144  932 R    0  0.3   0:00.03 top
^C$

$ echo $?
0
$ docker ps -a | grep topdemo

7998ac8581f9        ubuntu:14.04        "/usr/bin/top -b"   38 seconds ago      Exited (0) 21 seconds ago                          topdemo
```

### Get the exit code of the container's command

And in this second example, you can see the exit code returned by the `bash`
process is returned by the `docker attach` command to its caller too:

```console
$ docker run --name test -d -it debian
275c44472aebd77c926d4527885bb09f2f6db21d878c75f0a1c212c03d3bcfab

$ docker attach test
root@f38c87f2a42d:/# exit 13

exit

$ echo $?
13

$ docker ps -a | grep test

275c44472aeb        debian:7            "/bin/bash"         26 seconds ago      Exited (13) 17 seconds ago                         test
```
