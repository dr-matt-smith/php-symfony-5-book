
# Introduction

## What is Symfony 4?

It's a PHP 'framework' that does loads for you, if you're writing a secure, database-drive web application.

## What to I need on my computer to get started?

I recommend you install the following:

- PHP 7 (download/install from [php.net](https://php.net/))
- a MySQL database server - e.g. MySQLWorkbench Community is free and cross-platform
- a good text editor (I like [PHPStorm](https://www.jetbrains.com/phpstorm/specials/phpstorm/phpstorm.html?&gclid=CJTK_8SDrtICFWq-7Qodh98NpQ&gclsrc=aw.ds.ds&dclid=CNPY28WDrtICFQGn7QodqekBWg), but then it's free for educational users...)
- [Composer](https://getcomposer.org/) (PHP package manager - a PHP program)

or ... you could use something like [Cloud9](https://c9.io/dr_matt_smith), web-based IDE. You can get started on the free version and work from there ...

Learn more about the software needed for Symfony developmnent in Appendix \ref{appendix_software}. For steps in installing PHP and the other software, see Appendices \ref{appendix_php} and \ref{appendix_software_setup}.

## How to I get started with a new Symfony project

In a CLI (Command Line Interface) terminal window, `cd` into the directory where you want to create your Symfony project(s). Then create a new Symfony 4 empty project, named `project01` (or whatever you wish) by typing:

```bash
    $ composer create-project symfony/skeleton project01
```

You should see the following, if all is going well:


```bash
    Installing symfony/skeleton (v4.0.5)
      - Installing symfony/skeleton (v4.0.5): Loading from cache
    Created project in my-project
    Loading composer repositories with package information
    Updating dependencies (including require-dev)
    Package operations: 21 installs, 0 updates, 0 removals
      - Installing symfony/flex (v1.0.66): Downloading (100%)
      - Installing symfony/polyfill-mbstring (v1.6.0): Loading from cache
      ...

      * Run your application:
        1. Change to the project directory
        2. Execute the php -S 127.0.0.1:8000 -t public command;
        3. Browse to the http://localhost:8000/ URL.

           Quit the server with CTRL-C.
           Run composer require server for a better web server.
```

NOTE:
- If the first line does not show a Symfony version starting with `v4` then you may have an old version of PHP installed. You need PHP 7.1.3 minimum to run Symfony 4.


Another way to get going quickly with Symfomy is to download one of the projects accompanying this book ...

## Where are the projects accompanying this book?

All the projects in this book are freely available, as public repositories on Github as follows:

- [https://github.com/dr-matt-smith/php-symfony4-book-codes](https://github.com/dr-matt-smith/php-symfony4-book-codes)

To retrieve and setup a sample project follow these steps:

1. download the project to your local computer (e.g. `git clone URL`)

1. change (`cd`) into the created directory

1. type `composer update` to download any required 3rd-party packages into a `/vendor` folder

1. Then run your web server (see below) and explore via a web browser


## How to I run a Symfony webapp?

### From the CLI with PHP built-in web server
At the CLI (command line terminal) ensure you are at the base level of your project (i.e. the same directory that has your `composer.json` file), and type the following to run the PHP built-in web server:

```bash
    $ php -S localhost:8000 -t public
```

See Figure \ref{default_home_page} for a screenshot of the default Symfony 4 home page.

![Screenshot default Symfony 4 home page. \label{default_home_page}](./03_figures/chapter02/0_default_page_sm.png)

### From the CLI with Symfony's web server

However Symfony offers its own (better!) server, which is easily installed and run.

To install the Symfony server component just type the following at the CLI (having changed into the project directory):

```bash
    $ composer req --dev server
```

Check this vanilla, empty project is all fine by running the web sever and visit website root at `http://localhost:8000/`

To run the server:

```bash
    $ php bin/console server:run
     [OK] Server listening on http://127.0.0.1:8000
     // Quit the server with CONTROL-C.
```

Then open a web browser and visit the website root at `http://localhost:8000`.


### From a Webserver application (like Apache or XAMPP)

If you are running a webserver (or combined web and database server like XAMPP or Laragon), then point your web server root to the project's  `/public` folder - this is where public files go in Symfony projects.

## It isn't working! (Problem Solving)

If you have trouble with running Symfony, take a look at Appendix \ref{appendix_problem_solving}, which lists some common issues and how to solve them.

## Can I see a demo project with lots of Symfony features?

Yes! There is a full-featured Symfony demo project. Checkout Appendix \ref{appendix_sf_demo} for details of downloading and running the demo and its associated automated tests.

## Any free videos about SF4 to get me going?

Yes! Those nice people at Symfonycasts have released a bunch of free videos all about Symfony 4 (and OO PHP in general).

So plug in your headphones and watch them, or read the transcripts below the video if you're no headphones. A good rule is to watch a video or two **before** trying it out yourself.

You'll find the video tutorials at:

- [https://symfonycasts.com/tracks/symfony](https://symfonycasts.com/tracks/symfony)
