# sandboxd
Speed up your bashrc / zshrc: avoids running (slow) setup commands until you actually need them.

# Why?
Having installed `nvm`, `rvm`, `virtualenvwrapper` and other similar [gubbins](http://www.urbandictionary.com/define.php?term=gubbins) over time, my shell starts horrifically slowly. By running these setup scripts on-demand, my time-to-first-prompt is nice and fast again. On top of this, I rarely need all of these tools enabled at the same time...

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
  source $(brew --prefix nvm)/nvm.sh #long running setup command
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
[20:45:53 ~] echo 'console.log("hi")' | node
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

# Shims
Tools like [pyenv](https://github.com/pyenv/pyenv), [rbenv](https://github.com/rbenv/rbenv), [nodenv](https://github.com/nodenv/nodenv) etc use the concept of ['shims'](https://github.com/pyenv/pyenv#understanding-shims), which are helper scripts created when installed packages create command line utilities (e.g. installing the AWS CLI via Pip creates the `aws` command). To make it easy for sandboxd to lazy-load these environments when any of these 'shims' are called, use the function `sandbox_hook_shims {<name>} [<dir>]`. See [sandboxrc.example.pyenv](sandboxrc.example.pyenv) for more information.

Example:
Shims created under `~/.pyenv/shims/`
```bash
$ ls -l ~/.pyenv/shims
ansible
ansible-config
aws
flask
[...]
```

To automatically add them all as a sandboxd `hook`, simply add the following to `.sandboxrc`:
```bash
# in ~/.sandboxrc

sandbox_init_pyenv() {
  export PYENV_ROOT="$HOME/.pyenv"
  export PATH="$PYENV_ROOT/bin:$PATH"
  export VIRTUAL_ENV_DISABLE_PROMPT=1
  eval "$(pyenv init -)"
  eval "$(pyenv virtualenv-init -)"
}

sandbox_hook_shims pyenv
```

Here, the `pyenv` argument of `sandbox_hook_shims` matches the function name defined above (`sandbox_init_pyenv`).
`sandbox_hook_shims` assumes the shim directory to be `~/.<name>/shims` where `<name>` is the first argument. If the diretory is different, pass it as an argument:

```bash
sandbox_hook_shims pyenv /path/to/shim/directory
```

## sandboxrc configuration file
The location of the configuration file depends on the environment's configuration. The file is searched in the following order:

1. `$SANDBOXRC` -  if it is set. This has highest precedence. Thus you can set this to override to custom location.
1. `$XDG_CONFIG_HOME/sandboxd/sandboxrc` - if the directory `sandboxd/` exist. Note that `$XDG_CONFIG_HOME` defaults to `$HOME/.config`
1. `$HOME/.sandboxrc` - fall back to old default location.
