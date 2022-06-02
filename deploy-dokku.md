# Dokku Metabase

## Installing Docker & Dokku

```shell
# Installing Docker
apt remove docker docker-engine docker.io containerd runc
apt update
apt install -y apt-transport-https ca-certificates curl gnupg-agent software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add - apt-key fingerprint 0EBFCD88
apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
apt update
apt install -y docker-ce docker-ce-cli containerd.io

# To test Docker installation, run:
docker run --rm hello-world

# Installing Dokku
wget -nv -O - https://packagecloud.io/dokku/dokku/gpgkey | apt-key add -
export SOURCE="https://packagecloud.io/dokku/dokku/ubuntu/"
export OS_ID="$(lsb_release -cs 2>/dev/null || echo "bionic")"
echo "xenial bionic focal" | grep -q "$OS_ID" || OS_ID="bionic"
echo "deb $SOURCE $OS_ID main" | tee /etc/apt/sources.list.d/dokku.list
apt update
apt install -y dokku
dokku plugin:install-dependencies --core

# Installing Dokku plugins
sudo dokku plugin:install https://github.com/dokku/dokku-postgres.git postgres
sudo dokku plugin:install https://github.com/dokku/dokku-letsencrypt.git
```

## Configuring Dokku

You must add your SSH public key to Dokku so you can deploy applications to it
using git. Put your public key inside a file and run:

```shell
dokku ssh-keys:add admin path/to/pub_key
```

## Creating the Application

On Dokku server, set the correct values for the variables:

```shell
export ADMIN_EMAIL="admin@example.com"
export ADMIN_NAME="Admin"
export APP_NAME="metabase"
export DOMAIN="$APP_NAME.example.com"
export PG_NAME="pg_$APP_NAME"
```

Now, run the commands to create and configure the app:

```shell
# Create app
dokku apps:create $APP_NAME
dokku checks:disable $APP_NAME

# Config app
dokku config:set --no-restart $APP_NAME HEROKU=true JAVA_OPTS="-Xmx12g -Xss512k -XX:CICompilerCount=2 -Dfile.encoding=UTF-8"
dokku config:set --no-restart $APP_NAME DOKKU_LETSENCRYPT_EMAIL=$ADMIN_EMAIL
dokku domains:add $APP_NAME $DOMAIN

# Create database services
dokku postgres:create $PG_NAME

# Link database server to app
dokku postgres:link $PG_NAME $APP_NAME

# Add buildpacks
dokku buildpacks:add $APP_NAME https://github.com/heroku/heroku-buildpack-jvm-common.git
dokku buildpacks:add $APP_NAME https://github.com/metabase/metabase-buildpack

## First Deployment

Since you've created and configured the app on the server, you can switch to
your local machine and execute the first app deploy! After cloning the
repository (and inside its directory), execute:

```shell
git remote add dokku dokku@<server-ip>:<app-name>
git push dokku master
```

## Finishing Installation

To finish the installtion you need to configure the SSL certificate, mount the
data path and import

Finally, on server again:

```shell
# Create/configure SSL certificate:
dokku letsencrypt:enable $APP_NAME
```

## Importing data

TODO
