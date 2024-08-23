---
title: How to troubleshooting k8s pods and containers
date: 2023-09-16 17:00:06
categories:
 - Kubernetes
tags:
 - Docker
 - Kubernetes
---
# How to troubleshooting k8s pods and containers
## hold on program by `tail -f /dev/null`
The command "tail -f /dev/null" is a Unix/Linux command that is often used as a placeholder or a trick to prevent a command or a script from exiting or terminating.

The tail command is typically used to display the last few lines of a file. When used with the -f option, it enables "follow" mode, where tail will continuously display new lines appended to the file in real-time. However, when tail -f is used with /dev/null, which is a special file that discards all data written to it, the command essentially does nothing. /dev/null is commonly used as a bit bucket or a sink for unwanted output.

So, in essence, running the command "tail -f /dev/null" will cause the tail command to start and keep running indefinitely, but it won't display any output because /dev/null discards everything written to it. It's often used as a placeholder command to prevent scripts or programs from exiting or to keep a terminal session open without any active output.

### How to use
#### Dockerfile

```dockerfile
command: ["tail", "-f", "/dev/null"]
```

#### Kubernetes

```yaml
containers:
- name: my-container
  image: my-image:tag
  command: ["tail", "-f", "/dev/null"]
```
> NOTE
  Keep in mind that while ENTRYPOINT defines the default command in dockerfile, it can be overridden by providing commands in the Kubernetes pod. This means that if you specify a command  `tail -f /dev/null` within the pod, the container within it will be held up by the provided command, and any commands specified in the ENTRYPOINT won't be executed.
