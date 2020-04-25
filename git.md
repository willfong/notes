# Git

How I use git...

Using ZSH

1. `gco master` - Ensure you're on the master branch
1. `git pull` - Ensure you have all the latest updates
1. `gcb <name>` - Shortcut to create a new branch called `<name>` 
1. Make your changes
1. `git status` - Check to see which files have changed
1. `git diff <filename>` - Check each file to ensure proper changes
1. `git add <filename>` - Manually add each file to the checkout
1. `git status` - Check to make sure proper files have been added
1. `git commit -m "<some message>"` - Commit all the changes
1. `git push origin HEAD` - Push your changes to your branch in origin
1. The above step will provide a URL to create a Pull Request/Merge Request
1. `gco master` - Return back to master


## Tips

1. **Small and Frequent Commits** - Increment master with small commits that incremently improve the code base. Major code revisions should be avoided as possible to ensure uptime.


## External

- https://thoughtbot.com/blog/5-useful-tips-for-a-better-commit-message
- https://scottchacon.com/2011/08/31/github-flow.html


