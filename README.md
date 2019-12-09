# Dockerized Yii2 with MariaDB and PHP 7.3

## Overview

Whether you're a beginner or an advanced Yii developer, dockerizing your app can be daunting with a ton of configurations when all you wanted is a quick way to get set up and running in your local development. This docker base is made to help solve those setup requirements but addresses the need to use custom templates and virtual hosts.

## Initial Setup

Clone this repository to use as your docker base for Yii2-enabled sites. Launch docker and create a network:

```
docker network create amp-yii
```

Update the environment variables inside `secret/dev.env` to suit your needs.

Start and run the containers:

```
docker-compose up -d --build
```

This docker base have a persistent storage for the Yii2 application and for Composer to successfully create it using the basic template, we need to empty it. Run the following commands inside the `web` container:

```
rm /app/.gitkeep
composer create-project yiisoft/yii2-app-basic /app
```

Or, run these from your terminal instead:

```
docker-compose run --rm web rm /app/.gitkeep
docker-compose run --rm web composer create-project yiisoft/yii2-app-basic /app
```

If you want to use the advanced template, change `yiisoft/yii2-app-basic` to `yiisoft/yii2-app-advanced`.

## Persisting Data

To keep files down to a minimum and to prepare for a combined CI/CD (continuous integration and continuous delivery) practice, containers are removed when you stop the dockerized app. Simply meant, all your data and configurations will be lost and the next time you run the images, the database will be reinitialized.

To avoid this loss of data, volumes are mounted that will persist even after the containers were removed.

```
docker-yii2
|- data
|- yii
\-docker-compose.yml
```

## Local Access

Access the site through http://localhost/ in your browser, no need to add the port as it's automatically configured to use port 80.

To access the database, use an SQL client and enter the following info:

```
Hostname: 127.0.0.1
Port: 8890
Username: root
Password: root123!
```

If you changed the default values in the `secret/dev.env` file, use those values instead.

Do not use the default SQL port (3306) - it will not work when accessing the database container. Use the assigned port in the `docker-compose.yml` file. The port number can be changed as long as you know what you are doing but for the purpose of this docker base, let's keep it as it is.

For security reasons, it is advisable to use SSH tunneling to access the database server for production use.

## Configurations

### Pretty URLs

By default, Apache `mod_rewrite` module is enabled but pretty URLs in the app are not. Open `config/web.php` and find these lines:

```php
/*
'urlManager' => [
    'enablePrettyUrl' => true,
    'showScriptName' => false,
    'rules' => [
    ],
],
*/
```

Uncomment them and add some URL rules, as such:

```php
'urlManager' => [
    'enablePrettyUrl' => true,
    'showScriptName' => false,
    'rules' => [
        '/'         => 'site/index',
        '/about'    => 'site/about',
        '/contact'  => 'site/contact',
        '/login'    => 'site/login',
        '/logout'   => 'site/logout'
    ],
],
```

Refresh the page and you should now see the updated URL routes.

### Virtual Host

By default, the Apache virtual host configuration is set to `/app/web` as the document root. If you opted to create an app using the advanced template (or any custom skeleton template), you need to update the virtual host entries.

Update `data/conf/000-default.conf` before you run your dockerized app. If you made any virtual host changes while docker is running, don't forget to restart the Apache server from inside the `web` container.

### Custom Template

If you want to use a custom template instead of the ones provided officially by the Yii Framework, you can do it during project creation. For example:

```
docker-compose run --rm web composer create-project --prefer-dist justcoded/yii2-starter /app
```

And as explained in the previous subsection, set up the virtual host configuration for your custom template to work.

### Database Connection

As the database server is containerized, using `localhost` in your app's DB config will not work. You will need to use the database container's name and port instead, as follows:

```php
return [
    'class' => 'yii\db\Connection',
    'dsn' => 'mysql:host=db:3306;dbname=app',
    'username' => 'devs',
    'password' => 'devs123!',
    'charset' => 'utf8',
];
```

If you are using an `.env` file, use it like this:

```
DB_HOST=db:3306
DB_NAME=app
DB_USER=devs
DB_PASS=devs123!
```