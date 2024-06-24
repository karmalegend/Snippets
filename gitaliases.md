# Git aliases 

Some handy everyday git aliases.


### zsh
```
[alias]
	getremote = !git config --get remote.origin.url | awk -F: '{print \"https://github.com/\"$2}'
	refresh = !git fetch && git pull
	new = '!f() { git stash; git checkout master; git refresh; git stash pop; git checkout -b $1; }; f'
```