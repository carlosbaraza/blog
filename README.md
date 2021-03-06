# carlosbaraza.com Blog

## Development

In one terminal:

```sh
nvm use 12
cd server
ghost start
```

On another terminal:

```sh
nvm use 12
cd content/themes/blog-theme
npm i
npm run dev
```

## Development (Initial setup)

Install ghost in the local server folder and symlink to content folder.

```sh
mkdir server
cd server

# https://ghost.org/docs/install/local/
ghost install local

ln -s ../content content
```

Now you need to ensure that the submodule: https://github.com/carlosbaraza/blog-theme is initialized to `./content/themes/blog-theme`.

```bash
# Init blog-theme submodule
git submodule update --init
```

When changes are made to the blog theme, you have to commit them to that
sub repository. The reason for this is that it will make it easier to update
the theme with new updates of Casper theme.

## Local Login

Admin UI: http://localhost:2368/ghost
User: admin@admin.com
Password: 2rL4@TXd5L@8cqDb

## Updating Ghost
```
docker pull ghost:latest
docker tag ghost:latest dokku/carlosbaraza.com:latest
dokku tags:deploy carlosbaraza.com latest
```

## Initial Dokku deployment

Deployed to dokku. Inspired by [this article](https://matthisk.com/running-ghost-publishing-on-dokku/):

```bash
# Create Ghost app
dokku apps:create carlosbaraza.com
docker pull ghost:latest
docker tag ghost:latest dokku/carlosbaraza.com:latest
dokku tags:deploy carlosbaraza.com latest
dokku proxy:ports-add carlosbaraza.com http:80:2368

# Persist the content folder
dokku storage:mount carlosbaraza.com /opt/carlosbaraza.com/ghost/content:/var/lib/ghost/content
mkdir -p /opt/carlosbaraza.com/ghost/content

# Use HTTP because Cloudflare to Dokku would be set to flexible, unless
# origin certificates are configured in the future
dokku config:set carlosbaraza.com url=http://carlosbaraza.com

# Setup SSL certificate
# Cloudflare offers a free end-to-end SSL certificate. You need to set SSL config to full and download private and public keys from the dashboard. To install certificates on your Dokku NGINX run the following commands:
# vim carlosbaraza.com.crt (similar than .pem)
# vim carlosbaraza.com.key
tar cvf carlosbaraza.com.certificates.tar carlosbaraza.com.crt carlosbaraza.com.key
dokku certs:add carlosbaraza.com < carlosbaraza.com.certificates.tar

# Add domain
dokku domains:add carlosbaraza.com carlosbaraza.com

# Increase max upload size
mkdir /home/dokku/carlosbaraza.com/nginx.conf.d
echo 'client_max_body_size 50M;' > /home/dokku/carlosbaraza.com/nginx.conf.d/upload.conf
service nginx reload

# Connect MySQL
dokku mysql:create carlosbaraza_com
dokku mysql:link carlosbaraza_com carlosbaraza.com

# Configure Ghost database env variables
dokku config:set carlosbaraza.com database__client=mysql \
  database__connection__host=dokku-mysql-carlosbaraza-com \
  database__connection__user=mysql \
  database__connection__password=PASSWORD \
  database__connection__database=carlosbaraza.com
```
