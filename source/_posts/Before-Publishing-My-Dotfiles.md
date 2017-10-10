---
title: Before Publishing My Dotfiles...
tags:
  - git
  - ansible
date: 2016-02-09 21:07:25
comments: true
---


A lot of people have published their dotfiles,
 and I have recently published [my dotfiles](https://github.com/smizoe/dotfiles) on my github repository.

Although I have been using git to manage my dotfiles, I did not make it public for a long time,
because several files in my dotfiles repository contain sensitive information
(for example, my proxy.pac contained FQDN of a proxy server to connect to servers in a datacenter.)

To bring my repository to a publishable state, I followed the following steps:

1. after taking a backup of the sensitive files, remove them from the index of git.

2. encrypt the files (using ansible-vault.)

3. put them when running a provision script.


In this post, I explain the steps above I took.

<!-- more -->

Removal of Sensitive Data
-------------------------
github has an article on [removing sensitve data](https://help.github.com/articles/remove-sensitive-data/) and I followed the way the article describes.
To make the story short, I ran the following script with paths to files with sensitive data after taking backup of the files:

{% gist 3aa4d1e4a684a4a3c9d4 %}

Encrypting Files
----------------

There are several viable options ([AES crypt](https://www.aescrypt.com) etc.) to encrypt files.
This time I chose ansible-vault among them, since I have automated setup of a new pc using ansible.

Encrypting files with ansible-vault is fairly easy:

```
$ ansible-vault encrypt file_name
```

Once you run the command above, input of a password is prompted and the file is encrypted with the password supplied.
If the number of files to be encrypted is large, one can supply `--new-vault-password-file` option to avoid typing a password multiple times.

How to Use Encrypted Data
-------------------------

As of ansible 2.0.0.2 (installed using homebrew), encrypted files are not decrypted if we use copy module directly.
My current solution for copying encrypted files is to use lookup plugin in combination with copy plugin
in the following manner (except from [my repository](https://github.com/smizoe/dotfiles/blob/master/provisioning/roles/vault/tasks/main.yml)):

```
- name: copy encrypted files
  copy: content="{{lookup('file', item.path)}}" dest="{{home}}/dotfiles/{{item.path| relpath(role_path ~ '/files')}}"
  with_items: encrypted_files.files
```
