Up and Running
Now let's build the environment, and get it up running. We'll also be installing composer dependencies as well as some artisan command.
$ docker-compose build && docker-compose up -d && docker-compose logs -f
Creating network "backend-network" with the default driver
Creating mysql-db    ... done
Creating laravel-app ... done
Attaching to laravel-app, mysql-db
...
Once all the containers are up and running, we can check them by docker ps:
CONTAINER ID        IMAGE                 COMMAND                  CREATED             STATUS              PORTS                  NAMES
c1ae3002d260        laravel_laravel-app   "docker-php-entrypoi…"   4 minutes ago       Up 4 minutes        0.0.0.0:8000->80/tcp   laravel-app
6f6546224051        mysql:5.7             "docker-entrypoint.s…"   4 minutes ago       Up 4 minutes        3306/tcp               mysql-db
Composer and artisan:
$ docker exec -it laravel-app bash -c "sudo -u devuser /bin/bash"
devuser@c1ae3002d260:/var/www/html$ composer install
...
Generating optimized autoload files
> Illuminate\Foundation\ComposerScripts::postAutoloadDump
> @php artisan package:discover --ansi
Discovered Package: beyondcode/laravel-dump-server
Discovered Package: fideloper/proxy
Discovered Package: laravel/tinker
Discovered Package: nesbot/carbon
Discovered Package: nunomaduro/collision
Package manifest generated successfully.
devuser@c1ae3002d260:/var/www/html$ php artisan key:generate
Application key set successfully.
devuser@c1ae3002d260:/var/www/html$ php artisan migrate
Migrating: 2014_10_12_000000_create_users_table
Migrated:  2014_10_12_000000_create_users_table
Migrating: 2014_10_12_100000_create_password_resets_table
Migrated:  2014_10_12_100000_create_password_resets_table
devuser@c1ae3002d260:/var/www/html$ php artisan make:auth
Authentication scaffolding generated successfully.
With hostfile:
127.0.0.1       laravel-app.local
We're all set!
App Screenshot
App Screenshot

Helper scripts (optional)
From time to time, I want to be able to quickly run CLI commands (composer, artisan, etc.) without having to type docker exec everytime. So here are some bash scripts I made wrapping around docker exec:

container
#!/bin/bash

docker exec -it laravel-app bash -c "sudo -u devuser /bin/bash"
Running ./container takes you inside the laravel-app container under user uid(1000) (same with host user)
$ ./container
devuser@8cf37a093502:/var/www/html$
db
#!/bin/bash

docker exec -it mysql-db bash -c "mysql -u dbuser -psecret db"
Running ./db will connect to your database container's daemon using mysql client.
$ ./db
mysql>
composer
#!/bin/bash

args="$@"
command="composer $args"
echo "$command"
docker exec -it laravel-app bash -c "sudo -u devuser /bin/bash -c \"$command\""
Run any composer command, example:
$ ./composer dump-autoload
Generating optimized autoload files> Illuminate\Foundation\ComposerScripts::postAutoloadDump
> @php artisan package:discover --ansi
Discovered Package: beyondcode/laravel-dump-server
Discovered Package: fideloper/proxy
Discovered Package: laravel/tinker
Discovered Package: nesbot/carbon
Discovered Package: nunomaduro/collision
Package manifest generated successfully.
Generated optimized autoload files containing 3527 classes
php-artisan
#!/bin/bash

args="$@"
command="php artisan $args"
echo "$command"
docker exec -it laravel-app bash -c "sudo -u devuser /bin/bash -c \"$command\""
Run php artisan commands, example:
$ ./php-artisan make:controller BlogPostController --resource
php artisan make:controller BlogPostController --resource
Controller created successfully.
phpunit
#!/bin/bash

args="$@"
command="vendor/bin/phpunit $args"
echo "$command"
docker exec -it laravel-app bash -c "sudo -u devuser /bin/bash -c \"$command\""
Run ./vendor/bin/phpunit to execute tests, example:
$ ./phpunit --group=failing
vendor/bin/phpunit --group=failing
PHPUnit 7.5.8 by Sebastian Bergmann and contributors.



Time: 34 ms, Memory: 6.00 MB

No tests executed!
TL;DR
Links:

Repository
commit
Dockerfile consists of basic apache document root config, mod_rewrite and mod_header, composer and sync container's uid with host uid.

docker-compose.yml boots up php-apache (mount app files) and mysql (mount db files), using networks to interconnect.

Use the environment:
$ docker-compose build && docker-compose up -d && docker-compose logs -f
$ ./composer install
$ ./php-artisan key:generate
