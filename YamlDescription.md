# Yaml Description #

To use Fantasm, you create an `fsm.yaml` file in your App Engine application (e.g., alongside `app.yaml`). This file contains all the configuration for Fantasm and may include multiple machine definitions.

## Example `fsm.yaml` ##

The following represents a simple, three-state machine. From initial `StartingState`, we may transition to `FinalState` or `FailureState`. `FinalState` is a final state and no events are expected from the action. The action for `FailureState` is also a final state, but it may also emit a `recover` event that drives the machine to `FinalState`.

The YAML file references classes in your code that execute the various actions of the state machine. Your code is responsible to emit the events to drive the machine to the next state. You are able to execute any Python code in your classes.

<table>
<tr>
<td>
<blockquote><code>fsm.yaml</code>
</td>
<td>
<code>simple_example.py</code>
</td>
</tr>
<tr valign='top'>
<td>
<pre><code>state_machines:<br>
<br>
- name: SimpleExample<br>
  namespace: simple_example<br>
<br>
  states:<br>
<br>
  - name: StartingState<br>
    initial: True<br>
    action: StartingStateClass<br>
    transitions:<br>
    - event: ok<br>
      to:    FinalState<br>
    - event: failure<br>
      to:    FailureState<br>
<br>
  - name: FinalState<br>
    final: True<br>
    action: FinalStateClass<br>
<br>
  - name: FailureState<br>
    final: True<br>
    action: FailureStateClass<br>
    transitions:<br>
    - event: RECOVER_CONST # store event names in code, if preferred<br>
      to:    FinalState<br>
<br>
- name: AnotherMachine<br>
  ...<br>
</code></pre>
</td>
<td>
<pre><code>RECOVER_CONST = 'recover'<br>
<br>
class StartingStateClass(object):<br>
  def execute(self, context, obj):<br>
    # your code here, returning the next event to dispatch, e.g., <br>
    if somethingGood:<br>
      return 'ok'<br>
    else:<br>
      return 'failure'<br>
<br>
class FinalStateClass(object):<br>
  def execute(self, context, obj):<br>
    # your final state code here, no event to return<br>
<br>
class FailureStateClass(object):<br>
  def execute(self, context, obj):<br>
    # your code here, optionally returning an event, e.g.,<br>
    if ableToRecover:<br>
      return RECOVER_CONST<br>
</code></pre>
</td>
</tr>
</table></blockquote>

Your code is contained in the `action` classes. Your class is responsible to return the appropriate event to drive the machine to the next state. More information on the specific code interfaces can be found on [FSM Actions](FsmActions.md).

## Top-level YAML attributes ##

|`root_url`|The root url where Fantasm is mounted. Must match the setting in [app.yaml](AppYaml.md). Default: `/fantasm/`.|
|:---------|:-------------------------------------------------------------------------------------------------------------|
|`enable_capabilities_check`|A boolean indicating if Fantasm should check to see if the required App Engine capabilities are currently operating before allowing the state to execute. Set this to `False` for a small performance boost. Default: `True`|
|`state_machines`|A list of state machines. See below for state machine attributes.                                             |

## State-machine attributes ##

|`name`|The unique name of the state machine. Can only contain letters, numbers, and dashes.|
|:-----|:-----------------------------------------------------------------------------------|
|`namespace`|The default namespace where your code is located. Can be overridden on states or transitions. Optional.|
|`queue`|The queue that this machine uses. Must be defined in `queue.yaml`. Default: `default`.|
|`logging`|Set this to `persistent` to enable datastore-based tracking. If enabled, Fantasm will consume datastore and more taskqueue tasks. If enabled, you will likely also want to set up the [scrubber](ScrubbingFantasm.md). To disable, simply leave this feature out.|
|`states`|A list of machine states. See below for state attributes. One `initial` and at least one `final` state is required (may be the same state).|
|`context_types`|A list of datatypes for context variables. See below for a description. Optional.   |
|`countdown`|The number of seconds the transition of this machine should wait before executing. Can be overridden on transitions. Default: 0.|
|`task_retry_limit`|The maximum number of times to retry a failed transition before giving up. If `task_age_limit` is specified then the transition will be retried until both limits are reached (as per `TaskRetryOptions`). Can be overridden on transitions. Optional.|
|`min_backoff_seconds`|The minimum number of seconds to wait before retrying a transition after failure (as per `TaskRetryOptions`). Can be overridden on transitions. Optional.|
|`max_backoff_seconds`|The maximum number of seconds to wait before retrying a transition after failure (as per `TaskRetryOptions`). Can be overridden on transitions. Optional.|
|`task_age_limit`|The number of seconds after creation afterwhich a failed transition will no longer be retried. If `task_retry_limit` is also specified then the transition will be retried until both limits are reached (as per `TaskRetryOptions`). Can be overridden on transitions. Optional.|
|`max_doublings`|The maximum number of times that the interval between failed transition retries will be doubled before the increase becomes constant. The constant will be: `(2**(max_doublings - 1) * min_backoff_seconds)` (as per `TaskRetryOptions`). Can be overridden on transitions. Optional.|
|`target`|The default task target for this machine, i.e., `Task(target='...')`. Useful to specify particular versions or backends for machine transitions. Can be overridden on transitions. Optional.|
|`use_run_once_semaphore`|The taskqueue system can occasionally invoke handlers for a duplicated task. Fantasm uses a datastore transaction to prevent against this by default. This adds some additional overhead to each transition, but it is important if using Fantasm to compute statistics, etc. If this additional safety is not required (e.g., if your states are truly idempotent), then you can set this to False and turn off this overhead for this machine. Default: `True`.|

## State attributes ##

|`name`|The unique name of the state. Can only contain letters, numbers, and dashes.|
|:-----|:---------------------------------------------------------------------------|
|`namespace`|The namespace for the code associated to this state. Overrides any namespace specified at the machine level. Note that classes can be fully package specified as well and namespace is not required. Optional.|
|`action`|The action for the state. Responsible to return an event (for non-final states) to drive the machine forward.|
|`entry`|An action that will be executed immediately before the action. No return value is expected. Optional.|
|`exit`|An action that will be executed immediately before a transition. No return value is expected. This action is the first item executed _after_ a task is dequeued. Optional.|
|`initial`|A boolean flag indicating that this is the start state for the machine. Each machine requires exactly one initial state. Default: `False`.|
|`final`|A boolean flag indicating that this state is a final state for the machine. Each machine requires at least one final state. The action for a final state does not have to return an event to transition the machine. Default: `False`.|
|`transitions`|A list of valid transitions out of this state. See below for transition attributes. Optional (but only for machines that have a single state that is both initial and final).|
|`continuation`|A boolean flag specifying that this state is a fan-out that creates machine replicas. See [Advanced Concepts](AdvancedConcepts#Continuation.md) for details. Default: `False`.|
|`continuation_countdown`|An integer indicating the number of seconds to delay a continuation task; useful to "slow down" continuations across a single entity group that might otherwise cause transaction collisions. Optional. Default: 0|
|`fan_in`|Indicates that this is a fan-in state that joins machine replicas together. The value is an integer indicating the maximum number of seconds to wait while gathering a batch of machines. See [Advanced Concepts](AdvancedConcepts#Fan-In.md) for details. Default: not a fan-in state.|
|`fan_in_group`|Specifies a context key that will group machine fan-ins together. The value is a string that matches a context key.  See [Advanced Concepts](AdvancedConcepts#Fan-In.md) for details. Optional. If specified, `fan_in` must also be specified.|

## Transition attributes ##

|`event`|The name of the event that fires this transition. Must be unique within the transitions of the parent state. Can only contain letters, numbers, and dashes. This string value is expected to be returned by your state action. Fantasm will attempt to resolve this item using the machine namespace (or state namespace if specified); i.e., it will try to look up the string in your code. If this resolution fails, it will use this value as a raw string. Fantasm will raise a configuration error if this value resolves to an object in your code that is not a string.|
|:------|:------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|`to`   |The name of the state to transition to after receiving this event. Must match a state name of another state in the same machine.                                                                                                                                                                                                                                                                                                                                                                                                                                               |
|`action`|An action that will be executed after exiting the current state, but before entering the `to` state. No return value is expected. Optional.                                                                                                                                                                                                                                                                                                                                                                                                                                    |
|`namespace`|The namespace for the code associated to this transition. Overrides any namespace specified at the machine level. Note that classes can be fully package specified as well and namespace is not required. Optional.                                                                                                                                                                                                                                                                                                                                                            |
|`queue`|The queue that this transition uses. Must be defined in `queue.yaml`. Overrides any queue specified at the machine level. Optional.                                                                                                                                                                                                                                                                                                                                                                                                                                            |
|`countdown`|The number of seconds this transition should wait before executing. Overrides the `countdown` specified at the machine level. Optional. It is also possible to configure a random countdown; to do this, instead of providing a single integer, specify a YAML dictionary with two values: `minimum` and `maximum` and when queuing the task, Fantasm will choose a random countdown between these two values.                                                                                                                                                                 |
|`task_retry_limit`|The maximum number of times to retry a failed transition before giving up. If `task_age_limit` is specified then the transition will be retried until both limits are reached (as per `TaskRetryOptions`). Overrides the `task_retry_limit` specified at the machine level. Optional.                                                                                                                                                                                                                                                                                          |
|`min_backoff_seconds`|The minimum number of seconds to wait before retrying a transition after failure (as per `TaskRetryOptions`). Overrides the `min_backoff_seconds` specified at the machine level. Optional.                                                                                                                                                                                                                                                                                                                                                                                    |
|`max_backoff_seconds`|The maximum number of seconds to wait before retrying a transition after failure (as per `TaskRetryOptions`). Overrides the `max_backoff_seconds` specified at the machine level. Optional.                                                                                                                                                                                                                                                                                                                                                                                    |
|`task_age_limit`|The number of seconds after creation afterwhich a failed transition will no longer be retried. If `task_retry_limit` is also specified then the transition will be retried until both limits are reached (as per `TaskRetryOptions`). Overrides the `task_age_limit` specified at the machine level. Optional.                                                                                                                                                                                                                                                                 |
|`max_doublings`|The maximum number of times that the interval between failed transition retries will be doubled before the increase becomes constant. The constant will be: `(2**(max_doublings - 1) * min_backoff_seconds)` (as per `TaskRetryOptions`). Overrides the `max_doublings` specified at the machine level. Optional.                                                                                                                                                                                                                                                              |
|`target`|The default task target for this machine, i.e., `Task(target='...')`. Useful to specify particular versions or backends for machine transitions. Overrides the `target` specified at the machine level. Optional.                                                                                                                                                                                                                                                                                                                                                              |

## Context types ##

`context_types` is a dictionary of context variable names and their corresponding datatype. This can be used to have Fantasm automatically cast context variables to their appropriate types when deserializing from the taskqueue task. Whatever datatype you specify must have a constructor that takes a string representation and constructs the appropriate datatype. A typical `context_types` section might look like the following:

```
context_types:
  counter: int
  report-key: google.appengine.ext.db.Key
  running-total: float

  # you can also use NDB-style Keys:
  new-report-key: google.appengine.ext.ndb.Key
```

## Importing machine definitions from other files ##

To keep your `fsm.yaml` file orderly and place your state machine definitions closer to their code, you can use `import` statements in your `fsm.yaml`. To import another file, add an import statement after the `state_machines` declaration in `fsm.yaml`:

```
state_machines:

- import: src/backup/backup.yaml
- import: src/bulk_email/bulk_email.yaml
```

The files your refer to (e.g., in the above example `backup.yaml` and `bulk_email.yaml`) must start with the `state_machines` declaration. Any given machine must be wholly defined in a single YAML file.