# Large Messages Shovelling

Shovel large messages in a low-bandwidth network (e.g. 50 MB messages, network 750 kbps).

Original thread: https://groups.google.com/forum/#!topic/rabbitmq-users/rv0vj30dnY8

## Pre-requisites

VirtualBox and Vagrant (2.x+).

## Setup

```
$ git clone git@github.com:rabbitmq/shovel-large-messages.git
$ cd shovel-large-messages
$ vagrant up
$ vagrant ssh node1
node1 $ /vagrant/install.sh
node1 $ exit
$ vagrant ssh node2
node2 $ /vagrant/install.sh
node2 $ exit
$ vagrant ssh node1
node1 $ /vagrant/declare-shovel.sh
```

Management plugin is available on http://192.168.33.11:15672 and http://192.168.33.12:15672 (user / password is admin / admin).

## Sending messages

The `send-messages.groovy` script can publish large messages periodically. Launch it from `node1` with `groovy /vagrant/send-messages.groovy`.

## Shovelling

The default configured shovel is on `node1` and shovels from `node1#shovel-source` to `node2#shovel-destination`. The `send-messages.groovy` script can be used to send large messages to `node1#shovel-source` to start the shovelling.

## Useful commands

```
# same as above but with hard throughput limit
sudo tc qdisc add dev eth1 root tbf rate 750Kbit burst 32kbit latency 200ms peakrate 1mbit minburst 1520
# list default queuing disciplines
sudo tc qdisc show
# delete limiting disciplines
sudo tc qdisc del dev eth1 root
# combine network latency, reordering, and loss with throughput limit
sudo tc qdisc add dev eth1 root handle 1: netem delay 10ms reorder 25% 50% loss 0.2%
sudo tc qdisc add dev eth1 parent 1: handle 2: tbf rate 750Kbit burst 32kbit latency 200ms peakrate 1mbit minburst 1520

groovy /vagrant/send-messages.groovy
tail -n 50 -f /var/log/rabbitmq/rabbit@vm-node1.log
tail -n 50 -f /var/log/rabbitmq/rabbit@vm-node2.log
```
