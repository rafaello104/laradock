# Laradock guide
Welcome to the Laradock guide,<br>
Here you will learn how to deploy laravel in a docker container.<br/>
Everything is nicely explained, some of the line-numbers may have changed,  since we keep pulling the latest version
of laradock into our new projects, if you experience errors and can't fix them,<br/>don't forget to check for the latest versions 
of laradock on it's github repository and look if there are any posted issues there,<br/> if the current build is failing, than take 
the last passing build.


## Table of Content
1. Create the folder structure
1. Setting up Laradock
1. Spinning up Laradock
1. Setting up Laravel
1. Important docker commando`s



## 1. Creating the folder structure
Create folder on your harddrive to hold the laradock and laravel projects. <br>Use the following commands in a terminal:
````
mkdir Your-folder-name
````

Enter this folder and initialise a local Git repository.
````
cd Your-folder-name
git init
````

***(optional)*** Link the project to your remote repository
````
// Dont forget to change username and repository 
git remote add origin git@github.com:username/repository.git
````



Now add Laradock and Laravel as a submodule to your repository like so
````
git submodule add https://github.com/Laradock/laradock.git
git submodule add https://github.com/laravel/laravel.git
````



Your folder structure should now look like this:
````
+ laradock
+ laravel
   .gitignore
   .gitmodules
````


Our folder structure should be all set for now. 
<br>We have our basic components to spin up our laravel project with Docker. 
If you want to you could add/commit/push this to Github repository 
to save your current status.<br>



## 2. Setting up Laradock
Its now time to set up nginx, mariadb and phpmyadmin. 
<br>This way our workspace should be all set to start our development.



### Create a .env file
Enter the laradock folder and copy ``env-example`` to ``.env``



````
cd laradock

// for Linux
cp env-example 
.env

// for Windows
copy env-example 
.env
````



We need to edit the ``.env`` file to choose which software’s we want to be 
installed in your environment. 
We can always refer to the ``docker-compose.yml`` file to see how those variables are being used. 
All we need to know for now are these following steps:



### Change the settings inside the .env file
1. Open the .env file with any installed editor
1. Change line 8 to the following: ``APP_CODE_PATH_HOST=../laravel/``
1. Change on line 36 the variable ``COMPOSE_PROJECT_NAME`` to something unique like your project name.
1. Starting from line 250, Change the settings of the Mariadb Server. Database, Username, Password and root password.
1. Change line 314 to the following: ``PMA_DB_ENGINE=mariadb`` and change the credentials below.
1. Save the changes into this file and close it.</li></ol>



### Change the settings inside the docker-compose.yml file (Optional)
At the time of writing this column, laradock has a known bug into its settings..
<br>If we now try to load up our mariadb container, it will give us a error.
<br>In order to solve this we need to open the ``docker-compose.yml`` file and change the settings on line 370:

```` 
- mariadb:/var/lib/mysql
````



I am well aware that i remove the dynamic part of the mariadb container and 
by doing this we are making a recursive container.. 
But it is a fix we are grown custom to and laradock hasn't removed the bug for a while now.
<br>Hopefully, by the time you read this, this bug is resolved.



## 3. Spinning up Laradock for the first time
Now that all our settings are in order its time to start our engine`s and see what this baby is made off..
<br><br>In order to start our containers, enter the following commands:

````
docker-compose up -d nginx mariadb phpmyadmin
````



This might take a while, depending on whether you have the images installed locally.
<br>If you do need to wait, I advice you to get yourself a coffee.. this might take a while depending on your internet speed


After the installation is done, make sure all the containers are up and running by the following command:
````
docker ps -a
````


This should give you something like this:
````
CONTAINER ID        IMAGE                        COMMAND                  CREATED             STATUS              PORTS                                            NAMES
fe3b8b8a83d2        michelpietersbe_phpmyadmin   "/run.sh supervisord…"   4 minutes ago       Up 3 minutes        9000/tcp, 0.0.0.0:8080-&gt;80/tcp                michelpietersbe_phpmyadmin_1
8b10a517699a        michelpietersbe_nginx        "/bin/bash /opt/star…"   4 minutes ago       Up 4 minutes        0.0.0.0:80-&gt;80/tcp, 0.0.0.0:443-&gt;443/tcp   michelpietersbe_nginx_1
8399cb008b36        michelpietersbe_php-fpm      "docker-php-entrypoi…"   4 minutes ago       Up 4 minutes        9000/tcp                                         michelpietersbe_php-fpm_1
f0712df65f3a        michelpietersbe_workspace    "/sbin/my_init"          4 minutes ago       Up 4 minutes        0.0.0.0:2222-&gt;22/tcp                          michelpietersbe_workspace_1
035549ab5ec3        michelpietersbe_mariadb      "docker-entrypoint.s…"   4 minutes ago       Up 4 minutes        0.0.0.0:3306-&gt;3306/tcp                        michelpietersbe_mariadb_1
8c2cd6bdf46c        docker:dind                  "dockerd-entrypoint.…"   4 minutes ago       Up 4 minutes        2375/tcp                                         michelpietersbe_docker-in-docker_1
````



In total we should have 6 containers up and running.
<br>If we open up our browser and visit the url: ``localhost``
<br>we dont see anything going on, and we get an error:  
<br>**HTTP ERROR 500!**



In order to fix this we have to install Laravel to our docker container. 
Luckily our Laradock has all the tools already installed for us.



## 4. Setting up Laravel
Before we can install laravel we need to enter our docker container. 
In order to do so we need to execute the following command into our teminal:

````
docker exec -it
'YOUR_COMPOSE_PROJECT_NAME'_workspace_1 bash
````



We can install Laravel using composer:
````
composer install
````



Next we need to copy the ``.env-example`` to a ``.env`` file:
````
// for Linux
cp .env.example .env

// for Windows
copy .env.example .env
````



Open the ``.env`` file in a editor and change the following:



- on line 1: ``APP_NAME``
- on line 10: ``DB_HOST=mariadb``
- on line 12: ``DB_DATABASE=default``
- on line 13: ``DB_USERNAME=YOUR_SQL_USERNAME``
- on line 14: ``DB_PASSWORD=YOUR_SQL_PASSWORD``



Save the changes and close the file.
<br>Generate a laravel key and prepaire the DB:



````
php artisan key:generate 
php artisan migrate
````

All set! Visit with your browser the website by the following url: [http://localhost](http://localhost)



## 5. Important docker commando`s
In order to keep control over your containers you'll need to know a list of important commands. 
So study them closely. These commands will only work if your located inside the laradock folder.


````
// Starting laradock containers
docker-compose up -d nginx mariadb phpmyadmin

// Checking if containers are running
docker ps -a

 
 // Enter your workspace
 docker exec -it 'YOUR_COMPOSE_PROJECT_NAME'_workspace_1 bash 
 
 // Leave your workspace
 exit 
 
 // Stopping docker containers
 docker-compose down
 ````

Yaay! You finished setting up laravel - laradock container, this is entry level docker usage, 
if you want to know more about docker itself, go look up the official documentation on the website.


