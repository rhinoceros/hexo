---
title: python zk
date: 2018-04-26 21:06
categories: python
tags: 
- zookeeper
- KazooClient
- argparse
---


``` python
from kazoo.client import KazooClient
import argparse
import logging
logging.basicConfig()
def zk_fun():
    zk = KazooClient(hosts='127.0.0.1:2181')
    zk.start()
    # Ensure a path, create if necessary
    zk.ensure_path("/aliinte/CID_PATH")
    # Create a node with data
    if zk.exists("/aliinte/CID_PATH/test"):
        zk.set("/aliinte/CID_PATH/test", b"test value 2")
    else:
        zk.create("/aliinte/CID_PATH/test", b"a test value")
    # Determine if a node exists
    if zk.exists("/aliinte/CID_PATH/foo"):
        print "the node exist"
    # Print the version of a node and its data
    foo, stat = zk.get("/aliinte/CID_PATH/foo")
    print("Version: %s, data: %s" % (stat.version, foo.decode("utf-8")))
    # List the children
    children = zk.get_children("/aliinte/CID_PATH")
    print("There are %s children with names %s" % (len(children), children))
    for child in children:
        data, stat = zk.get("/aliinte/CID_PATH/"+child)
        print("Version: %s, data: %s" % (stat.version, data.decode("utf-8")))
    zk.stop()
def zk_get_data(zkpath):
    zk = KazooClient(hosts='127.0.0.1:2181')
    zk.start()
    data = ""
    if zk.exists(zkpath):
        data, stat = zk.get(zkpath)
        data = data.decode("utf-8")
    zk.stop()
    return data
def zk_set_data(zkpath,zkdata):
    zk = KazooClient(hosts='127.0.0.1:2181')
    zk.start()
    zk.ensure_path(zkpath)
    zk.set(zkpath,zkdata)
    data, stat = zk.get(zkpath)
    data = data.decode("utf-8")
    print 'data:' + data
    zk.stop()
 
def main():
    parser = argparse.ArgumentParser(description='zk config tool')
    parent_parser = argparse.ArgumentParser(add_help=False)
    parent_parser.add_argument('-p', required=True,  action="store", dest="zkpath")
    subparsers = parser.add_subparsers(title="commands", dest="command", help='commands')
    get_parser = subparsers.add_parser('get', parents=[parent_parser], help='get data')
    set_parser = subparsers.add_parser('set', parents=[parent_parser], help='set data')
    set_parser.add_argument('-d', action="store", dest="zkdata")
    args = parser.parse_args()
    print args
    if args.command == "get" and args.zkpath:
        data = zk_get_data(args.zkpath)
        print "data:"+data
    if args.command == "set" and args.zkdata:
        zk_set_data(args.zkpath, args.zkdata)
```
