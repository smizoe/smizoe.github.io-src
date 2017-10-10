---
title: Managing Multiple Git Configurations
tags:
  - git
date: 2016-08-01 21:15:40
comments: true
---


Since I have a personal laptop which is different from the laptop I use at work, I have a problem when I would like to make a commit to my personal repository. (e.g., when I add a new function to my dotfiles, I usually want to use it on my personal laptop.); I need to manage multiple git configurations so that I can commit with my personal user profile instead of my profile in the company (user name, email etc.)

To address this problem, I so far set the following script to git's pre-commit hook together with a global git config with user.email and user.name set to 'SET\_ME\_LOCALLY':

```
#!/bin/sh
if [ "`git config user.name`" == SET_ME_LOCALLY ]; then
    echo "fatal: user.name is not set locally"
    exit 1
fi
if [ "`git config user.email`" == SET_ME_LOCALLY ]; then
    echo "fatal: user.email is not set locally"
    exit 1
fi
```

Although this stops commit when either of user name or email address is not set, this has a problem: I have to type in my name and email address even when I'm on my personal laptop.
To avoid typing my profile every time I make a new repository, I now set the following script as my pre-commit hook:

{% code pre-commit lang:bash %}
{% raw %}
#!/bin/bash
set -eu

## We read value of multiuser.config in global git config and find out a config file
## that we would like to merge with global config.
## We suport reading only one file since it should suffice for our use case

## currently we skip setup of config if .prev_config file exists in .git directory

if [ "${GIT_DIR}" = "$(pwd)" ] ; then
## bare repository
## see https://git-scm.com/docs/githooks/2.9.2
    GIT_CFG_PATH="$(pwd)/config"
else
    GIT_CFG_PATH="$(pwd)/.git/config"
fi

function check_if_merged(){
  ## 1st argument: path to the gitconfig (something like ${worktree_root}/.git/config)

  set +e
  stat "$(dirname "${1}")/.prev_config" &> /dev/null
  if [ "$?" -eq 0 ] ; then
      echo "We skip setting up config since we have .prev_config file" >&2
      exit 0
  fi
  set -e
}

function get_additional_cfg(){
    ## the following command fails if multiuser.confdir is not set
    local my_conf_dir="$(git config multiuser.confdir)"
    ## we need to expand ~; otherwise ls fails
    local my_conf_dir="${my_conf_dir/#\~/${HOME}}"

    echo "Please input the prefix of the config file: " >&2
    exec < /dev/tty
    read conf_prefix

    local pattern="${my_conf_dir}/${conf_prefix}*"
    local candidates=($(ls -d ${pattern}))

    local num_matched_files="${#candidates[@]}"
    if [ "${num_matched_files}" -ne 1 ] ; then
        if [ "${num_matched_files}" -gt 1 ] ; then
            echo "More than 1 file matched with pattern: ${pattern}" >&2
            echo -e "The matched files are:\n${candidates[@]}\n" >&2
        else
            echo "no files mathched with pattern: ${pattern}" >&2
        fi
        echo -e "The files in ${my_conf_dir} are:\n$(echo ${my_conf_dir}/*)\n" >&2
        echo "aborted the commit." >&2
        exit 2
    fi

    echo "${candidates[0]}"
    return 0
}


function update_conf(){
    ## 1st argument: default config (something like ${worktree_dir}/.git/config)
    ## 2nd argument: additional config which override the default one
    cp "${1}" "$(dirname "${1}")/.prev_config"
    DEFAULT_CFG="${1}" ADDITIONAL_CFG="${2}" python2 <<EOF
from __future__ import print_function
from StringIO import StringIO
from ConfigParser import ConfigParser
import os

cfg_file = os.environ['DEFAULT_CFG']
override_file = os.environ['ADDITIONAL_CFG']
parser = ConfigParser()

for filename in [cfg_file, override_file]:
  with open(filename, "r") as cfg_io:
    ini = ''.join(map(lambda line: line.lstrip(), cfg_io.readlines()))
    parser.readfp(StringIO(ini))

with open(cfg_file, "w") as f:
  for section_name in parser.sections():
    f.write("[{}]\n".format(section_name))
    for k, v in parser.items(section_name):
      f.write("\t{} = {}\n".format(k, v))
EOF
}

check_if_merged "${GIT_CFG_PATH}"
additional_cfg="$(get_additional_cfg)"
update_conf "${GIT_CFG_PATH}" "${additional_cfg}"
## we need to reload config by exiting the current process
echo "update finished. Please retry the commit. (This commit will exit with 1)" >&2
exit 1
{% endraw %}
{% endcode %}

The code above does the following:

1. checks if .git directory contains a file named .prev_config. If it did, this script exits with 0.
2. gets a list of git configuration files stored in a directory specified by value of multiuser.confdir in the global git config.
3. prompts the user to input (the prefix of) a git config to use.
4. merges the repository's local config file with the specified file and puts it to .git/config. The previous config file is backed up as .git/.prev_confing.

By using this, we can use different values for user names, email addresses and even for other configuration values.
