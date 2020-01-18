# zsh-abbr

An enhanced [zsh](http://www.zsh.org/) port of [fish shell](http://www.fishshell.com/)'s abbreviations.

**abbr** manages abbreviations - user-defined words that are replaced with longer phrases after they are entered.

For example, a frequently-run command like `git checkout` can be abbreviated to `gco`. After entering `gco` and pressing <kbd>Space</kbd>, the full text `git checkout` will appear in the command line. To prevent expansion, press <kbd>Ctrl</kbd><kbd>Space</kbd> in place of <kbd>Space</kbd>. Pressing <kbd>Enter</kbd> after an abbreviation will expand the abbreviation and accept the current line.

Abbreviations are expanded whether or not they are the first word on the line. Cross-session abbreviations are stored in the clear in a plaintext configuration file.

Run `abbr --help` (or `abbr -h`) for documentation.

## Contents

1. [Installation](#installation)
1. [Quick Start](#quick-start)
1. [Scopes](#scopes)
1. [Options](#options)
1. [Configuration](#configuration)
1. [Uninstalling](#uninstalling)

## Installation

- **[antibody](https://getantibody.github.io/)**: Add `olets/zsh-abbr` to your plugins file. If you use static loading, reload plugins.

- **[Antigen](https://github.com/zsh-users/antigen)**: Add `antigen bundle olets/zsh-abbr` to your `.zshrc`.

- **[Oh-My-Zsh](https://github.com/robbyrussell/oh-my-zsh)**:

  - Clone to OMZ's plugins' directory:

    ```shell
    git clone https://github.com/olets/zsh-abbr.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-abbr
    ```

  - Add to the OMZ plugins array in your `.zshrc`:

    ```shell
    plugins=( [plugins...] zsh-syntax-highlighting)
    ```

- **[Prezto](https://github.com/sorin-ionescu/prezto)**: if you know, open a PR!

- **[zgen](https://github.com/tarjoilija/zgen)**: add `zgen load olets/zsh-abbr` to your `.zshrc`.

- **[zplug](https://github.com/zplug/zplug)**: add `zplug "olets/zsh-abbr"` to your `.zshrc`.

- **[zplugin](https://github.com/zplug/zplug)**: add `zplugin load olets/zsh-abbr` to your `.zshrc`.

If running `abbr` gives an error "zsh: permission denied: abbr", reload zsh:

```shell
% source ~/.zshrc # or your custom .zshrc path
```

## Quick Start

```shell
# Add and use an abbreviation
% abbr m master
% m[Space] # expands to `master`
% git checkout m[Space] # becomes `git checkout master`
% git checkout m[Enter]
# expands to and runs `git checkout master`

% abbr gcm git checkout master
# Note the `m`s in `gcm` and `master` did not expand. Expansion only run on entire words
% gcm[Enter]
# expands to and runs `git checkout master`

# The above will in other open sessions, and in future sessions.
# Add an abbreviation only available in the current terminal
% abbr -g gco[Ctrl-Space] https://en.wikipedia.org/wiki/GCO
# (Note that Ctrl-Space opts out of space-triggered expansion)
% gco[Space] # expands to `https://en.wikipedia.org/wiki/GCO`

# Rename it
% abbr -r gco[Ctrl-Space] wgco
% gco[Space] # expands to `git checkout`
% wgco[Space] # expands to `https://en.wikipedia.org/wiki/GCO`

# Delete it
% abbr -g wgco[Ctrl-Space]
% wgco[Space] # no expansion

# Migrate your aliases to abbreviations
% abbr -p # creates abbreviations from all aliases
% abbr -i # creates abbreviations from all Git aliases, adding 'g' prefix
```

## Scopes

A given abbreviation can be made available in the current zsh session (i.e. in the current terminal) —these are called *global* abbreviations— or to all terminals —these are called *universal* abbreviations.

## Options

```
abbr [(--add | -a)
      | (--create-aliases | -c)
      | (--erase | -e)
      | (--expand | -x)
      | (--global | -g)
      | (--help | -h)
      | (--git-populate | -i)
      | (--list | -l)
      | (--populate | -p)
      | (--rename | -r)
      | (--show | -s)
     ]
     [(--global | -g)
      | (--universal | -U)]
```

`zsh-abbr` has options to add, rename, and erase abbreviations; to add abbreviations for every alias or Git alias; to list the available abbreviations with or without their expansions; and to create aliases from abbreviations.

The default when `abbr` is run with no arguments is to show all abbreviations and their expansions.

### Add

```
abbr (--add | -a) [(--global | -g) | (--Universal | -U)] WORD EXPANSION
```

Add a new abbreviation. If the **--global** scope is used, the abbreviation will only be available in the current session. Otherwise, or if the **--Universal** scope is used, the abbreviation will be immediately available to all open and future sessions.

```shell
% abbr --add gcm git checkout master
% gcm[Space] # expands as git checkout master
% gcm[Enter] # expands and accepts git checkout master
```

The following are equivalent:

```shell
% abbr --add --Universal gcm git checkout master
% abbr -a --Universal gcm git checkout master
% abbr --Universal gcm git checkout master
% abbr --add -U gcm git checkout master
% abbr -a -U gcm git checkout master
% abbr -U gcm git checkout master
% abbr gcm git checkout master
```

### Create Aliases

```
abbr (--create-aliases | -c) [(--global | -g) | (--Universal | -U)] [DESTINATION]
```

Export abbreviations as aliases declarations. To export global abbreviations, use  **--global**. Othewise, or if the **--Universal** scope is used, universal abbreviations are exported.

```shell
% abbr --add gcm git checkout master
% abbr --add --global g git
```
```shell
% abbr --create-aliases
alias -g gcm='git checkout master'
```
```shell
% abbr --create-aliases --global
alias -g g='git'
```
```shell
% abbr --create-aliases ~/.zshrc
% cat ~/.zshrc
# -- snip --
alias -g g='git'
```

### Erase

```
abbr (--erase | -e) [(--global | -g) | (--Universal | -U)] WORD
```

Erase an abbreviation. Specify **--global** scope to erase a global abbreviation.

```shell
% abbr --add gcm git commit master
% gcm[Enter] # expands and accepts git commit master
Switched to branch 'master'
% abbr --erase gcm[Ctrl-Space][Enter]
% gcm[Space|Enter] # nothing
% abbr --add --global gcm echo gimme cookie monster
% gcm[Enter]
gimme cookie monster
% abbr --erase --global gcm[Ctrl-Space][Enter]
% gcm[Enter]
Already on 'master'
```

### Expand

```
abbr (--expand | -x) WORD
```

Output the WORD's EXPANSION. Useful in scripting.

```shell
% abbr --add gc git checkout
% abbr --expand gc[Ctrl-Space]
git checkout
```

### Git-Populate

```
abbr (--git-populate | -i) [(--global | -g) | (--Universal | -U)]
```

Add abbreviations for every Git alias available in the current session. WORDs are prefixed with `g`; EXPANSIONs are prefixed with `git[Space]`. Use the **--global** scope to create global abbreviations.

This command is useful for migrating from aliases to abbreviations.

```shell
% git config alias.co checkout
% abbr --git-populate --global
% gco[Space] # expands to git checkout
% source ~/.zshrc
% gco[Space] # no expansion
% abbr --git-populate
% source ~/.zshrc
% gco[Space] # expands to git checkout
```

Note for users migrating from Oh-My-Zsh: [OMZ's Git aliases are shell aliases](https://github.com/ohmyzsh/ohmyzsh/blob/master/plugins/git/git.plugin.zsh), not aliases in the Git config. To add abbreviations for them, use **Populate**.

### List

```shell
abbr (--list|-l)
```

List all abbreviations available in the current shell. Universal abbreviations appear first.

```shell
% abbr a apple
% abbr b ball
% abbr -g c cat
% abbr -l
a
b
c
% source ~/.zshrc
% abbr -l
a
b
```

### Populate

```
abbr (--git-populate | -i) [(--global | -g) | (--Universal | -U)]
```

Add abbreviations for every alias available in the current session. Use the **--global** scope to create global abbreviations.

This command is useful for migrating from aliases to abbreviations.

See also **Git-Populate**.

```shell
% cat ~/.zshrc
# --snip--
alias -g d='bin/deploy'
# --snip--
% abbr --populate --global
% d[Space] # expands to bin/deploy
% source ~/.zshrc
% d[Space] # no expansion
% abbr --git-populate
% source ~/.zshrc
% d[Space] # expands to bin/deploy
```

### Rename

```
abbr (--rename | -r) [(--global | -g) | (--Universal | -U)] OLD_WORD NEW_WORD
```

Rename an abbreviation. Use the **--global** scope to rename a global abbreviation.

```shell
% abbr --add gcm git checkout master
% gcm[Space] # expands to git checkout master
% gm[Space] # no expansion
% abbr --rename gcm[Ctrl-Space] gm
% gcm[Space] # no expansion
% gm[Space] # expands to git checkout master
```

### Show

```
abbr [(--show|-s)]
```

Show all the abbreviations available in the current session, along with their expansions. Global abbreviations are marked `-g` and follow universal abbreviations, which are marked `-U`.

```shell
% abbr --add gcm git checkout master
% abbr --add --global a apple
% abbr --show # or `abbr` with no arguments
abbr -a -U -- gcm git checkout master
abbr -a -g -- a apple
% source ~/.zshrc
% abbr --show
abbr -a -U -- gcm git checkout master
```

## Configuration

### Storage

Universal abbreviations live in a plain text file which you can manually edit, shared, etc. Its default location is `${HOME}/config/zsh-abbr/universal`. Customize this by setting the `ZSH_ABBR_UNIVERSALS_PATH` variable in your `.zshrc` before loading zsh-abbr.

```shell
% cat ~/.zshrc
# -- snip --
ZSH_ABBR_UNIVERSALS_PATH="path/to/my/universal/abbreviations"
# -- snip --
# load zsh-abbr
```

Note that the default file is created the first time zsh-abbr is run. If you customize the path, you may want to delete the default file or even the default zsh-abbr config directory.

### Bindings

By default

- <kbd>Space</kbd> expands abbreviations
- <kbd>Ctrl</kbd><kbd>Space</kbd> is a normal space
- <kbd>Enter</kbd> expands and accepts abbreviations

(In incremental search mode, <kbd>Space</kbd> is a normal space and <kbd>Ctrl</kbd><kbd>Space</kbd> expands abbreviations.)

If you want to set your own bindings, set `ZSH_ABBR_DEFAULT_BINDINGS` to `false` in your `.zshrc` before loading zsh-abbr. In the following example, expansion is bound to <kbd>Ctrl</kbd><kbd>a</kbd>:

```shell
% cat ~/.zshrc
# -- snip --
ZSH_ABBR_DEFAULT_BINDINGS=false
bindkey "^A" _zsh_abbr_expand_space
# -- snip --
# load zsh-abbr
```

## Uninstalling

Delete the zsh-abbr configuration directory. Note that this will permanently delete the universal abbreviations file.

```shell
% rm -rf $(dirname "$ZSH_ABBR_UNIVERSALS_PATH")
```

Then follow the standard uninstallation procedure for your installation method. This is typically the reverse of what you did to install.