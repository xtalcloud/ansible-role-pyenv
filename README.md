# Ansible Role: `kriipke.pyenv` 
Spencer Smolen <spencer@xtal.cloud>

```yaml
- hosts: localhost
  roles:
    - role: pyenv
      vars: { user: 'eric', install_release: '3.9.1' }
```
*example playbook*

## Role Variables

No variables are required to run the role. The role will do a variety of things depending on which variables are defined (see examples below), but for those looking to just run it to install a particular version of python quickly for a particular user will need to have `user` + `install_release` defined OR `user` + `default_release`. The difference is basically whether you want that particular version of python to be merely accessible via `pipenv`, say for a particular project, or whether you want it to be the default python release for that user.

### `user`

This is the user for which the subsequent `pipenv` configuration will occur. If this variable is not defined, `pipenv` will not be installed.

This variable must correspond to a single user on the system this role is being applied to, so if you want to configure `pipenv` for multiple users you must do something like this:

```yaml
- hosts: localhost
  roles:
    - role: pyenv
      vars:
        user: "{{ item }}"
        release: '3.8.0'
      loop:
        - user1
        - user2
        - user3
```

### `default_release`

This variable is used to set the default python interpreter for `user`. If the release specified has not been built by `pipenv`, then it will be built. This command is effectively a wrapper for `pipenv global $RELEASE`.

### `install_release`

This variable is used to build a version of python from source as `user` via `pipenv`. For a list of possible options run: `pipenv install --list`. After runnning the role one with only the `user` variable defined. This command is effectively a wrapper for `pipenv install $RELEASE`.

### Other Variables

The role makes use of other variables defined in `defaults/main.yml` which, under most circumstances you will not have any need to change. They mostly pertain to information regarding the user's shell. If you have a funky shell setup, see the section after Examples named "Additonal Variables" for more info.

## Examples

```yaml
- hosts: localhost
  roles:
    - role: pyenv
```
This will just install the system-wide packages required for a user to install `pyenv` and build their own python releases within their home directory without actually configuring `pyenv` for any particular user.

```yaml
- hosts: localhost
  roles:
    - role: pyenv
      vars: { user: 'eric' }
```
This will both install the system packages needed to build python releases as well as install `pyenv` + `pyenv-virtualenv` for the user `eric` without actually building any python releases

```yaml
- hosts: localhost
  roles:
    - role: pyenv
      vars: { user: 'eric', install_release: '3.9.1' }
```

This will do everything the two examples above do, however it will additionally build Python 3.9.1 as the user `eric` for use with `pyenv`.

```yaml
- hosts: localhost
  roles:
    - role: pyenv
      vars: { user: 'eric', default_release: '3.9.1' }
```

Finally, the last example does all the things mentioned above, as well as set the default `python` bin for the user `eric` to `~/.pyenv/versions/3.9.1/bin/python`.

The role is configured such that if you specify a `default_release` for a user that they have not yet built, it will automatically be built. Additionally, releases will not be re-built if they have already been installed.

## Additional Variables

If your shell is configured to source it's default init file everytime a shell is opened you don't need to worry about these. In othe words, if your login shell is bash and you haven't changed it so that it *won't* source `~/.bashrc` at some point when you open a new shell (or `~/.zshrc` for zsh), then ignore everything below.

For the rest of you: the role will add the relevant `pipenv` init code to *your login shells default rc file*. This means there are two circumstances which you will want to change these additional variables:

1. the shell in which you work in *is not* the one defined in `/etc/passwd` as your login shell. This could be because you do something like `exec tmux` upon login or in your X init scripts or what have you. If you're not sure just see if the output of `echo $SHELL` matches the `awk '/$USER/ {print $5} /etc/passwd/'`; if it does not, keep reading.

2. The other quirky case is if you have configured your shell, for whatever reason, to ignore the default init script to be run upon launching a new shell, e.g. `~/.bashrc` for bash or `~/.zshrc` for zsh. The aformentioned files are where this role is going to addd the `pipenv` init commands, so keep reading if this is you.

### `user_shell`

This is used to both:
* determine the path it should look for `user_shell_rc`
* to actually run `pipenv` commands once the init commands are placed in the correct shell rc file

If you need to adjust it any of the following values are acceptable:
 - `sh`
 - `bash`
 - `zsh`
 - `csh`

### `user_shell_rc`

This is the name of the file that is considered the default rc script path for   the shell defined in `user_shell`, the assumed values are
 - `~/.bashrc` for bash
 - `~/.zshrc` for zsh
 - `~/.cshrc` for csh
 - *POSTIX sh does not have a specified init rc file*
