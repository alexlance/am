am
==

Print a list of EC2 instances by their instance name - and/or run commands on them.

Prerequisites
-------------

1. A working aws-cli setup
2. SSH access to the relevant instances

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
    am webservers ps -ef

    # remember to escape pipes and ampersands etc if you want them to run remotely
    am webservers ps -ef \| grep apache

    # run privileged commands too using sudo - only type sudo password once, even though multiple hosts!
    am workers sudo docker ps

    # run a shell
    am some-host bash

    # print out list of IP addresses of servers when output not going to terminal
    for i in $(am cluster); do ping -c1 $i; done

    # any mentions of sudo in the command will be replaced with 'echo "$password" | sudo -S'
    am server1 'sudo docker start $(sudo docker ps -qa -f Name=stuff)'
