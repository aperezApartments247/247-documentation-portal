# Introduction
- This guide will help in getting a local CMS environment set up on `MacOS Catalina: Version 10.15.6`
- This is working as of 09/09/2023

# Tools
* [Homebrew](https://brew.sh/)
* [PostgreSQL 14](https://formulae.brew.sh/formula/postgresql)
* [Pyenv](https://formulae.brew.sh/formula/pyenv) - A python version manager
* [Pyenv-virtualenv](https://formulae.brew.sh/formula/pyenv-virtualenv) - A python virtualenv manager
* [RabbitMQ](https://formulae.brew.sh/formula/rabbitmq) - Required for celery
* [MailHog](https://formulae.brew.sh/formula/mailhog) - SMTP

# Pre-Setup
_By default, MacOS Catalina comes with `zsh`._
- Add this to your `~/.zshrc` if you're using `zsh` (or `~/.bash_profile` if you're using `bash`.)
```
# Pyenv
export PATH="$HOME/.pyenv/bin:$PATH"
export PYENV_VIRTUALENV_DISABLE_PROMPT=1
eval "$(pyenv init -)"
eval "$(pyenv virtualenv-init -)"

# PostgreSQL 14
PATH="/usr/local/opt/postgresql@14/bin:$PATH"

# Get unique paths
export PATH=`echo $PATH | tr ':' '\n' | sort | uniq | tr '\n' ':' | sed 's/.$//'`
```
- Source the config
```
source ~/.zshrc

or 

source ~/.bash_profile
```

# Installations
**PostgreSQL 14**
```
$ brew install postgresql@14
$ brew install postgis
$ brew install gdal
$ brew install geos
$ brew services stop postgresql*
$ brew services start postgresql
 
To verify: 
$ psql -V
psql (PostgreSQL) 14.7
```

**Python 3.10+ environment**
```
$ brew install pyenv
$ brew install pyenv-virtualenv
$ pyenv install 3.10-dev
$ pyenv global 3.10-dev
$ pyenv virtualenv 3.10-dev cms-env

To verify: 
$ pyenv -v
pyenv 2.3.2

$ python3 -V
Python 3.10+

$ pyenv virtualenvs
  3.10-dev/envs/cms-env (created from /Users/username/.pyenv/versions/3.10-dev)
  cms-env (created from /Users/username/.pyenv/versions/3.10-dev)
```

**RabbitMQ**
```
$ brew install rabbitmq
$ brew services start rabbitmq

To verify: 
http://localhost:15672
```

**MailHog**
```
$ brew install MailHog
$ brew services start mailhog

To verify: 
http://localhost:8025/
```

# Setup
_The folder structure defined in this guide will be specific to this setup. Any path modifications will be your own responsibility._ 
- **PSQL**
  - Create your user, password, and database.
    - _Remember the user and password for your `local.py`_
    ```
    $ sudo -u postgres psql
    $ CREATE USER myuser WITH PASSWORD 'mypassword';
    $ ALTER USER myuser CREATEDB;
    $ \q
    $ createdb -U myuser cms
    ```
  - Add the extensions **(Note: Not needed for Python 3+)**
    ```
    $ psql -d cms -c "CREATE EXTENSION postgis"
    $ psql -d cms -c "CREATE EXTENSION hstore"
    ```
  - Forgot your password? (Remember ChatGPT is your friend):
    <img width="777" alt="reset-mac-pass" src="https://github.com/Apartments24-7/247/assets/24421412/26510e36-4a4e-421b-8526-ac0e5b5c4929">

- **CMS**
  - Repo
  ```
    $ mkdir -p ~/git/cms
    $ git clone git@github.com:Apartments24-7/247.git ~/git/cms/src
  ```
  - Packages
  ```
    $ cd ~/git/cms/src
    $ pyenv activate cms-env
    $ pip install -r requirements.txt
  ```
  - Fieldkeys
  ```
    $ mkdir -p ~/git/cms/src/fieldkeys
    $ cd ~/git/cms/src/
    $ keyczart create --location=fieldkeys --purpose=crypt
    $ keyczart addkey --location=fieldkeys --status=primary --size=256
  ```
  - Pull latest db in to your local machine
    - _Note: {USERNAME} will have to be created by a TomServo admin_
  ```
    $ mkdir -p ~/git/cms/db
    $ FIND_DB='find /data/db -name "`ls -St /data/db | head -1`"'
    $ LATEST_CMS_DB=$(ssh -p 2224 {USERNAME}@office.apartments247.com $FIND_DB)
    $ scp -P 2224 {USERNAME}@office.apartments247.com:$LATEST_CMS_DB ~/git/cms/db/.
  ```
  - Reimport the local db
  ```
    $ cd ~/git/cms/db
    $ ls -St | head -1 | xargs cat | zcat | psql -U $USER -f - cms
  ```
  - `local.py` setup
    - `$ touch ~/git/cms/src/247/settings/local.py` 
    - Get an example [here](https://github.com/Apartments24-7/247/wiki/CMS-Dev-Setup:-local.py)
    - add (path may be different): `GDAL_LIBRARY_PATH = "/opt/homebrew/opt/lib/libgdal.dylib"`
    - add (path may be different): `GEOS_LIBRARY_PATH = "/opt/homebrew/opt/lib/libgeos.dylib"`
 
# Common commands
- CMS Environment
  - `pyenv activate cms-env`
- CMS Path
  - `cd ~/git/cms/src/247`
- CMS Runserver
  - `python manage.py runserver`
- CMS Migrate
  - `python manage.py migrate --noinput`
- CMS Shell
  - `python manage.py shell_plus --quiet`
- CMS Celery
  - `celery -A celery_init worker --loglevel=debug --purge`