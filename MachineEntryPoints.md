# Machine Entry Points #

To kick off a machine, you simply have to hit a url. E.g., if you have a machine called `EmailBatch` defined in your `fsm.yaml`, you can start the machine simply by hitting the url:

```
http://www.yourdomain.com/fantasm/fsm/EmailBatch/
```

The `/fantasm/` root can be altered in your [fsm.yaml](YamlDescription#Top-level_YAML_attributes.md) file.

You can pass parameters to your machine on the query string. These parameters become part of the machine `context`.

```
http://www.yourdomain.com/fantasm/fsm/EmailBatch/?since=2010-08-10&new-users=True
```

The parameters are available immediately on your initial state:

```
class MyInitialStateAction(object):

  def execute(self, context, obj):
    sinceStr = context['since']
    newUsers = bool(context['new-users'])
    # ...
```

## `GET` vs. `POST` ##

Each machine will persist the method that started it. For example, if you start the machine with a `GET`, all subsequent tasks for that machine (and replicas it creates through continuation) will use `GET` method.

If you start a machine with `POST`, you should not pass initial state arguments on the query string; instead you should pass them as regular POST args.