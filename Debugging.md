# Introduction #

It is often easier to debug a fantasm state machine when it is not running in the taskqueue infrastructure. We support this with an "immediate mode" which runs fsm actions and dispatches events in a tight loop within the webapp request hander.

No additional taskqueue Tasks are enqueued, so some functionality is disabled/restricted - see section "Caveats".

# Details #

To debug a state machine in "immediate mode", simply append the url parameter `__im__=1` to the url.

```
http://www.yourdomain.com/fantasm/fsm/EmailBatch/?__im__=1
```

On success, a JSON response containing the `context` and  `obj` variables is returned. For example, hitting `/fantasm/fsm/SimpleMachine/?__im__=1` in the supplied test application returns

```
{"obj": {"__ms__": [], "__im__": true}, "context": {"unicode": "\u00e8", "foo": "bar", "__step__": 2, "__sa__": 1300298993.2276039}}
```

Any keys surrounded by double underscores are private variables used by fantasm. You should be able to extract your own application specific data easily.

Any exceptions in the state machine will immediately terminate the machine, and the stack trace will be returned in the content of the HTTP response.

# Caveats #

  * fan-out only operates over the first element (continuation() does not enqueue any further tasks).
  * fan-in is not supported.
  * spawn() is not supported.
  * fork() is not supported.
  * the same `obj` is passed through the entire machine.