## .bashrc

```sh
# ~/.bashrc
# If not running interactively, don't do anything
case $- in
    *i*) ;;
      *) return;;
esac

HISTCONTROL=ignoreboth
shopt -s histappend
HISTSIZE=1000
HISTFILESIZE=2000
shopt -s checkwinsize

## Git prompt
gitbr() {
	git branch --show-current 2> /dev/null | sed -e 's/\(.*\)/ (\1)/'
}

# A very basic prompt
# Terminal title to user@host
# [user@host ~/path] (git-branch) % 
export PS1='\[\e]0;$USER@$HOSTNAME \W\a\][\[\e[1;35m\]\u@\h \[\e[1;36m\]\W\[\e[0m\]]\[\e[1;33m\]$(gitbr)\[\e[1;00m\] % '
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