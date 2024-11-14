# Docker Symfony Starter kit

- [Docker Symfony Starter kit](#docker-symfony-starter-kit)
  - [Dependencies](#dependencies)
  - [First install](#first-install)
  - [Accessing services](#accessing-services)
  - [Docker compose cheatsheet](#docker-compose-cheatsheet)
  - [Recommendations](#recommendations)
  - [Notes](#notes)

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

Fill in the variable names in the `.env` file specific to your project.  

Build the Docker containers: `docker compose build`.  
Once this is done, you can run the containers: `docker compose up -d`.  

If your port `80` is in use just change nginx port into `docker-compose.yml` (line 25).

At the first launch after building the containers, wait a few seconds for MySQL to launch properly.  

This stater kit goes with Symfony 7.1.  
If you want to install another Symfony version, just do `rm -rf sfstarterkit` then change it using the command below:

```sh
docker compose exec php symfony new sfstarterkit --version="7.1.*" --webapp
```

Now you can find a folder called `sfstarterkit`, and Symfony is inside. Just move the files from this folder to the boilerplate base and delete the `sfstarterkit` folder because it is now empty. Make sure to not replace the .env file, you have to merge it to keep the contents of the 2 .env files.  

If you set the database address in the `.env` file. So you have to replace it also in `sfstarterkit/.env`:
`DATABASE_URL="mysql://${MYSQL_USER}:${MYSQL_PASSWORD}@mysql:3306/${DATABASE_NAME}"`

The project is now available if you go to the URL `localhost` !  
If you want to have a better URL, you can add the following line to your hosts file, if you are using Windows WSL it is located in `/c/Windows/System32/drivers/etc/hosts`.  
Just append this `127.0.0.1 sfstarterkit.local` (or your chosen `PROJECT_URL` in your `.env`).  
Then the project will be available on the URL you set in the PROJECT_URL line of your .env file !  

You can put your project files at the root of this boilerplate.

## Accessing services

To access PHPMyAdmin, just go to the port `81` of your project.  
To access MailHog, just go to the port `82` of your project.  
To access Composer, Node or NPM, you must enter the PHP container: `docker compose exec php bash`. You will then be able to do your Composer/Node/NPM commands.  

Be careful, the IP address of the database is the name of the container. The address to indicate in the configuration files of your app is therefore: `mysql`.  

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

## Recommendations

It's hard to avoid file permission issues when fiddling about with containers due to the fact that, from your OS point of view, any files created within the container are owned by the process that runs the docker engine (this is usually root). Different OS will also have different problems, for instance you can run stuff in containers using `docker exec -it -u $(id -u):$(id -g) CONTAINER_NAME COMMAND` to force your current user ID into the process, but this will only work if your host OS is Linux, not mac. Follow a couple of simple rules and save yourself a world of hurt.

- Run composer outside of the php container, as doing so would install all your dependencies owned by `root` within your vendor folder.
- Run commands (ie Symfony's console, or Laravel's artisan) straight inside of your container. You can easily open a shell as described above and do your thing from there.

## Notes

Ensure the webserver config on `docker\nginx.conf` is correct for your project. phpdocker.io will have customised this file according to the application type you chose on the generator, for instance `web/app|app_dev.php` on a Symfony project, or `public/index.php` on generic apps.

Note: you may place the files elsewhere in your project. Make sure you modify the locations for the php-fpm dockerfile, the php.ini overrides and nginx config on `docker compose.yml` if you do so.
