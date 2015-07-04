# Introduction #

**Fantasm has not been thoroughly tested in Python 2.7**, but it seems to work fine. You can look at Google's documentation for more details about the Python 2.7 environment - http://code.google.com/appengine/docs/python/gettingstartedpython27/

Your` app.yaml` will look something like the following examples:

# CGI #

```
application: application-name
version: version
runtime: python27
api_version: 1
threadsafe: no

handlers:

- url: /fantasm/.*
  script: fantasm/main.py
  login: admin
```

# WSGI #

```
application: application-name
version: version
runtime: python27
api_version: 1
threadsafe: no

handlers:

- url: /fantasm/.*
  script: fantasm.main.APP
  login: admin
```

# Threadsafe #

```
application: application-name
version: version
runtime: python27
api_version: 1
threadsafe: yes

handlers:

- url: /fantasm/.*
  script: fantasm.main.APP
  login: admin
```