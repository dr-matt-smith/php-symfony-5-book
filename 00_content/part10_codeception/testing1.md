
# Unit testing in Symfony with Codeception


## Codeception Open Source BDD project

Codeception is a BDD (Behaviour Driver Design) open source project, to support acceptance (end-to-end, and unit testing for PHP project. The community supports its close integration with Symfony:

- Github
    
    - [https://github.com/codeception/codeception](https://github.com/codeception/codeception)

- Project home page

    - [https://codeception.com/](https://codeception.com/)
    
- Codeception Symfomy docs

    - [https://codeception.com/for/symfony](https://codeception.com/for/symfony)

## Adding Codeception to an existing project (project `codeception01`)

For these examples we'll start with an existing project;

- security09

    - [https://github.com/dr-matt-smith/php-symfony4-book-codes-security-09-twig-security](https://github.com/dr-matt-smith/php-symfony4-book-codes-security-09-twig-security)

    - this project has a public home page, student (ROLE_USER) page, admin (ROLE_ADMIN) pages, user roles, login pages, CRUD pages

So do the following:

1. clone this existing Symfony project

    - [https://github.com/dr-matt-smith/php-symfony4-book-codes-security-09-twig-security](https://github.com/dr-matt-smith/php-symfony4-book-codes-security-09-twig-security)

1. since we are now in a **TESTING** mode, immediately change the `.env` file adding the suffix `test` to the database name - we don't test on a project's **production** database !

    ```
        DB_USER=root
        DB_PASSWORD=passpass
        DB_HOST=127.0.0.1
        DB_PORT=3306
        DB_NAME=security5test
        DATABASE_URL=mysql://${DB_USER}:${DB_PASSWORD}@${DB_HOST}:${DB_PORT}/${DB_NAME}
    ```
    
    - then create the new database with `doctrine:database:create`

1. Add Codeception via composer

    ```bash
        $ composer req codeception/codeception --dev
    
        Using version ^2.5 for codeception/codeception
        ./composer.json has been updated
        Loading composer repositories with package information
        Updating dependencies (including require-dev)
        Package operations: 118 installs, 0 updates, 0 removals
        
          - Installing .... packages ...
          
        Writing lock file
        Generating autoload files
        Symfony operations: 2 recipes (d7cfe8af216b9f6fdaf106b4fa7854e0)
          - Configuring phpunit/phpunit (>=4.7): From github.com/symfony/recipes:master          
    ```
    
    - you'll get a **WARNING** that you are not installing an 'official' Symfony package, but one contributed by the open source community:
    
        ```bash
          -  WARNING  codeception/codeception (>=2.3): From github.com/symfony/recipes-contrib:master
            The recipe for this package comes from the "contrib" repository, which is open to community contributions.
            Review the recipe at https://github.com/symfony/recipes-contrib/tree/master/codeception/codeception/2.3
        
            Do you want to execute this recipe?
            [y] Yes
            [n] No
            [a] Yes for all packages, only for the current installation session
            [p] Yes permanently, never ask again for this project
            (defaults to n): 
        ```

1. answer 'y' (YES!):

    ```bash
          - Configuring codeception/codeception (>=2.3): From github.com/symfony/recipes-contrib:master
        ocramius/package-versions:  Generating version class...
        ocramius/package-versions: ...done generating version class
        Executing script cache:clear [OK]
        Executing script assets:install public [OK]
        
        Some files may have been created or updated to configure your new packages.
        Please review, edit and commit them: these files are yours.
                                                                                                     
        Adding phpunit/phpunit as a dependency is discouraged in favor of Symfony's PHPUnit Bridge. 
                                                                                                     
          * Instead:
            1. Remove it now: composer remove --dev phpunit/phpunit
            2. Use Symfony's bridge: composer require --dev phpunit
    ```
    
    - we see the 'bootstrapping' process for Codeception, so you don't need to bootstrap it yourself (which may be required if using Codeception for a non-Symfony project):
    
        ```bash
             Bootstrapping Codeception 
            File codeception.yaml created       <- global configuration
            tests/unit created                  <- unit tests
            tests/unit.suite.yaml written       <- unit tests suite configuration
            tests/functional created            <- functional tests
            tests/functional.suite.yaml written <- functional tests suite configuration
            tests/acceptance created            <- acceptance tests
            tests/acceptance.suite.yaml written <- acceptance tests suite configuration
            Codeception is installed for acceptance, functional, and unit testing
            Next steps:
            1. Edit tests/acceptance.suite.yaml to set url of your application. Change PhpBrowser to WebDriver to enable browser testing
            2. Edit tests/functional.suite.yaml to enable a Doctrine module if needs.
            3. Create your first acceptance tests using vendor/bin/codecept g:cest acceptance First
            4. Write first test in tests/acceptance/FirstCest.php
            5. Run tests using: vendor/bin/codecept run        
        ```
        

1. Notice the message from Symfony about `phpunit/phpunit` bneing **discouraged**, saying we should use the Symfony PHPUnit **Bridge**:

    ```bash
         Adding phpunit/phpunit as a dependency is discouraged in favor of Symfony's PHPUnit Bridge. 
        
          * Instead:
            1. Remove it now: composer remove --dev phpunit/phpunit
            2. Use Symfony's bridge: composer require --dev phpunit    
    ```
    
    - let's do what is recommended, so first run the `composer remove ...` command:
    
        ```bash
            $ composer remove --dev phpunit/phpunit
            
            phpunit/phpunit is not required in your composer.json and has not been removed
            Loading composer repositories with package information
            Updating dependencies (including require-dev)
            Restricting packages listed in "symfony/symfony" to "4.2.*"
            Nothing to install or update
            Generating autoload files
            ocramius/package-versions:  Generating version class...
            ocramius/package-versions: ...done generating version class
            Executing script cache:clear [OK]
            Executing script assets:install public [OK]
        ```
        
    - then run the `composer require ...` command to add the Symfony bridge:
    
        ```bash
            $ composer require --dev phpunit  
            
            Using version ^1.0 for symfony/test-pack
            ./composer.json has been updated
            Loading composer repositories with package information
            Updating dependencies (including require-dev)
            Restricting packages listed in "symfony/symfony" to "4.2.*"
            Package operations: 3 installs, 0 updates, 0 removals
              - Installing symfony/phpunit-bridge (v4.2.4): Loading from cache
              - Installing symfony/panther (v0.3.0): Loading from cache
              - Installing symfony/test-pack (v1.0.5): Loading from cache
            ...
        ```

## What Codeception has added to our project

You should now have added to your project the following:

- file `codeception.yaml` in main project folder file:

    ```yaml
          namespace: App\Tests
          paths:
              tests: tests
              output: tests/_output
              data: tests/_data
              support: tests/_support
              envs: tests/_envs
          actor_suffix: Tester
          extensions:
              enabled:
                  - Codeception\Extension\RunFailed
          params:
              - .env
    ```

    - NOTE: the default URL for testing is: `localhost:8000` - you may need to change this if using a web server other than the Symfony one ...
	
- new folder `/tests` - you'll create your tests in here

- new folder `vendor/bin/` containing executable: `codecept`

    - you run command line Codeception commands with:

        ```bash
            vendor/bin/codecept <command>
        ```
