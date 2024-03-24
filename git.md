# Git and Pull Requests

Git and Pull Requests (PR) are a powerful collaboration tool to ensure development with speed and accuracy. PRs make it super easy for a second-set of eyes to review the intention of the change. By following a common process, we can greatly reduce production level incidents.

## Guiding Principles

Principles are important to ensure everyone on the team has the same understanding. Every team will have different principles depending on the environment, but they need to be specified.

1. Everything in `main` is deployable to production. If it's in main, assume it's in Production.
1. PRs must be reviewed to ensure the intention of the change is correctly implemented.

## Command Line Process

For each new feature or change you want to make, create a new branch using the below steps:

1. `git checkout main` - Ensure you're on the main (primary) branch
1. `git pull` - Ensure you have all the latest updates
1. `git checkout -b <name>` - Shortcut to create a new branch called `<name>`
1. Make your changes
1. `git status` - Check to see a list of all the files that have been changed
1. `git diff <filename>` - Run this for each file to see the differences between what's in the main repository and the changes you made
1. `git add <filename>` - Stage each file to be committed
1. `git status` - Review the list of files that have been staged to be committed
1. `git commit -m "<some message>"` - Commit all the changes with a short description of the change
1. `git push origin HEAD` - Push your changes to your branch in origin
1. The above step will provide a URL to create a Pull Request/Merge Request
1. Follow to the link to create your actual pull request
1. `git checkout main` - Return back to main branch

Optionally, use can add shortcuts to your terminal to save you from typing:

```sh
alias gco='git checkout'
alias gcb='git checkout -b'
```

## Tips

1. **One PR per change** - Each pull request should have one scope / feature change. A large PR with multiple scopes take longer to merge and can hold up other PRs.
1. **Small and Frequent Commits** - Small commits are easier to review and ensure they do the right thing. This facilitates continuous reviews, and ensures the project is going in the right direction.
1. **Manually Add Files** - Add each file to your commit with intention. This helps ensure you only check in the files that are supposed to be in the commit. Don't add the entire directory.
1. **Prefix Branch Names** - Personal opinion, I like to prefix my branch names with my initials `wf-` to make it super simple to identify my branches, like `wf-support-new-endpoint`.

## External Links

- https://thoughtbot.com/blog/5-useful-tips-for-a-better-commit-message
- https://scottchacon.com/2011/08/31/github-flow.html
