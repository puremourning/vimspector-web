## Setting up Vimspector

This page contains instructions for:

* installing and configuring vimspector to debug Vim
* building a debug version of Vim

## Installation

See the [Vimspector][] readme for detailed installation instructions, but in
short, if you're running a supported OS:

* Untar a release into `~/.vim/pack`
* Add this to your `.vimrc`:

```viml
let g:vimspector_enable_mappings = 'HUMAN'
packadd! vimspector
```

## Getting Started

Once installed, you need to configure some debug profiles for your project. In
order to debug your code, you need to tell Vimspector:

* Which debug adapter to use,
* How to start up or connect to your application.

Both of these are configured in a `.vimspector.json` file.

But first, we need to something to debug. In our example, we're going to be
debugging Vim itself (why not), so let's get that set up. Whether you're on
macOS or Linux, the steps are roughly the same:

* Check out the Vim code,
* Build it with debug information
* Set up the `.vimspector.json` to start vim and run a Vim test
* Set a breakpoint in the Vim code
* Start up vimspector and debug Vim.

The idea is to be able to run a Vim test and step through the Vim code to see
what's going on. We're going to set it up so that you can run any Vim test,
rather than just a specific one.

First, let's check out the Vim code and build it with debug info.

### Check out and build the Vim code

This is simple. Just follow the instructions for building Vim, but make sure
that `CFLAGS` contains at least `-g`, but I recommend the following: 
`-g -gdwarf4 -DDEBUG -O0 -fno-omit-frame-pointer`.

Here's an example:

```
$ git clone https://github.com/vim/vim
$ cd vim
$ CFLAGS='-g -gdwarf4 -DDEBUG -O0 -fno-omit-frame-pointer' \
    ./configure --prefix=$(pwd)/root \
                --with-features=huge \
                --enable-python3interp \
                --enable-terminal  \
                --enable-fail-if-messing 
$ make
$ make install
```

You should now be able to run Vim by running `./src/vim`. Bonza.

## Set up the vimspector config

Starting Vim is simple, but we want to run a Vim test. For the sake of this
demo, we're going to run a Python test 'test_python3.vim'. This is a little more
involved, and the details aren't really all that important, but the command we
want to execute is, from the directory `src/testdir`:

```
$ VIMRUNTIME=../../runtime ../vim -f -u unix.vim -U NONE --noplugin --not-a-term -S runtest.vim test_python3.vim
```

You should be able to try this manually.

### Structure of the config file

The config file is called `.vimspector.json` and should be placed in the root of
your project. Vimspector looks up the directory stack from Vim's current working
directory to find it.

The JSON must contain a single object with the following structure:

* `adapters`: Object mapping adapter names to adapter configuration. We won't
  be covering this here as the bundled adapter will be used.
* `configurations`: Object mapping launch configuration name to vimspector debug
  configuration. 

Vimspector configurations contain the following keys:

* `adapter`: name of an adapter to use. Must appear either in the `adapters`
  block, or in one from `.gadgets.json`
* `remote-request`, `remote-cmdLine`: Used for remote debugging (not covered
  here)
* `variables`: preset variables, or variables set from commands (not covered
  here)
* `configuration`: the debug adapter protocol launch configuration object. This
  is where the actual debugger configuration goes and varies depending on the
  adapter being used.

In the `configuration` mapping, the following are mandatory:

* `request`: `launch` or `attach`
* `type`: sometimes has to be supplied for very picky adapters

### Config Variables

Vimspector pre-processes the values of the configuration object, performing
certain simple variable replacements.

Vimspector makes some things available to you as variables. In particular,
entries in the vimspector configuration can contain variable replacements. The
following predefined variables are relevant to this demo:

* `${workspaceRoot}` - Expands to the path containing `.vimspector.json`
* `${gadgetDir}` - Expands to the OS-specific gadget home.
* `$$` - A dollar sign
* `${dollar}` - A dollar sign

In addition, vimspector asks the user to fill in the values for any unknown
variables, so we can use something like this:

* `${anythingYouLikeHere}` - Asks the user to specify the value

Or, more usefully in this case:

* `${Test}` - Ask the user to supply the name of the test, e.g. `test_python3`

Variables can also be set in the launch configuration and adapter configuration.
In these cases it's possible to set the variables by running some shell command.
This won't be used in the demo, but for example, to set the variable by running
a script:

```json
{
  "configurations": {
    "some-configuration": {
      "variables": {
        "gdbserver-version": {
          "shell": [ "/path/to/my/scripts/get-gdbserver-version" ],
          "env": {
            "SOME_ENV_VAR": "Value used when running above command"
          }
        },
        "some-other-variable": "some value"
      }
    }
  }
}
```

That's fairly advanced usage and we won't use that here.

### Debug adapters

Vimspector supports any DAP-compliant debug adapter, and you have to tell
Vimspector how to launch the debug adapter thought an `adapter` block.
Fortunately, Vim is a C program and there is a very good debug adapter for this
bundled with vimspector called `vscode-cpptools`. All of its adapter
configuration is bundled too (in `.gadgets.json`) so we don't need any
`adapters`.

For this example configuration, we're going to use the pre-bundled
`vscode-cpptools` package. Fortunately the required launch configuration is
[nicely documented][vscode-cpptools-launch], so we can see where to specify:

* The program to launch, and how to pass command line arguments
* The working directory
* The debugger options, such as whether to break on entry

### Putting it together

So the info we need for our single debug configuration is:

* Debug adapter: `vscode-cpptools` 
* Program: `${workspaceRoot}/src/vim`
* Arguments: `-f -u unix.vim -U NONE --noplugin --not-a-term -S runtest.vim test_python3.vim`
* Environment: `VIMRUNTIME=${workspaceRoot}/runtime`
* Type of debugger: `lldb` for macOS, `gdb` for Linux

This leads to the following (complete) `.vimspector.json`. Write this to
`/path/to/your/vim/checkout/.vimspector.json`:

```json
{
  "configurations": {
    "Vim - run a test": {
      "adapter": "vscode-cpptools",
      "configuration": {
        "type":    "cppdbg",
        "request": "launch",
        "program": "${workspaceRoot}/src/vim",
        "args": [
          "-f",
          "-u", "unix.vim",
          "-U", "NONE",
          "--noplugin",
          "--not-a-term",
          "-S", "runtest.vim",
          "${Test}.vim"
        ],
        "cwd": "${workspaceRoot}/src/testdir",
        "environment": [
          { "name": "VIMRUNTIME", "value": "${workspaceRoot}/runtime" }
        ],
        "externalConsole": true,
        "stopAtEntry": true,
        "MIMode": "lldb",
        "logging": {
          "engineLogging": false
        }
      }
    }
  }
}
```

Note:

* I enabled `stopAtEntry` so that the dbeugger breaks on start. This is optionsl
* If you have problems, toggle `engineLogging`: this will include the gdb/lldb
  output in the console
* If you're on Linux, use `gdb` for `MIMode`
* You can also use `${MIMode}` and enter the MIMode when starting debugging if
  you run on multiple environments.

## Starting debugging

Now that we have written the config and installed Vimspector, let's fire it up.

First, let's set a breakpoint. We could do this after starting, but it's usually
convenient to do this when editing or navigating the code. So let's do that.

### Setting a breakpoint

Start Vim from the vim source tree and open up `src/if_python3.c`, and navigate
to the start of the function `ex_py3`. This is the entry point for the `:py3`
command.

If you enabled `HUMAN` mode mappings, press `<F9>` to toggle a breakpoint on
that line. You should see a marker in the Vim gutter. You can press `<F9>` again
to disable that breakpoint, and a third time to remove it.

Add a breakpoint on the `{` of the function.

### Start debugging

Press `<F5>`. 

This will read your configuration and as there's only one debug config launch
that one. You should be presented with a question about which test to run. You
expected that though, so enter it.

Enter `test_python3` and hit `<CR>`.

You may have to hit `<CR>` a couple more times, because Vimspector likes to
conservatively spam you at this point as it is starting up the debug adapter.

At this point you might be a bit overwhelmed, so let's take a look at the UI
that you should see.

[vimspector]: https://github.com/puremourning/vimspector
[vscode-cpptools-launch]: https://github.com/microsoft/vscode-cpptools/blob/master/launch.md
