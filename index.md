## Vimspector

This page contains the demo images, and probably more easily accessible form of documentation.

## UI Overview

When starting debugging, vimspector creates a new tab page with 5 main sections:

* The code window
* The output window
* The locals window
* The watch window
* The call stack window.


## The code window

This window has 2 elements:

* The debug toolbar (WinBar), and
* The code buffer.

In the code buffer, the current line is highlighted with a sign. When selecting a stack frame in the call stack, the code window is updated to display the current line in that stack frame:

![Code Window](/img/vimspector-code-window.png)

## The output window

This contains a number of buffers for both monitoring debugger state an interracting with the debugger itslef.

The most interesting buffers are:

* The Console, which allows for an interactive REPL with most debug adapters,
* The debugee standard error stream, and
* The vimspector log, which allows for debugging when vimspector breaks

These windows typically scroll automatically as new content is added asyncronously.

![Code Window](/img/vimspector-output-window.png)


