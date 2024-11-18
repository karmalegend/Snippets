# Git aliases 

Some handy everyday git aliases.


### zsh
```
[alias]
	getremote = !git config --get remote.origin.url | awk -F: '{print \"https://github.com/\"$2}'
	refresh = !git fetch && git pull
	new = "!sh -c 'default_branch=$(git for-each-ref --format=\"%(refname:short)\" refs/heads refs/remotes/origin | grep -E \"^(main|master)$\" | head -n 1); [ -z \"$default_branch\" ] && { echo \"Error: No main or master branch found locally or remotely.\"; exit 1; }; git stash push -q && git checkout \"$default_branch\" && git pull origin \"$default_branch\" && git checkout -b \"$@\" && git stash pop -q || true' -"

```


