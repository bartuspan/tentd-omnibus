# tentd Omnibus

Bundled collection of [status](https://github.com/tent/tent-status)
and [admin](https://github.com/tent/tent-admin) apps and the [0.3 reference server](https://github.com/tent/tentd).

tentd is an alpha implementation of a Tent Protocol server. It currently contains **broken code, many bugs, and security flaws**. The code should only be used to experiment with how Tent works. Under no circumstances should the code in its current form be used for data that is supposed to be private.

## Setup

### ENV

Name           | Required | Description
--------       | -------- | -----------
USERNAME       | Required | Username you want to signin with
PASSPHRASE     | Required | Passphrase you want to signin with (must be encrypted through `rake encrypt_passphrase[passphrase]`)
URL            | Required | URL pointing to this app
SESSION_SECRET | Required | Random string for session cookie secret
DATABASE_URL   | Required | Postgres database url
REDIS_URL      | Required | Redis url
TENT_ENTITY    | Optional | Defaults to the mounted path of tentd
API_ROOT       | Optional | Defaults to TENT_ENTITY, must point to where tentd is mounted.
ASSET_ROOT     | Optional | Defaults to `/assets`.
PATH_PREFIX    | Optional | Mounts all components with the prefix.

**tentd**

See the [tentd README](https://github.com/tent/tentd/blob/master/README.md) for details and additional configuration options.

**status**

See the [status README](https://github.com/tent/tent-status/blob/master/README.md) for details and additional configuration options.

**admin**

See the [admin README](https://github.com/tent/tent-admin/blob/master/README.md) for details and additional configuration options.

### Running it anywhere

#### Dependencies

##### Ruby

This app requires a Ruby 1.9.3 or 2.0.0 environment.

**OS X**

[Homebrew](http://mxcl.github.io/homebrew/) is the easiest method of installing Ruby on OS X.

```shell
brew install ruby
```

If you need to switch between Ruby versions, [chruby](https://github.com/postmodern/chruby) and [ruby-install](https://github.com/postmodern/ruby-install) are well worth considering.

**Ubuntu**

```shell
sudo apt-get install build-essential ruby1.9.1-full libxml2 libxml2-dev libxslt1-dev
sudo update-alternatives --config ruby # make sure 1.9 is the default
```

##### Postgres

A Postgres database is required for running [tentd](https://github.com/tent/tentd).

If you use [Homebrew](http://mxcl.github.io/homebrew/), run

```shell
brew install postgresql
```

Otherwise, use [Postgres.app](http://postgresapp.com/).

##### Redis

[Tentd](https://github.com/tent/tentd) also requires [Redis](http://redis.io).

If you use [Homebrew](http://mxcl.github.io/homebrew/), run

```shell
brew install redis
```

Otherwise see [redis.io/download](http://redis.io/download).

##### JavaScript Runtime

Asset compilation depends on a JavaScript runtime such as node.

#### Booting it up

Assuming you have the appropriate ENV variables set,

```shell
git clone git://github.com/tent/tentd-omnibus
bundle
bundle exec rake assets:precompile
createdb tentd
bundle exec rake db:migrate
bundle exec ruby ./runner.rb
```

will start an [instance of tentd](http://localhost:8080/tent) with
pre-compiled and pre-authed instances of [status](http://localhost:8080/status) and [admin](http://localhost:8080/admin).

### Running on Heroku

See the [Getting Started with Heroku](https://devcenter.heroku.com/articles/quickstart) guide for more information.

Assuming you have heroku configured, just copy and paste the following:

```shell
heroku create --addons heroku-postgresql:dev,rediscloud:25,memcachier:25
heroku pg:promote $(heroku pg | head -1 | cut -f2 -d" ")
heroku labs:enable user-env-compile
heroku config:add \
 SESSION_SECRET=$(openssl rand -hex 16 | tr -d '\r\n') \
 USERNAME=admin \
 URL=$(heroku info -s | grep web_url | cut -f2 -d"=" | sed 's/http/https/' | sed 's/\/$//') \
 REDIS_URL=$(heroku config:get REDISCLOUD_URL | tr -d '\r\n') \
 RUN_SIDEKIQ=true
heroku config:add PASSPHRASE=$(heroku run bundle exec rake encrypt_passphrase\[passphrase\] | tail -1 | tr -d '\r\n')
git push heroku master
heroku run bundle exec rake db:migrate
heroku open
```
