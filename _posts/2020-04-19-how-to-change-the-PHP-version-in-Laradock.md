---
title: How to change the PHP version in Laradock?
---

I've recently switched to Laradock as my primary local development setup for Laravel. It's a full PHP development environment based on Docker. It took me a few days to properly set it up and learn the basics of Docker itself. Laradock offers a flexible way to initialize all kind of services, like

I previously used Laravel Homestead, an official, a pre-packaged Vagrant box that provides something similar. To run this box I had to install VirtualBox. This setup works totally fine and is very reliable, but I wanted to switch to a more general solution. I do still use it on my Laptop.

Changing the PHP version was one of the customizations I had to make. Laradock made this really easy.

**Step 1**

Find the `.env` file in your Laradock directory.

**Step 2**

Search for the `PHP_VERSION` variable and update the version to the preferred one.

**Step 3**

Rebuild the `php-fpm` and `workspace`  by running:

```bash
docker-compose build php-fpm
docker-compose build workspace
```

**Step 4**

Before these changes are taken into account you will have to recreate the Laradock container.

```bash
docker-compose down
docker-compose up -d nginx mysql phpmyadmin redis workspace
```

**Step 5**

If you want to determine if these actions have changed the PHP version, you will have to do the following things:

1 - View a PHP file with `echo phpinfo()` through your webserver and check the version.

2 - Run `php -v` on your server by opening bash via `docker-compose exec workspace bash`

## Official documentation

* [Change the (PHP-FPM) Version](https://laradock.io/#change-the-php-fpm-version)
* [Change the PHP-CLI Version](https://laradock.io/#change-the-php-cli-version)
