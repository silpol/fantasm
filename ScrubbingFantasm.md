Fantasm maintains some Datastore entities for logs, stats and other items to ensure proper operation.

Some of this information is historical, and some of this information is for proper machine operation. For the latter, due to the fact that tasks can retry, some of this information needs to hang around for at least the task tombstone period - approximately 7 days.

Fantasm comes with a built-in scrubber that can be configured to run on a schedule and delete out the old data. In `cron.yaml`, specify the following:

```
- description: Fantasm Scrubber
  url: /fantasm/fsm/FantasmScrubber/?age=90&method=POST
  schedule: every day 01:00
```

Of course, you can set whatever schedule you want. The `age` parameter indicates the number of days worth of data to keep. For correct operation, you should keep at least 10 days worth of data. If not specified, the default value for `age` is 90.