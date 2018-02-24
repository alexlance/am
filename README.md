am
==

Print a list of AWS EC2 instances that match a name, and run commands remotely on those instances.

![am demo](http://alexlance.com/am_for_github.gif)


Prerequisites
-------------

1. aws-cli installed and your ~/.aws config setup
2. ssh access to the relevant instances you want to administrate


Installation
------------

Get the script into your $PATH:

    git clone git@github.com:alexlance/am
    ln -s am/am /usr/bin/am

If you'd like tab-completion for your instance names:

    # copy the tab-completion script into the proper place:
    cp am.completion /etc/bash_completion.d/am

    # add a once per hour cronjob
    0 * * * * (for region in ${AWS_REGIONS:-$(aws configure get region)}; do aws --region $region ec2 describe-tags --filters "Name=resource-type,Values=instance" --query "join(' ',sort(Tags[?Key=='Name'].Value))" | tr -d '"'; done) > /tmp/.instances

These environment variables may be useful:

    # a list of AWS regions to target
    AWS_REGIONS="ap-southeast-2 us-west-2"

    # to skip typing out your sudo password
    AM_SUDO_PASSWORD="tops3cret"

    # use `ssh -f` for faster command execution when there are many servers
    FAST=1

Usage
-----

    # usage:
    am <instancename> [command]

    # print a list of servers matching this name
    am webservers

    # run a command on a bunch of servers that match this name
    am webservers 'ps -ef | grep apache'

    # run a shell on each in turn - basically ssh-ing to the server
    am webservers bash

    # any mentions of sudo in the command will be replaced with 'echo "$password" | sudo -S'
    # so you only need to type your sudo password once, even across multiple servers
    am webservers 'sudo docker start $(sudo docker ps -qa -f Name=stuff)'

    # use env var FAST=1 to run commands using ssh -f, for backgrounded execution
    # this speeds up command execution when targeting a lot of servers
    FAST=1 am pattern-that-matches-many-servers 'run a command, the output is going to be all jumbled up'
