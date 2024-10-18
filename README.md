pre-commit is a widely-used code quality framework. It allows a developer to add hooks for various code quality tools that check your code for any errors or issues before committing them to a repository. Django adopted pre-commit in 2020, and for many developers, it is an indispensable tool used on basically every project.

In this tutorial, we will create a new Django project, install pre-commit, and configure three of the more popular Django-related hooks:

Black, the Python code formatter
isort, to sort and group imports properly
ruff, a Python linter written in Rust

Django Set Up
Let's go through the standard installation for a new Django project. On the command line, navigate to a new directory, create a new virtual environment, and install Django.

# Windows
$ python -m venv .venv
$ .venv\Scripts\Activate.ps1
(.venv) $ python -m pip install django~=5.0.0

# macOS
$ python3 -m venv .venv
$ source .venv/bin/activate
(.venv) $ python3 -m pip install django~=5.0.0
Show light mode
Create a new project called django_project and then run runserver to start up the local web server and view the Django welcome page in your web browser at 127.0.0.1:8000.

(.venv) $ django-admin startproject django_project .
(.venv) $ python manage.py runserver
Show light mode
Django Welcome Page


Install pre-commit
There are multiple ways to install pre-commit. In this tutorial, we will use pip, but some developers on macOS prefer installing it via Homebrew and updating it that way.

(.venv) $ python -m pip install pre-commit
Show light mode
To confirm it installed correctly, check its version:

(.venv) $ pre-commit version
pre-commit 3.3.3
Show light mode

Add a Configuration File
pre-commit is configured via a .pre-commit-config.yaml file located in the root of your directory. Create this new file now with your text editor.

├── .pre-commit-config.yaml  # new
├── db.sqlite3
├── django_project
│   ├── ...
└── manage.py
Show light mode
There is a helpful command to output a sample configuration.

(.venv) $ pre-commit sample-config
# See https://pre-commit.com for more information
# See https://pre-commit.com/hooks.html for more hooks
repos:
-   repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v3.2.0
    hooks:
    -   id: trailing-whitespace
    -   id: end-of-file-fixer
    -   id: check-yaml
    -   id: check-added-large-files
Show light mode
Briefly, this configuration references the source repository, version number, and contains four hooks that respectively remove whitespace, make sure text files end with a newline, check that YAML files are valid, and prevent large files (the default is 500kB) from being committed.

Copy and paste this into the .pre-commit-config.yaml file.

# .pre-commit-config.yaml
repos:
-   repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v3.2.0
    hooks:
    -   id: trailing-whitespace
    -   id: end-of-file-fixer
    -   id: check-yaml
    -   id: check-added-large-files
Show light mode
It is also possible--and highly recommended--to specify which version of Python pre-commit should be used to run hooks. This can be done via the default_language_version top-level key. We are using Python 3.11 here.

# .pre-commit-config.yaml
default_language_version:
  python: python3.11

repos:
-   repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v3.2.0
    hooks:
    -   id: trailing-whitespace
    -   id: end-of-file-fixer
    -   id: check-yaml
    -   id: check-added-large-files
Show light mode

Git setup
Make sure that git is already installed since we can't use pre-commit without git itself.

Confirm that git is installed by checking its version number.

(.venv) $ git --version
git version 2.39.2 (Apple Git-143)
Show light mode
Then add a .gitignore file to the root directory and include one line excluding the virtual environment directory, .venv.

# .gitignore
.venv
Show light mode
We can now initialize a new repository and add all changed files, but do not commit it yet.

(.venv) $ git init
(.venv) $ git add .
Show light mode

Install git Hook Scripts
Run the pre-commit install command to set up the git hook scripts in our repository.

(.venv) $ pre-commit install
pre-commit installed at .git/hooks/pre-commit
Show light mode
pre-commit will now run automatically on any future git commit command!

When adding new hooks, it is a good idea to run them against all files since usually, pre-commit only runs on changed files during git hooks.

(.venv)
$ pre-commit run --all-files
Trim Trailing Whitespace.................................................Passed
Fix End of Files.........................................................Failed
- hook id: end-of-file-fixer
- exit code: 1
- files were modified by this hook

Fixing .gitignore

Check Yaml...............................................................Passed
Check for added large files..............................................Passed
Show light mode
What's this? Our pre-commit hook noticed an error already: we failed to add an extra line to the end of the .gitignore file, so it flagged it as "failed" and then fixed it automatically. If you look at the .gitignore file, you'll see a second line now.

# .gitignore
.venv
Show light mode
Everything will pass if you run pre-commit run --all-files again.

(.venv) $ pre-commit run --all-files
Trim Trailing Whitespace.................................................Passed
Fix End of Files.........................................................Passed
Check Yaml...............................................................Passed
Check for added large files..............................................Passed
Show light mode
It's time to stage the changes to the .gitignore file and make our first git commit.

(.venv) $ git add .
(.venv) $ git commit -m "initial commit" 
Show light mode

Add hooks: Black
Black is a popular Python code formatter that checks all code against the PEP 8 standard. Django has formally adopted it per DEP 8. We can add it to our .pre-commit-config.yaml file as a hook.

# .pre-commit-config.yaml
default_language_version:
  python: python3.11

repos:
-   repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.4.0
    hooks:
    -   id: trailing-whitespace
    -   id: end-of-file-fixer
    -   id: check-yaml
    -   id: check-added-large-files
-   repo: https://github.com/psf/black
    rev: 23.7.0
    hooks:
    -   id: black
Show light mode
Make sure to include that extra line at the end of the file, or pre-commit will complain. Now add our changes to git. Since we are adding a new hook, it is necessary to run it initially against all files.

(.venv) $ git add .
(.venv) $ pre-commit run --all-files 
trim trailing whitespace.................................................Passed
fix end of files.........................................................Passed
check yaml...............................................................Passed
check for added large files..............................................Passed
black....................................................................Passed
Show light mode
Everything passed, so we can make a git commit with a message about adding a hook for Black.

(.venv) $ git commit -m "add Black hook"
trim trailing whitespace.................................................Passed
fix end of files.........................................................Passed
check yaml...............................................................Passed
check for added large files..............................................Passed
black................................................(no files to check)Skipped
[main b29d356] add black
 1 file changed, 4 insertions(+), 4 deletions(-)
Show light mode
Notice that Black "Skipped" running because the only change was to the .pre-commit-config.yaml file, and Black only applies to Python files.


pyproject.toml
A pyproject.toml file acts as a unified Python project settings file. Create a new blank file in your root project directory.

├── .pre-commit-config.yaml
├── db.sqlite3
├── django_project
│   ├── ...
├── manage.py
└── pyproject.toml  # new
Show light mode
We can use it to add another layer of configuration to our hooks. First, set a target-version for Python 3.11 within the pyproject.toml file. We can also specify a Python version for our Black hook.

# pyproject.toml
target-version = "py311"

[tool.black]
target-version = ["py311"]
Show light mode
Now git add the new pyproject.toml file and create a git commit.

(.venv) $ git add .
(.venv) $ git commit -m "add pyproject.toml file"
trim trailing whitespace.................................................Passed
fix end of files.........................................................Passed
check yaml...........................................(no files to check)Skipped
check for added large files..............................................Passed
black................................................(no files to check)Skipped
[main d247439] pyproject.toml
 1 file changed, 4 insertions(+)
 create mode 100644 pyproject.toml
Show light mode

Add hooks: isort
isort is a code formatter that sorts imports automatically. It is a very popular package and has been used by Django since 2015. To add it as a hook, update the .pre-commit-config.yaml file as follows:

# .pre-commit-config.yaml
default_language_version:
  python: python3.11

repos:
-   repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.4.0
    hooks:
    -   id: trailing-whitespace
    -   id: end-of-file-fixer
    -   id: check-yaml
    -   id: check-added-large-files
-   repo: https://github.com/psf/black
    rev: 23.7.0
    hooks:
    -   id: black
-   repo: https://github.com/pycqa/isort 
    rev: 5.12.0
    hooks:
    -   id: isort
Show light mode
Then run the new hook against all files.

(.venv) $ pre-commit run --all-files
trim trailing whitespace.................................................Passed
fix end of files.........................................................Passed
check yaml...............................................................Passed
check for added large files..............................................Passed
black....................................................................Passed
isort....................................................................Passed
Show light mode
Now it can be added to git and commited with a message.

(.venv) $ git add .
(.venv) $ git commit -m "add isort hook"
trim trailing whitespace.................................................Passed
fix end of files.........................................................Passed
check yaml...............................................................Passed
check for added large files..............................................Passed
black................................................(no files to check)Skipped
isort................................................(no files to check)Skipped
[main 8af5fa2] add isort hook
 1 file changed, 4 insertions(+)
Show light mode

Add hooks: ruff
Historically, flake8 was the Python linter of choice, but recently ruff has emerged, an alternative written in Rust and promising much faster speeds. I will avoid any deep discussion of flake8 vs ruff other than to say I appreciate both tools but will focus on ruff here. You can see ruff's take on comptability with Black in the docs.

Add ruff as follows to the pre-commit config file.

# .pre-commit-config.yaml
default_language_version:
  python: python3.11

repos:
-   repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.4.0
    hooks:
    -   id: trailing-whitespace
    -   id: end-of-file-fixer
    -   id: check-yaml
    -   id: check-added-large-files
-   repo: https://github.com/psf/black
    rev: 23.7.0
    hooks:
    -   id: black
        alias: autoformat
-   repo: https://github.com/pycqa/isort
    rev: 5.12.0
    hooks:
    -   id: isort
-   repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.0.285
    hooks:
    -   id: ruff
        alias: autoformat
        args: [--fix]
Show light mode
Then in pyproject.toml, we'll add some additional configuration to enable Pyflakes by default and ignore errors around line lengths (E501) and ambiguous file names (E741).

# pyproject.toml 
target-version = "py311"

[tool.black]
target-version = ["py311"]

[tool.ruff]
# Enable Pyflakes `E` and `F` codes by default.
select = ["E", "F"]
ignore = ["E501", "E741"] 
Show light mode
Run ruff against all files now.

(.venv) $ pre-commit run --all-files
trim trailing whitespace.................................................Passed
fix end of files.........................................................Passed
check yaml...............................................................Passed
check for added large files..............................................Passed
black....................................................................Passed
isort....................................................................Passed
ruff.....................................................................Passed
Show light mode
Then we can git add changes to the .pre-commit-config.yaml and pyproject.toml and make a git commit.

(.venv) $ git add .
(.venv) $ git commit -m "add ruff"
trim trailing whitespace.................................................Passed
fix end of files.........................................................Passed
check yaml...............................................................Passed
check for added large files..............................................Passed
black................................................(no files to check)Skipped
isort................................................(no files to check)Skipped
ruff.................................................(no files to check)Skipped
[main 979212f] add ruff
 2 files changed, 12 insertions(+)
