# gitignore-per-branch

`gitignore-per-branch` is a small command-line utility that allows you have 
unique gitignore rules on a per-branch basis by installing a git hook in your 
local repository which automatically loads a branch-specific gitignore file 
each time you switch to that branch branches with `git checkout` or `git switch`.  

A common use would be to have content which is common to all branches but 
also have branch-specific directories ignored in some or all other branches.  

With the script installed and enabled, each branch will behave as expected in 
accordance to any `.gitignore` files, including any global `.gitignore`, but 
additional rules are added if a `.gitignore-<branchname>` file is found at the 
root of the project, matching the branch name is switched to.    

These `.gitignore-<branchname>` files can follow the same format as `.gitignore` 
or they can be executable in order allow referencing other `.gitignore*` files 
so one can copy or extend rules from other branches by referencing them. 

## Installation and Removal

Since git hooks do not get pushed to remote branches, it is important to  
include the `gitignore-per-branch` script within your repository.  

From your local repository's root directory, the following will add the script:  

```sh
% curl https://raw.githubusercontent.com/Jeff-Russ/gitignore-per-branch/main/gitignore-per-branch > ./gitignore-per-branch
% chmod 755 ./gitignore-per-branch
```

If you find yourself doing this a lot, for different repositories, create a 
reusable command that aliases this (for bash terminals):  

```sh
% echo "\n"'alias dl-gitignore-per-branch="curl https://raw.githubusercontent.com/Jeff-Russ/gitignore-per-branch/main/gitignore-per-branch > ./gitignore-per-branch && chmod 755 ./gitignore-per-branch"' >> ~/.bashrc 
```

For zsh terminals, do the same but with `~/.zshrc` in place of `/.bashrc`.  

To use the alias, just run the command `dl-gitignore-per-branch` from the 
root directory of local repository.  


## Usage

The `./gitignore-per-branch` script should be at the root of your repository, 
made executable with `chmod 755 ./gitignore-per-branch`. If you do not already 
have a git repository initialized, you then run the following command:  

```sh
% ./gitignore-per-branch --init
```

The above, if no repository is detected, is equivalent to the following:  

```sh
% git init
% ./gitignore-per-branch --enable
```

In distributing your repository, your README should instruct anyone who may 
switch branches to run the following command once after downloading or cloning:   

```sh
% ./gitignore-per-branch --enable
```

You should only need to run one of these two `./gitignore-per-branch` commands 
once, after which the installed git hook will do all the work in the background.  

As for creating new repositories, ideally one should plan ahead with the file 
structure and fully flesh out all `.gitignore-<branchname>` files *before* 
*creating any new branches*.

Note that changes to a `.gitignore-<branchname>` file are only observed while
switching branches. So, much like how you would `git add .gitignore` so  that 
`git add .` doesn't add things you've excluded, you would do the following
after changes to `.gitignore-<branchname>` files are made, as a means to force 
re-reading of it without switching to the branch:  

```sh
% ./gitignore-per-branch --update
```

## Executable `.gitignore-<branchname>` files

Another option for these `.gitignore-<branchname>` files allows you to copy 
or extend the rules from one or more other branches. If a 
[shebang](https://en.wikipedia.org/wiki/Shebang_(Unix)) is found is found 
as the first line, their standard output should follow `.gitignore` format.  

For example:  

```sh
#!/bin/sh
cat << EOF
$(cat .gitignore-main)
anotherdir/
andthis/
EOF
```

Beware that if `.gitignore-main` does not end with a blank line, you'll need 
one inserted, such as with `echo`, if you choose not to use HEREDOC:  

```sh
#!/bin/sh
cat .gitignore-main
echo
cat .gitignore-macOS
```

## Removal from a Repository

If you decide later than you do not want to use `gitignore-per-branch` in a given
repository, you can run:  

```sh
% gitignore-per-branch --remove # This will remove the hook added to .git/hooks/
% git rm ./gitignore-per-branch # optional, remove the script that added the hook
```

The `.gitignore-<branchname>` file can be removed from each branch but there is 
no issue with leaving them as they will have no effect.  
