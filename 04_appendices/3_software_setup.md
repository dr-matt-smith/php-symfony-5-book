

# Get/Update your software tools\label{appendix_software_setup}

NOTE: All the following are already available on the ITB college computers. All you may need to do is:

1. ensure that Composer is up to date by running:

    ```bash
        composer self-update
    ```

2. enable the PDO options for MySQL and SQLite (see Appendix \ref{appendix_php} for how to do this by editing ther `c:\php\php.ini` file ...)


## Composer

The Composer tool is actually a **PHAR** (PHP Archive) - i.e. a PHP application packaged into a single file. So ensure you have PHP installed and in your environment **path** before attempting to install or use Composer.

Ensure you have (or install) an up-to-date version of the Composer PHP package manager.

```bash
    composer self-update
```

### Windows Composer install

Get the latest version of Composer from

- [getcomposer.org](https://getcomposer.org/)

- run the provided **Composer-Setup.exe** installer (just accept all the default options - do NOT tick the developer mode)

    -- [https://getcomposer.org/doc/00-intro.md#installation-windows](https://getcomposer.org/doc/00-intro.md#installation-windows)

## PHPStorm editor

Ensure you have your free education Jetbrains licence from:

- [Students form: https://www.jetbrains.com/shop/eform/students](https://www.jetbrains.com/shop/eform/students) (ensure you use your ITB student email address)

Downdload and install PHPStorm from:

- [https://www.jetbrains.com/phpstorm/download/](https://www.jetbrains.com/phpstorm/download/)

To save lots of typing, try to install the following useful PHPStorm plugins:

- Twig
- Symfony
- Annotations

## MySQL Workbench

While you can work with SQLite and other database management systems, many ITB modules use MySQLWorkbench for database work, and it's fine, so that's what we'll use (and, of course, it is already installed on the ITB windows computers ...)

Download and install MySQL Workbench from:

- [https://dev.mysql.com/downloads/workbench/](https://dev.mysql.com/downloads/workbench/)

## Git

Git is a fantastic (and free!) DVCS - Distributed Version Control System. It has free installers for Windows, Mac, Linus etc.

Check is Git in installed on your computer by typing `git` at the CLI terminal:

```bash
    > git
    usage: git [--version] [--help] [-C <path>] [-c name=value]
               [--exec-path[=<path>]] [--html-path] [--man-path] [--info-path]
               [-p | --paginate | --no-pager] [--no-replace-objects] [--bare]
               [--git-dir=<path>] [--work-tree=<path>] [--namespace=<name>]
               <command> [<args>]

    These are common Git commands used in various situations:

    start a working area (see also: git help tutorial)
       clone      Clone a repository into a new directory
       init       Create an empty Git repository or reinitialize an existing one

    ...

    collaborate (see also: git help workflows)
       fetch      Download objects and refs from another repository
       pull       Fetch from and integrate with another repository or a local branch
       push       Update remote refs along with associated objects

    'git help -a' and 'git help -g' list available subcommands and some
    concept guides. See 'git help <command>' or 'git help <concept>'
    to read about a specific subcommand or concept.

    >
```

If you don't see a list of **Git** commands like the above, then you need to install Git on your computer.

## Git Windows installation

Visit this page to run the Windows Git installer.

- [https://git-scm.com/downloads](https://git-scm.com/downloads)

NOTE: Do **not** use a GUI-git client. Do all your Git work at the command line. It's the best way to learn, and it means you can work with Git on other computers, for remote terminal sessions (e.g. to work on remote web servers) and so on.

