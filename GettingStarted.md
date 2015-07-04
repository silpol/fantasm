# Getting Started #

  1. Checkout the [Fantasm source](http://code.google.com/p/fantasm/source/checkout).
  1. Place the `fantasm` package in your Python path (found in the `src` directory).
  1. Write an `fsm.yaml` file for your desired machine. See YamlDescription for details.
  1. Write event handlers for the states. See FsmActions for details.
  1. Add the Fantasm mount point to your `app.yaml` file. See AppYaml for details.
  1. Hit the machine-specific url for your machine. The FSM will begin to execute, queuing tasks and executing your event handlers. See MachineEntryPoints for details.

## Sample Machines ##

There is a sample application located in the `test` directory of the download that shows a number of sample machines. Use the development app server pointing at this application and hit the root page (e.g., http://localhost:8102). Instructions appear on that page.

**NOTE** The test application relies on a symbolic link to the fantasm directory in `src`. Depending on your operating system or your svn client, you may need to manually copy the `src/fantasm` directory into your `test` directory.