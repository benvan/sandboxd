#!/bin/bash

: ${SANDBOXRC:="$HOME/.sandboxrc"}

# holds all hook descriptions in "cmd:hook" format
sandbox_hooks=()

# deletes all hooks associated with cmd
function sandbox_delete_hooks(){
  local cmd=$1
  for i in "${sandbox_hooks[@]}";
  do
    if [[ $i == "${cmd}:"* ]]; then
     local hook=$(echo $i | sed "s/.*://")
     unset -f "$hook"
    fi
  done

}


# prepares environment and removes hooks
function sandbox(){
  local cmd=$1

   if [[ "$(type $cmd | grep -o function)" = "function" ]]; then
    (>&2 echo "sandboxing $cmd ...")
    sandbox_delete_hooks $cmd
    sandbox_init_$cmd # run user-defined sandbox
  else
    (>&2 echo "sandbox '$cmd' not found.\nHave you defined sandbox_init_$cmd(){ ... } in your ~/.sandboxrc?")
    return 1
  fi
}

function sandbox_hook(){
  local cmd=$1
  local hook=$2

  sandbox_hooks+=( "${cmd}:${hook}" )
  eval "$hook(){ sandbox $cmd; $hook \$@ ; }"
}




source "$SANDBOXRC"

# create hooks for the sandbox names themselves
function sandbox_initialise(){
  local funcs="$( cat $SANDBOXRC | sed 's/#.*$//g' | grep -o 'sandbox_init_[^(]\+' )"
  while read f; do
    local cmd=$(echo $f | sed s/sandbox_init_//)
    sandbox_hook $cmd $cmd;
  done < <(echo "$funcs")
}

sandbox_initialise
