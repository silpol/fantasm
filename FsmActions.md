# FSM Action Interfaces #

There are four possible code entry points where Fantasm will call your code: state `entry`, state `action`, state `exit`, and transition `action`. Only state `action` is required and a return value is only expected from a non-final state `action`.

The interface for each of these entry points is as follows:

```
class MyClass(object):

  def execute(self, context, obj):
    # your code
    # return a string event value if this is an action for a non-final state
```

## `context` ##

`context` is the machine context that is passed between states and transitions. It is a dictionary to which you are free to add and remove values.

```
    context['report-key-name'] = report.key().name()
```

Keep in mind that all values are serialized to a query string so use objects that have good string representations.

In later states and transitions, the context values are available to you.

```
    report_key_name = context['report-key-name'] # This will be a string in this example - see note on context types below.
```

See [context types](YamlDescription#Context_types.md) to configure Fantasm to automatically cast these values to the appropriate datatypes.

**IMPORTANT** The entire context for a machine is carried in the taskqueue task itself. Because of this there are limitations around the overall size of the context. taskqueue's overall limit is 10KB, however, note that Fantasm also manages some internal context information under the hood. It is very important to keep your contexts concise; this may result in storing information to datastore and passing only, e.g., keys on the context.

## `obj` ##

`obj` is a generic dictionary object that is scoped to the life of the request (i.e., a single taskqueue task execution). It is useful to pass information between the state exit, transition action, state entry, and state action of a given request. The event lifecycle is as follows:


---

**task dequeued** -> state1 exit -> transition action -> state2 entry -> state2 action -> **task queued**

---


The `obj` object can be used to pass information between the events above. Items on `obj` are not part of the machine context and thus will not be serialized.

```
    obj['my_temp_results'] = my_large_dataset
```

# Continuation Interface #

If your state is demarcated as a `continuation`, in addition to your `execute()` method, you are also required to implement a `continuation()` method.

```
class MyContinutationStateAction(object):

  def continuation(self, context, obj, token=None):
    # using the optional token, you are responsible to return the next token
    # returning None means the continuation is complete

  def execute(self, context, obj):
    # your action code
```

The event cycle is slightly modified for a continuation state:


---

**task dequeued** -> state1 exit -> transition action -> state2 entry -> _continuation_ -> state2 action -> **task queued**

---


For each continuation token returned, a **replica machine is forked** and a task is queued to continue it. In this way, you can scale-out your machine to operate over a large number of servers.

A very typical use case is to build a continuation that iterates over the results of a datastore query. Fantasm provides a helper class to bundle some of the boilerplate for you, with a slightly altered continuation interface.

```
from fantasm.action import DatastoreContinuationFSMAction

class MyDatastoreContinuationStateAction(DatastoreContinuationFSMAction):

  def getQuery(self, context, obj):
    # return a datastore query object
    return Subscribers.all().filter('new =', True)

  def getBatchSize(self, context, obj):
    # return an integer specifying how many results should be fetched per machine replica
    return 1 # a new machine for _each_ query result

  def execute(self, context, obj):
    # the fetched results will be on obj['results'] (obj['result'] contains the single/first value)
    # NOTE: because of the manner in which datastore queries operate, obj['results'] and obj['result']
    # may be empty; your code needs to consider this case.
    subscriber = obj['result']
    if subscriber: # the datastore query may return no results
      subscriber.send_email()
```

## Notes on using NDB ##

Fantasm provides a specialized helper class to allow the use of NDB queries in continuations: `NDBDatastoreContinuationFSMAction`. The interface is very similar to the above `DatastoreContinuationFSMAction` in that you need to implement `getQuery()` and `execute()`.

Additionally, because NDB specifies `keys_only` queries differently, you can also override `getKeysOnly()` to specify the appropriate value; the default is `False`. NDB uses a default RPC timeout of 5 seconds when performing querying; you can override this by overriding the method `getDeadline()`.

```
from fantasm.action import NDBDatastoreContinuationFSMAction

class MyDatastoreContinuationStateAction(NDBDatastoreContinuationFSMAction):

  def getQuery(self, context, obj):
    # return an NDB query object
    return NDBSubscribers.query(NDBSubscribers.new == True)

  def getKeysOnly(self, context, obj):
    return False

  def getDeadline(self, context, obj):
    return 10

  def getBatchSize(self, context, obj):
    # return an integer specifying how many results should be fetched per machine replica
    return 1 # a new machine for _each_ query result

  def execute(self, context, obj):
    # the fetched results will be on obj['results'] (obj['result'] contains the single/first value)
    # NOTE: because of the manner in which datastore queries operate, obj['results'] and obj['result']
    # may be empty; your code needs to consider this case.
    subscriber = obj['result']
    if subscriber: # the datastore query may return no results
      subscriber.send_email()
```



---


The finite state implementation used by Fantasm is inspired by J. van Gurp, J. Bosch, "On the Implementation of Finite State Machines", in Proceedings of the 3rd Annual IASTED International Conference Software Engineering and Applications, IASTED/Acta Press, Anaheim, CA, pp. 172-178, 1999. (http://www.jillesvangurp.com/static/fsm-sea99.pdf)