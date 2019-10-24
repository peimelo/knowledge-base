# Untitled

## Install

```bash
$ /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

When it asks you to install the Xcode command line tools, say yes.

## Rbenv

```bash
$ brew install rbenv ruby-build

$ echo 'eval "$(rbenv init -)"' >> ~/.bash_profile
$ echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bash_profile

# restart the session
$ source ~/.bash_profile

# list all available versions:
$ rbenv install -l

# install a Ruby version:
$ rbenv install 2.6.5

# Sets the global version of Ruby to be used in all shells
$ rbenv global 2.6.5

# Verify that rbenv is properly set up using this rbenv-doctor script:
$ curl -fsSL https://github.com/rbenv/rbenv-installer/raw/master/bin/rbenv-doctor | bash

# Uninstalling Ruby versions
$ rbenv uninstall 2.6.5
```

