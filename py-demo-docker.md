---
title: py-demo-docker
date: 2018-09-18 13:45
categories: demo
tags: 
- py-demo-docker
---


``` python
import docker
import socket

def is_running_container(cid):
    try:
        client = docker.from_env()
        c=client.containers.get(cid)
        if c is not None and c.status=='running':
            return True
    except:
        client.close()

    return False

    #c.attrs['Config']['Image']

    #for c in client.containers.list(all=True):
    #    print c.image.tags[0]

def portIsOpen(port):
    isOpen=False
    try:
        s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        result = s.connect_ex(('172.18.23.234',port))
        print result
        isOpen=(result == 0)
    except:
        time.sleep(1)
    finally:
        #s.shutdown(socket.SHUT_RDWR)
        s.close()

    return isOpen

#print portIsOpen(10000)
#print portIsOpen(10879)


#######################################################################

from flask import Flask, jsonify, g
from flask_httpauth import HTTPTokenAuth
import cid_util

app = Flask(__name__)
auth = HTTPTokenAuth(scheme='Bearer')

tokens = {
    "************************": "rock"
}

@auth.verify_token
def verify_token(token):
    g.user = None
    if token in tokens:
        g.user = tokens[token]
        return True
    return False

@app.route('/')
@auth.login_required
def index():
    return "Hello, %s!" % g.user


@app.route('/run_info/<cid>', methods=['GET'])
@auth.login_required
def get_run_info(cid):
    if cid_util.is_running_container(cid):
        hsf_port=int(cid.replace('C',"1"))
        shut_port=int(cid.replace('C',"2"))
        rest_port=int(cid.replace('C',"3"))
        jpda_port=int(cid.replace('C',"4"))

        isHsfPortOK=cid_util.portIsOpen(hsf_port)
        return jsonify({'data': 'Hello, %s, cid %s, hsf_port: %s, isHsfPortOK: %s !' % (g.user,cid,hsf_port,isHsfPortOK)})

    return jsonify({'data': 'Hello, %s, cid %s, status: %s !'%(g.user,cid,False)})

if __name__ == "__main__":
    app.run(host='0.0.0.0')
```
