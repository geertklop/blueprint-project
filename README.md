# Project Blueprint Repo

## Use this cookiecutter blueprint

### Create new local project from blueprint

1. Clone this repo
1. Create a virtualenv `python -m venv .venv`
1. Activate the virtualenv `source .venv/bin/activate`
1. Install cookiecutter `pip install cookiecutter`
1. Move out of the repo (`cd ..`)
1. Use the template to create a new project `cookiecutter .\blueprint-repo` and follow the prompts.
1. Move into the newly created project

### Transfer local project to GIT

1. Create a remote GIT project
1. Inside your local project, initialize a git repo (`git init`)
1. Add the remote `git remote add <REMOTE>`
1. Stage all the files `git add .`
1. Commit the files `git commit -am "Copied files from project blueprint"`
1. Push to remote `git push -u origin`
