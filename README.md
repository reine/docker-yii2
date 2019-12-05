# Dockerized Yii2 with MariaDB and PHP 7.3

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

This docker base have a persistent storage for the Yii2 application and for Composer to successfully create it using the basic template, we need empty it. Run the following commands inside the `web` container:

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

Access the site through http://localhost/ in your browser and you should see the WordPress setup page on initial run. Proceed with the installation procedure until you have successfully logged inside the WordPress admin panel.

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

*** Currently work-in-progress. ***