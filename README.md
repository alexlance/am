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
    AM_FAST=1

    # sometimes you want to exactly match an instance name, instead of a star-pattern-star match
    AM_EXACT=1


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

    # use shell heredocs if you want to run a bunch of commands
    am webservers <<EOF
    sudo service apache stop
    sudo rm -rf /var/log/apache/*
    sudo service apache start
    EOF

    # use env var AM_FAST=1 to run commands using ssh -f, for backgrounded execution
    # this speeds up command execution when targeting a lot of servers
    AM_FAST=1 am pattern-that-matches-many-servers 'run a command, the output is going to be all jumbled up'


Tricks
------

    # search for instances by name fuzzy match
    am name-of-thing

    # or by instance id
    am i-0b0f3c0bea60e1d

    # or by partial ip address
    am 172.22

    # or by partial instance type
    am r4

    # or by AMI id
    am ami-abcd1234

    # non-fuzzy exact name matches only
    AM_EXACT=1 am exact-name-of-thing

    # tab completion works (if you copy am.completion /etc/bash_completion.d/am)
    am dashboard<tab>

    # ssh to one or many boxes sequentially
    am dashboard bash / am i-abdedfsd1234545 bash

    # take a look at all docker processes (only need to type sudo password once, even though multiple machines)
    am dashboard sudo docker ps

    # run a command on a bunch of servers but without waiting for each command to finish (useful when lots of machines)
    AM_FAST=1 am dashboard sudo apt-get install -y tmux

    # if you need to run lots of commands, you can use single quotes or heredocs
    am dashboard 'sudo docker restart $(sudo docker ps -q)'

    # heredoc for lots of ad-hoc commands
    am dashboard <<'EOF'
    ids=$(sudo docker ps -qa)
    sudo docker restart $ids
    more commands go here
    and here
    and here
    EOF

    # easily sort the results by any column
    am dashboard | sort -k8

    # easily parse out a particular field from the results, eg grab only the instance-ids
    am dashboard | awk '{print $2}'

    # then whack them in a loop if you like
    for i in $(am dashboard | awk '{print $2}'); do aws ec2 terminate-instances --instance-ids $i; done

    # interactive
    function bam() {
      ops="$(am $1 | nl -n ln | tee /dev/stderr)"
      read -p "Enter server number: " number
      id=$(echo "$ops" | awk '{print $3}' | sed "${number}q;d")
      am $id bash
    }
