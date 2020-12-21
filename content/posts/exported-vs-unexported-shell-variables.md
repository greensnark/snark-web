---
title: "Exported vs Unexported Shell Variables"
date: 2020-12-21T16:04:34-05:00
---

Many programmers new to Unix shells are confused by shell variables, and how
they relate to the Unix environment.

<!--more-->

Newcomers are often unclear on the difference between:

```sh
$ A=x
```

and

```sh
$ export A=x
```

I've seen this cause a lot of confusion when people first start writing shell
scripts.

As an example, let's use a simple script (we'll call it `ec2-ip-ls`) that
reports AWS EC2 instance private IP addresses:

```sh
#! /usr/bin/env bash
#
# ec2-ip-ls
#
# Lists the private IP addresses of all EC2 instances
# for the active AWS account

aws ec2 describe-instances \
    | jq -r .Reservations[].Instances[].PrivateIpAddress
```

This script lists the private IP addresses for the AWS EC2 instances in the AWS
account your [AWS CLI is configured to
use](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html)

Now, let's say you'd like to see the EC2 instances in a different AWS region,
perhaps `eu-west-1`. Here's a way of doing that that *may or may not work*!

```sh
$ AWS_DEFAULT_REGION=eu-west-1
$ ec2-ip-ls
```

That sequence will do what you want if you've _exported_ the
`AWS_DEFAULT_REGION` environment variable. If you haven't exported it, you'll
just get the same list of EC2 IP addresses that you get normally.

Exporting the variable fixes the problem:

```sh
$ export AWS_DEFAULT_REGION=eu-west-1
$ ec2-ip-ls
```

So what exactly is the difference between an *un-exported* and an *exported*
shell variable?

### Un-exported shell variables

Un-exported shell variables are visible *only* in the current shell. Any
*subprocesses* of the current shell, such as any programs that your shell runs,
won't see un-exported shell variables:

```sh
$ A=unexported
$ sh -c 'echo $A' # prints nothing: `sh` subprocess can't see A
$ echo $A # "unexported", *this* shell knows that A=unexported
```

An important thing to keep in mind here is that shell variables are expanded by
your shell *before* invoking programs. This is why we use `sh -c '...'`,
because the single-quoted argument to `sh -c` is guaranteed protected from
shell variable expansion.

### Exported shell variables (aka "environment variables")

Exported shell variables are visible in the current shell, and in all
subprocesses, because once exported, a shell variable is an environment
variable.

```sh
$ export A=exported
$ sh -c 'echo $A' # "exported": `sh` can see env A
$ echo $A # "exported", shell also sees A
```

Some programs like `sudo`, deliberately take efforts to not pass on their
environment variables to *their* subprocesses for security, but most other programs
will be able to see all the environment variables your shell has exported.

What makes exported variables trickier to understand is that, once exported, a
variable *stays exported*. Reassigning an exported variable doesn't *un-export*
it:

```sh
$ export A=exported
$ sh -C 'echo $A' # "exported"
$ A=different
$ sh -C 'echo $A' # "different"
```

If you'd like to get rid of an exported environment variable, you can `unset` it:

```sh
$ export A=exported
$ sh -C 'echo $A' # "exported"
$ unset A
$ sh -C 'echo $A' # nothing
$ A=unexported
$ sh -C 'echo $A' # still nothing
```

### One-off exports

It's common to want to run a program with a different environment. In our example,
with `AWS_DEFAULT_REGION`, it'd be a pain to have to export it and then
immediately unset it if we don't want to leave it behind in the environment,
potentially affecting *future* commands:

```sh
$ export AWS_DEFAULT_REGION=eu-west-1
$ ec2-ip-ls
$ unset AWS_DEFAULT_REGION
```

POSIX shells like `bash` and `zsh` offer a convenient syntax for this common
task: prefix the command-line with the environment variable settings, and the
command will be run with the environment variable set, but the environment
variable won't affect further commands, or change the environment in the active
shell:

```sh
$ AWS_DEFAULT_REGION=eu-west-1 ec2-ip-ls
$ echo AWS_DEFAULT_REGION # unchanged
```

This is particularly handy for use with commands like date:

```sh
$ TZ=Europe/Berlin date   # show the time in Berlin
```

### Examining environment and shell variables

The command-line program `env` will print out all environment variables when run with no arguments:

```sh
$ env
LANG=...
PATH=...
USER=...
```

The shell builtin `set` will print out all shell variables *and* all environment variables:

```sh
$ set
LANG=...
PROMPT=...
...
```
