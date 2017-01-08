# Deploy

Super Simple Deploys for side projects and tiny apps.

I've been looking for a good way to deploy simple, small node (and other) projects. This approach is nice since it's all achieved with a couple small bash scripts. But it does require quite a few config files and setting up the system in a certain way.

Which is why so far, I haven't been using this approach myself and instead switched to the awesome https://flynn.io/ - "Throw away the duct tape".

## Usage

On your server, you should:

    $ apt-get install nodejs nginx
    $ npm install -g pm2
    $ pm2 startup systemd -u app
    $ adduser --disabled-password --gecos "" app
    $ adduser app www-data
    $ chown -R root:www-data /etc/nginx/conf.d/
    $ chmod -R g+rwX /etc/nginx/conf.d/
    $ echo "app ALL=(ALL) NOPASSWD: /usr/sbin/nginx -s reload" > /etc/sudoers.d/app
    $ chmod 0440 /etc/sudoers.d/app

Ok, all ready. In your app now, create a deploy.conf:

```
[production]
user app
host 1.2.3.4
repo git@github.com:KidkArolis/foo.git
path /home/app/foo
ref origin/master
forward-agent yes
post-deploy npm install --production && ./node_modules/.bin/taco-nginx --port 3000 --public /home/app/foo/current/public && pm2 startOrRestart pm2.json --env production
```

Create a `pm2.json`:

```
{
  name: 'foo',
  script: `.`,
  cwd: '/home/app/foo/current',
  merge_logs: true,
  env_production: {
    NODE_ENV: 'production'
  }
}
```

Add an entry to your `~/.ssh/config` for key forwarding:

```
host 1.2.3.4
  ForwardAgent yes
```

And

    $ npm install --save KidkArolis/deploy
    $ ./node_modules/.bin/deploy production setup
    $ ./node_modules/.bin/deploy production

Point your DNS to this server `1.2.3.4`

And your app should be running at foo.yourdomain.com.

If you want to deploy a toplevel domain, pass `--domain yourdomain.com` to `taco-nginx`.

## Changes

Differences from http://github.com/tj/deploy:

* None

Differences from https://github.com/mafintosh/taco-nginx

* Doesn't run your app, only writes an nginx config file
* Takes a `--public` param to point to root for static files
* Doesn't use `sudo` for `mv /tmp/app.conf /etc/nginx/conf.d/app.conf`
