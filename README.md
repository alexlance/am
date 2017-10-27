am
==

Print a list of AWS EC2 instances by their instance name - and/or run commands on them.

Prerequisites
-------------

1. a working aws-cli setup
2. ssh access to the relevant instances

Installation
------------

    git clone git@github.com:alexlance/am am.repo
    ln -s am.repo/am /usr/bin/am

Usage
-----

    # usage:
    am <hostname> [command]

    # print a list of servers matching this name
    am server-name

    # run a command on a bunch of servers that match this name eg
    am webservers 'ps -ef | grep apache'

    # run a shell
    am some-host bash

    # any mentions of sudo in the command will be replaced with 'echo "$password" | sudo -S'
    am server1 'sudo docker start $(sudo docker ps -qa -f Name=stuff)'
