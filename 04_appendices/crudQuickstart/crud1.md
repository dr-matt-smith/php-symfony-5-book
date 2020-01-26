# Preparation

## Ensure PHP is installed on your computer

## Ensure the `Composer` PHP command line tool is installed on your computer

Type `composer` in a command line terminal. If you get an error saying no such application, then install Composer from:

- [https://getcomposer.org/doc/00-intro.md#installation-windows](https://getcomposer.org/doc/00-intro.md#installation-windows) 

## Ensure the SQLite PHP database extension is enabled (reqeuired for Windows)

If Symfony is working, you have PHP setup on your computer. However, the SQLite database extension may not be setup.

You can either work ahead, hoping it is setup, and fix it if you hit a problem when trying to create a database. Or you can check, and fix it now.

PHP extensions are already installed with PHP, but may not be activated. All we have to do is ensure there is no semi-colon character `;` at the beginning of the line `extension=php_pdo_sqlite.dll'. 

Do the following:

1. Use Notepad++ to open file `C:\php\php.ini`

1. Search for the line ‘extension=pdo_sqlite'
    - use `<CTRL>-F` to search

1. If the line reads `;extension=php_pdo_sqlite.dll` remove the semi-colon `;` at the beginning of the line and save the changed file.

1. That's it - you have now enabled SQLite for PHP applications.

If you want to enable the MySQL database as well, then do the same for the line saying `extension=php_pdo_mysql.dll'.

![SQLite being enabled in php.ini in the Notepad++ editor.](./03_figures/app_crud/crud00_notepad_enable_pdo_sqlite.png){ width=60% }

## Install the PHPStorm code editor

1. Get your free one-year subscription to Jetbrains products using your **TUDublin** university email address

    - `https://www.jetbrains.com/shop/eform/students`

1. Download and install the PHPStorm editor

## Download project template and open in a code editor

1. Download to the Desktop and unzip folder 'crud1.zip' from Moodle

1. Start your IDE editor (e.g. Notepad++ or PHPStorm)

1. Open a terminal window (either a Terminal applicaiton like `cmd`, or open a Terminal window inside your IDE)
 
1. In the terminal `cd` into folder `crud1`


## Run the Symfony web sever

Let's run the web server on our machine (`localhost:8000`) by entering terminal command:

```bash
php bin\console server:run`.
```
Note, you might get some warnings/info messages about version of PHP etc. - just ignore them!

```bash
    > php bin\console server:run
    Sep 27 07:22:19 |DEBUG| PHP    Using PHP version 7.3.8 (from default version in $PATH) 
    Sep 27 07:22:19 |INFO | PHP    listening path="/usr/local/php5-7.3.8-20190811-205217/sbin/php-fpm" php="7.3.8" port=52271
    Sep 27 07:22:19 |DEBUG| PHP    started 
    Sep 27 07:22:19 |INFO | PHP    ready to handle connections 
                                                                                                                            
     [OK] Web server listening on http://127.0.0.1:8000 (PHP FPM 7.3.8)                                                     
```

## Visit the home page `localhost:8000`

Open a web browser and visit our website home page at `http://localhost:8000`.

Since we didn't create a home page, we'll see a default Symfony home page. See Figure \ref{homepage} shows a screenshot of PHPStorm and our new class PHP code.

![Default Symfony home page. \label{homepage}](./03_figures/appendices/crud04_homepage.png){ width=75% }