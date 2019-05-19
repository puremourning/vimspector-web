## Vimspector

This page contains the demo images, and probably more easily accessible form of documentation.

## UI Overview

When starting debugging, vimspector creates a new tab page with 5 main sections:

* The code window
* The scopes window
* The watch window
* The call stack window.
* The output window

![Overview](/img/vimspector-overview.png)

## The code window

This window has 2 elements:

* The debug toolbar (WinBar), and
* The code buffer.

In the code buffer, the current line is highlighted with a sign. When selecting a stack frame in the call stack, the code window is updated to display the current line in that stack frame:

![Code Window](/img/vimspector-code-window.png)

## The scopes window

Displays the variables in different visible scopes. Entries can be expanded by
putting the cursor on the line and pressing `<CR>`.

![Scopes Window](/img/vimspector-locals-window.png)

## The watch window

Allows execution and display of arbitrary expressions. This window contains a
prompt buffer. When you enter insert mode (e.g. with `i`) you can type an
expression. When hitting `<CR>`, the expression is evaluated and the result is
displayed. If the result is has structure, it can be expanded like in the scoped
window by hitting `<CR>`. The little `[+]` and `[-]` icons show what is
expandable.

Watches are recomputed regularly so update as you're stepping through code, for
example.

![Watch Window](/img/vimspector-watch-window.png)

## The call stack windows

This shows the call stack for all of the threads in the program.

As with variables, stacks can be expanded and collapsed with `<CR>` and have
little `+` and `-` icons where appropriate. In addition, hitting `<CR>` on a
particular stack frame will open the file in the code window and place the
cursor there.

![Call Stack](/img/vimspector-callstack-window.png)

## The output window

This contains a number of buffers for both monitoring debugger state an interracting with the debugger itslef.

The most interesting buffers are:

* The Console, which allows for an interactive REPL with most debug adapters,
* The debugee standard error stream, and
* The vimspector log, which allows for debugging when vimspector breaks

These windows typically scroll automatically as new content is added asyncronously.

![Code Window](/img/vimspector-output-window.png)

