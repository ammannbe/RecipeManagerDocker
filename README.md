# RecipeManagerDocker

This is a starter-kit repo for an easy installation with docker.

## Getting Started

Get the latest [release](https://github.com/ammannbe/RecipeManagerDocker) or clone the repo with

```bash
git clone https://github.com/ammannbe/RecipeManagerDocker.git
```

## Installation

- Copy the .env.example files and edit them to your needs:

```bash
# Global:
cp .env.example .env
nano .env

# Api:
cp ./api/.env.example ./api/.env
chown 1000:1000 ./api/.env
nano ./api/.env

# Web:
cp ./web/.env.example ./web/.env
nano ./web/.env
```

- For help about the .env variables see
  - [RecipeManagerApi > Docker > README.md](https://github.com/ammannbe/RecipeManagerApi/blob/8.x/storage/docker/README.md)
  - [RecipeManagerWeb > Docker > README.md](https://github.com/ammannbe/RecipeManagerWeb/blob/8.x/storage/docker/README.md)
- IMPORTANT: make sure that you changed the `APP_ENV` to `production`
- Edit the docker-compose.yml to your needs:
- Optional: automatically start the container after system boot with systemd:
- Copy or symlink the [recipe-manager.service](recipe-manager.service) service file to `/etc/systemd/system/recipe-manager.service`.
- Enable the service: `systemctl enable recipe-manager.service`
- Start the service: `systemctl start recipe-manager.service`

## Create the first user

If you have the registration disabled, create the first user via CLI:

- Start the PHP tinker CLI

```bash
docker-compose exec api php artisan tinker
```

- Create the user and give admin rights

```php
$name = '<YOUR FULL NAME>';
$email = '<YOUR EMAIL ADDRESS>';
$password = '<YOUR PASSWORD>';

$user = User::create([
  'email' => $email,
  'password' => \Hash::make($password),
]);
$user->admin = true; $user->update();
$user->author()->create(['name' => $name]);
$user->markEmailAsVerified();
```

# docker-compose: Files you can overwrite

With the _volumes:_ option you can overwrite every file/folder from the containers.

Example:

```bash
services:
  app:
    ...
    volumes:
      - ./api/.env:/var/www/html/.env
      - ./api/entrypoint.sh:/usr/local/bin/entrypoint.sh
      - ./api/html:/var/www/html
      - ./api/ports.conf:/etc/apache2/ports.conf
      - ./api/data/app:/var/www/html/storage/app
      - ./shared/data/favicon.ico:/var/www/html/public/favicon.ico

  db:
    ...
    volumes:
      - ./data/api/mysql:/var/lib/mysql
```

In the container _web_ you can override following paths:

- **./web/.env:/usr/src/recipe-manager/.env** This folder contains your database
- **./web/entrypoint.sh:/usr/local/bin/entrypoint.sh** This script is executet at the ENTRYPOINT (see Dockerfile)
- **./shared/data/favicon.ico:/usr/src/recipe-manager/static/favicon.ico** Specify a custom favicon

In the containers _app_, _scheduler_, _queue_ you can override following paths:

- **./api/.env:/var/www/html/.env** This is the environement variable file
- **./api/entrypoint.sh:/usr/local/bin/entrypoint.sh** This script is executet at the ENTRYPOINT (see Dockerfile)
- **./api/html:/var/www/html** This is the folder containing all project files
- **./api/ports.conf:/etc/apache2/ports.conf** Specify a custom Apache HTTP port
- **./api/data/app:/var/www/html/storage/app** This folder contains user data like recipe images
- **./shared/data/favicon.ico:/var/www/html/public/favicon.ico** Specify a custom favicon

In the container _db_ you can override following paths:

- **./api/data/mysql:/var/lib/mysql** This folder contains your database

In the container _meilisearch_ you can override following paths:

- **./api/data/meilisearch:/data.ms** This folder contains your indexed db models for searching
