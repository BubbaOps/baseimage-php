# A Minimal Ubuntu Base Image for PHP Development

What happens when [phusion/baseimage-docker](https://github.com/phusion/baseimage-docker) and [docker-library/php](https://github.com/docker-library/php) have a baby? This!

## baseimage
Baseimage-docker is a special [Docker](https://www.docker.com) image that is configured for correct use within Docker containers. It is Ubuntu, plus:

 * Modifications for Docker-friendliness.
 * Administration tools that are especially useful in the context of Docker.
 * Mechanisms for easily running multiple processes, [without violating the Docker philosophy](#docker_single_process).

 We are using it as the base for these images because it is so small while still providing all the tools we will need for true development environment. If you are wondering why this rather than Busybox or Alpine, take a look at their [README.md](https://github.com/phusion/baseimage-docker/blob/master/README.md) where they explain all the details.

## Official PHP Images
The [Docker "Official Image"](https://github.com/docker-library/official-images#what-are-official-images) for [`php`](https://hub.docker.com/_/php/) (not to be confused with any official `php` image provided by `php` upstream) ought to have a very solid idea of how one goes about compiling and configuring PHP in a container, right? Therefore, we are modeling our build after theirs.

## What is inside?
Well, everything. Seriously, we are building nearly everything we can possibly compile in here. The goal is to have a base image that we can then create development hosts from. You read that correctly. Our ultimate goal is to provide a container that we can run [Visual Studio Code](https://code.visualstudio.com/) and do all the development type stuff from writting the code to stepping through it while debugging. 

In order to facilitate this, we built most all of the extensions that are packaged with PHP, unlike the Official PHP builds. To be clear, we built them as shared libraries and they are not all enabled by default. So do not be overly worried that the mssql extension is going to pop up in the development environment when you do not even have an SQL Server database involved.

It also means that if we latter decide to upgrade one of the extensions, we can simply pickle/pecl it and replace the shared library.

To that end, we have also retained all of the source and dev files used _(another departure from the official php image)_ in order to have everything we need if we decide to build something in a specific dev environment later.

### Shared Extension List
|            |            |           |          |
|------------|------------|-----------|----------|
| bcmath     | bz2        | calendar  | curl     |
| exif       | gd         | gettext   | intl     |
| mbstring   | mysqli     | mysqlnd   | opcache  |
| openssl    | pcntl      | pdo_mysql | pdo_psql |
| pdo_sqlite | pdo_sqlsrv | pgsql     | readline |
| redis      | soap       | sockets   | sodium   |
| sqlite3    | sqlsrv     | unixODBC  | xdebug   |
| xsl        | yaml       | zip       | zlib     |


### Pre-Installed Tools
There are some common tools that PHP Developers keep around, we have made it a point to pre-install them and ensure they are on the path.

* [Composer 2.0](https://getcomposer.org/)
* [Pickle](https://github.com/FriendsOfPHP/pickle)
* [Pecl](https://pecl.php.net/)_*_

_* Deprecated in PHP 7.4_
## Versions
All images are currently based on Ubuntu 18.04 on which we provide standard php builds for PHP 7.2, 7.3, 7.4, and 8.0.



