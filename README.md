# sandboxd
Speed up your bashrc / zshrc: avoids running (slow) setup commands until you actually need them.

# Why?
Having installed `nvm`, `rvm`, `virtualenvwrapper` and other similar gubbins over time, my shell starts horrifically slowly. By running these setup scripts on-demand, my time-to-first-prompt is nice and fast again. On top of this, I rarely need all of these tools enabled at the same time...

# How?
sandboxd creates a placeholder shell function for each command you specify (e.g. `rvm`). When this command gets run for the first time, the following happens:
- the `cmd` placeholder function (plus all associated placeholders) gets removed
- the setup you have associated with `cmd` gets run,
- `cmd` gets run with the original arguments

# Usage
To 'sandbox' a setup, wrap it in a function named `sandbox_init_[name]`:

```bash
# in ~/.bashrc / your shell rc file
source /path/to/sandboxd


# in ~/.sandboxrc
sandbox_init_nvm(){
  source $(brew --prefix nvm)/nvm.sh #long running command
}

# create hooks for commands 'nvm', 'node' and 'nodemon'
sandbox_hook nvm node
sandbox_hook nvm nodemon
# this one not needed: it's created automatically based on the sandbox name
# sandbox_hook nvm nvm
```

The sandbox setup gets run _once_, when either `nvm`, `nodemon` or `node` is used for the first time:

```bash
[20:45:44 ~] echo 'console.log("hi")' | node
sandboxing nvm ...
hi
20:45:53 [~] echo 'console.log("hi")' | node
hi
```

## Manually calling the sandbox command
To manually run a specific sandbox setup, run `sandbox [name]`

This might be useful if you want to run a sandbox that doesn't have an associated command, or to create "feature flags" in your rc file:

```bash
#uncomment to enable features

# sandbox virtualenv
sandbox rvm
sandbox nvm
```
