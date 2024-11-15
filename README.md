# Docker Symfony Starter kit

- [Docker Symfony Starter kit](#docker-symfony-starter-kit)
  - [Dependencies](#dependencies)
  - [First install](#first-install)
    - [Change localhost to a custom URL](#change-localhost-to-a-custom-url)
  - [Troubleshooting](#troubleshooting)
    - [Port 80 already used](#port-80-already-used)
    - [Use a prebuilt Docker image](#use-a-prebuilt-docker-image)
    - [Change database connection string](#change-database-connection-string)
  - [Accessing services](#accessing-services)
  - [Add a new endpoint](#add-a-new-endpoint)
  - [Docker compose cheatsheet](#docker-compose-cheatsheet)
  - [Upgrades](#upgrades)
    - [Symfony](#symfony)
    - [PHP](#php)
  - [Recommendations](#recommendations)
  - [Notes](#notes)
    - [Nginx](#nginx)
    - [Files location](#files-location)

## Dependencies

- Latest Docker engine: <https://docs.docker.com/engine/install/>
- Latest Docker compose plugin <https://docs.docker.com/compose/install/linux/>

Containers created in a base of [phpdocker.io](https://phpdocker.io).

  | Services installed in this boilerplate | Installation container |
  |----------------------------------------|------------------------|
  | PHP & PHP-FPM                          | php                    |
  | Composer                               | php                    |
  | Node & NPM                             | php                    |
  | NGINX                                  | nginx                  |
  | MySQL                                  | mysql                  |
  | PHPMyAdmin                             | phpmyadmin             |
  | MailHog                                | mailhog                |

## First install

Clone this repository (`git clone`) and change directory (`cd`) to get into it.  
_Optionally: change in the variable values in the `.env` file to your project needs._

Build the Docker containers: `docker compose build`.  
Once this is done, you can run the containers: `docker compose up -d`.  

**At the first launch** after building the containers:

- Install dependencies by running: `docker compose exec php bash -c "cd sfstarterkit && composer install"`
- Wait a few seconds for MySQL to launch properly

The project is now available if you go to the URL `localhost`!  
_(if not refer to the `Troubleshooting` section below)_

### Change localhost to a custom URL

If you want to have a better URL, you can add the following line to your hosts file, if you are using Windows WSL it is located in `/c/Windows/System32/drivers/etc/hosts`.  
Just append this `127.0.0.1 sfstarterkit.local` (or your chosen `PROJECT_URL` in your `.env`).  
Then the project will be available on the URL you set in the PROJECT_URL line of your .env file !  

You can put your project files at the root of this boilerplate.

## Troubleshooting

### Port 80 already used

If your port `80` is in use just change nginx port (ex: `8084:80`) into `docker-compose.yml` (line 25).  
By the way your host will be `localhost:8084` and not `localhost` (or `sfstarterkit.local:8084`).

### Use a prebuilt Docker image

In case you do not want to build your PHP image locally using the `.docker/php/Dockerfile`, just do a `git checkout dockerhub`.  
This branch use a remote image from Docker Hub: `gabrielrossetgravito/docker-starter-php:8.2`.  
After running `docker compose up -d` just install dependencies with `docker compose exec php bash -c "cd sfstarterkit && composer update"`.

### Change database connection string

If you changed the database address in the `.env` file. So you have to replace it also in `sfstarterkit/.env`:  
`DATABASE_URL="mysql://${MYSQL_USER}:${MYSQL_PASSWORD}@mysql:3306/${DATABASE_NAME}"`

## Accessing services

To access PHPMyAdmin, just go to the port `81` of your project.  
To access MailHog, just go to the port `82` of your project.  
To access Composer, Node or NPM, you must enter the PHP container: `docker compose exec php bash`. You will then be able to do your Composer/Node/NPM commands.  

Be careful, the IP address of the database is the name of the container. The address to indicate in the configuration files of your app is therefore: `mysql`.  

## Add a new endpoint

Just use the command below:

```sh
docker compose exec php bash -c "cd sfstarterkit && symfony console make:controller"
```

Then create for example a controller named: `HelloController`.
That's all you have now access to your new endpoint: `http://localhost/hello` (or `http://sfstarterkit.local/hello`).

## Docker compose cheatsheet

- Start the containers by watching their logs: `docker compose up`
- Start the containers in the background: `docker compose up -d`
- Stop the containers: `docker compose stop`
- Kill the containers: `docker compose kill`
- Delete the containers: `docker compose rm`
- Stop and delete the containers: `docker compose down`
- Check the status of the containers: `docker compose ps`
- Watch the container logs: `docker compose logs`
- Check CVE: `docker compose exec php bash -c "cd sfstarterkit && symfony check:security"`
- List CLI commands: `docker compose exec php bash -c "cd sfstarterkit && symfony console list make"`
- Making a command in a container: `docker compose exec CONTAINER_NAME COMMAND` where `COMMAND` is your command. Examples:  
  - Open a console in the php-fpm container: `docker compose exec php bash`
  - Open the Symfony console: `docker compose exec php bin/console`

## Upgrades

### Symfony

This stater kit goes with Symfony 7.1.  
If you want to install another Symfony version, just do `rm -rf sfstarterkit` then change the `7.1.*` into the command below:

```sh
docker compose exec php symfony new sfstarterkit --version="7.1.*" --webapp
```

Run the command, now you can find a folder called `sfstarterkit`, and your Symfony version is inside.  

### PHP

To upgrade PHP, just change the `FROM` instruction into `.docker/php/Dockerfile`.
Just be careful about:

- `php-ini-overrides.ini` volume path into `docker-compose.yml`
- requirements from your `${PROJECT_NAME}/composer.json`.
- updating your libraries with `docker compose exec php composer update`

## Recommendations

It's hard to avoid file permission issues when fiddling about with containers due to the fact that, from your OS point of view, any files created within the container are owned by the process that runs the docker engine (this is usually root). Different OS will also have different problems, for instance you can run stuff in containers using `docker exec -it -u $(id -u):$(id -g) CONTAINER_NAME COMMAND` to force your current user ID into the process, but this will only work if your host OS is Linux, not mac. Follow a couple of simple rules and save yourself a world of hurt.  

- Run composer outside of the php container, as doing so would install all your dependencies owned by `root` within your vendor folder
- Run commands (ie Symfony's console) straight inside of your container. You can easily open a shell as described above and do your thing from there

## Notes

### Nginx

Ensure the webserver config on `docker\nginx.conf` is correct for your project. phpdocker.io will have customised this file according to the application type you chose on the generator, for instance `web/app|app_dev.php` on a Symfony project, or `public/index.php` on generic apps.

### Files location

You may place the files elsewhere in your project. Make sure you modify the locations for the php-fpm dockerfile, the php.ini overrides and nginx config on `docker compose.yml` if you do so.  
