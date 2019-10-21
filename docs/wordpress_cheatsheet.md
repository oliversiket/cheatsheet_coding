# WordPress Git Initialize

## Commands we'll need

### Option 1: first commit on GitHub - // use this option if you committing your changes online regularly

```bash
# run as separate commands
$ git init
$ git remote add origin {repository URL, starts git@github.com….}
$ git fetch
$ git checkout master
$ git add {filename}
$ git commit -m "adding my first files"
$ git push
```

### Option 2: first commit on your computer // use this option if you keep working locally and commit it later

```bash
# run as separate commands
$ git init
$ git remote add origin {repository URL}
$ git add {filename}
$ git commit -m "adding my first file"
$ git push 
```