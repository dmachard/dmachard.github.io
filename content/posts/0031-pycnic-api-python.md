---
title: "Exemple de développement d'une API REST avec pycnic"
summary: "RESTful api"
date: 2019-03-03T00:00:00+01:00
draft: false
tags: ['python', 'api']
---

# Exemple de développement d'une API REST avec pycnic

Exemple de développement d'une API avec la librairie Python **[pycnic](http://pycnic.nullism.com/)**

Installation de la librarie

```python
pip install pycnic
```

Exemple simple d'une API serveur:

```python
from pycnic.core import WSGI, Handler
from pycnic.errors import HTTP_401, HTTP_400, HTTP_500, HTTP_403, HTTP_404

from wsgiref.simple_server import make_server, WSGIServer
from SocketServer import ThreadingMixIn


class HandlerCors(Handler):
    def options(self):
        return {}

class HelloWorld(HandlerCors):
    def post(self):
	    return { "msg": "success" }

class app(WSGI):
    headers = [
                ("Access-Control-Allow-Origin", "*"),
                ("Access-Control-Allow-Method", "*"),
                ("Access-Control-Allow-Headers", "content-type, authorization"),
            ]
    routes = [
        ('/helloworld', HelloWorld())
    ]

class ThreadingWSGIServer(ThreadingMixIn, WSGIServer):
    daemon_threads = True

class Server:
    def __init__(self, wsgi_app, listen='127.0.0.1', port=8080):
        self.wsgi_app = wsgi_app
        self.listen = listen
        self.port = port
        self.server = make_server(self.listen, self.port, self.wsgi_app,
                                  ThreadingWSGIServer)

    def serve_forever(self):
        self.server.serve_forever()

httpd = Server(app, '0.0.0.0', 8080)
httpd.serve_forever()
```
