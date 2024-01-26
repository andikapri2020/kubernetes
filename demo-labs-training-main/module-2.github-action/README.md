# Github Action

## Git

### Config Git
git-config - Get and set repository or global options

config with name and email
```
git config --global user.name "username"
git config --global user.email "user@mail"
```
check config
```
git config --global --list
```

location config
```
cat $HOME/.gitconfig
```

### Initialize Repository
git-init - Create an empty Git repository or reinitialize an existing one

Create new repository
```
git init new-repo
```

Init existing repository
```
git init exist-repo
```

check & verify git repository
```
ls -la repository
```

the output will show hidden folder .git
```
.git
```

### Managing Contents
#### Check status repository
git-status - Show the working tree status

check repository
```
git status
```
#### Add file to repository
git-add - Add file contents to the index

Add file
```
git add file
```
#### Add recursive file to repository
Add multiple file and folder
```
git add .
```
#### Save file to repository
git-commit - Record changes to the repository
```
git commit -m "messages"
```
#### Ignoring file
gitignore - Specifies intentionally untracked files to ignore

create .gitignore file with text editor
```
.gitignore
```
#### Rename and Moving file
git-mv - Move or rename a file, a directory, or a symlink

rename file
```
git mv old-file new-file
```

moving file
```
git mv source-file dest-file
```
#### Delete file
git-rm - Remove files from the working tree and from the index

delete file from repository and from working directory
```
git rm file
```

delete file from repository without delete on working directory
```
git rm --cache file
```
### Undo Changes
#### On Staging
git-reset - Reset current HEAD to the specified state

Unstagged file
```
git reset --stagged file
```

discard changed file
```
git reset file
```
#### On Repository
git-revert - Revert some existing commits

revert commit change
```
git revert -n commit-id
```
apply revert
```
git revert --continue
```
### Branching and Merging
#### Branch
git-branch - List, create, or delete branches

list branch
```
git branch
```

create branch
```
git branch branch-name
```

switch branch
```
git switch branch-name
```

delete branch
```
git branch -d branch-name
```
#### Show changes
git-diff - Show changes between commits, commit and working tree, etc

Show changes dev-branch and main-branch
```
git diff dev main
```

#### Merge
git-merge - Join two or more development histories together

merge dev-branch to main-branch
```
git switch main
git merge dev
```
#### Resolved Conflict
Fix manually and then commit changes
### Tagging
Tagging

List tag
```
git tag
```

Create tag
```
git tag -a versi -m "message" commit-id
```

Delete tag
```
git tag -d versi
```

## Github
### Create Account Github
create account on [here](https://github.com/)

### Git Remote
#### Working with Github
list remote
```
git remote -v
```

add remote
```
git remote add name-remote link-repository
```

delete remote
```
git remote remove name-remote
```

Push git repository to Github
```
git push remote branch
```

Fetch git repository github to local
```
git fetch remote branch
```

Pull git repository github to local
```
git pull remote branch
```

## Github Action 
### Setup Agent Node (Runner)
Download
```
# Create a folder
$ mkdir actions-runner && cd actions-runner
# Download the latest runner package
$ curl -o actions-runner-linux-x64-2.311.0.tar.gz -L https://github.com/actions/runner/releases/download/v2.311.0/actions-runner-linux-x64-2.311.0.tar.gz
# Optional: Validate the hash
$ echo "29fc8cf2dab4c195bb147384e7e2c94cfd4d4022c793b346a6175435265aa278  actions-runner-linux-x64-2.311.0.tar.gz" | shasum -a 256 -c
# Extract the installer
$ tar xzf ./actions-runner-linux-x64-2.311.0.tar.gz
```

Configure
```
# Create the runner and start the configuration experience
$ ./config.sh --url https://github.com/IDN-Training/devops-training-idn --token AQCGKIRDUXD65Z36DAOW7DTFN4VMA
# Last step, run it!
$ ./run.sh
```

Using your self-hosted runner as service
```
# Install
./svc.sh install USERNAME

# Start
sudo ./svc.sh start

# Check Status
sudo ./svc.sh status
```