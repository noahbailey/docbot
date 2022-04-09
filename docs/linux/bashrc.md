## Bashrc

```sh
# ~/.bashrc
# If not running interactively, don't do anything
case $- in
    *i*) ;;
      *) return;;
esac

# Configure history
HISTCONTROL=ignoreboth
shopt -s histappend
HISTSIZE=1000
HISTFILESIZE=2000
shopt -s checkwinsize

# A very basic prompt
export PS1="[\033[1;35m\u@\h \033[1;36m\W\033[0m] "

# Setup colours
eval "$(dircolors -b)"

# Import aliases
source ~/.bash_aliases

## Aliases & config
export EDITOR="/usr/bin/vim"
export PATH="$PATH:~/.local/bin"
```


### bash_aliases
```sh
# ~/.bash_aliases
alias ls='ls --color=auto'
alias l='ls -lrt'
alias grep='grep --color=auto'
alias egrep='egrep --color=auto'
alias ll='ls -la'
```