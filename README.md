am
==

Print a list of AWS EC2 instances that match a name, and run commands remotely on those instances.

![am demo](http://alexlance.com/am_for_github.gif)


Prerequisites
-------------

1. aws-cli
2. ssh access to the relevant instances


Installation
------------

Get the script into your $PATH:

    git clone git@github.com:alexlance/am am.repo
    ln -s am.repo/am /usr/bin/am   # for sequential command execution
    ln -s am.repo/am /usr/bin/amp  # for parallel command execution (faster, but non-sequential output)

If you'd like tab-completion for your instance names, run this as a cronjob every half hour or so:

    */30 * * * * aws ec2 describe-tags --filters "Name=resource-type,Values=instance" --query "join(' ',sort(Tags[?Key=='Name'].Value))" | tr -d '"' > /tmp/.instances

Then copy the tab-completion script into the proper place:

    cp am.completion /etc/bash_completion.d/am


Usage
-----

    # usage:
    am <instancename> [command]

    # print a list of servers matching this name
    am webservers

    # run a command on a bunch of servers that match this name eg
    am webservers 'ps -ef | grep apache'

    # run a shell on each in turn - basically ssh-ing to the server
    am webservers bash

    # any mentions of sudo in the command will be replaced with 'echo "$password" | sudo -S'
    am webservers 'sudo docker start $(sudo docker ps -qa -f Name=stuff)'

    # amp runs commands using ssh -f, for backgrounded execution. Much faster when targeting a lot of servers
    amp lots-and-lots-of-servers 'run a command, but the output is going to be all jumbled up'
