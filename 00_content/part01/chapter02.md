
# First steps

## What we'll make (`basic01`)

See Figure \ref{new_home_page} for a screenshot of the new homepage we'll create in our first project.

![New home page.\label{new_home_page}](./03_figures/lab02/5_twig_page.png)

There are 3 things Symfony needs to serve up a page:

1. a route
2. a controller class and method
3. a Response object to be returned to the web client

The first 2 can be combined, through the use of 'Annotation' comments, which declare the route in a comment immediately before the controller method defining the 'action' for that route. See this example:


```php
    /**
     * @Route("/", name="homepage")
     */
    public function indexAction()
    {
        ... build and return Response object here ...
    }
```

For example the code below defines:

- an annotation Route comment for URL pattern `/` (i.e. website route)

    -- ```@Route("/", name="homepage")```

    -- the Symfony "router" system attemptes to match pattern  `/` in the URL of the HTTP Request received by the server

- controller method `indexAction()`

    -- this method will be involved if the route matches

    -- controller method have the responsibility to create and return a Symfony `Response` object

- note, Symfony allows us to declare an internal name for each route (in the example above `homepage`)

    -- we can use the internal name when generating URLs for links in out templating system

    -- the advantage is that the route is only defined once (in the annotation comment), so if the route changes, it only needs to be changed in one place, and all references to the internal route name will automatically use the updated route

    -- for example, if this homepage route was changed from `/` to `/default` all URls generated using the `homepage` internal name would now generated `/default`


## Create a new Symfony project

1. Create new Symfony 4 project (and then `cd` into it):

    ```bash
        $ composer create-project symfony/skeleton basic01
        Installing symfony/skeleton (v4.0.5)
          - Installing symfony/skeleton (v4.0.5): Loading from cache

        ... etc. ...

        $ cd basic01
    ```


1. Add the Symfony local development server:

    ```bash
        composer req --dev server
    ```


NOTE: To **remove** a package use `composer rem <package>`, e.g. `composer rem server`.

Check this vanilla, empty project is all fine by running the web sever and visit website root at `http://localhost:8000/`:

```bash
$ php bin/console server:run
 [OK] Server listening on http://127.0.0.1:8000
 // Quit the server with CONTROL-C.
```

Figure \ref{default_page} shows a screenshot of the default page for the web root (path `/`), when we have no routes set up and we are in development mode (i.e. our `.env` file contains `APP_ENV=dev`).

![Screenshot default Symfony 4 page for web root (when no routes defined). \label{default_page}](./03_figures/lab02/0_default_page.png)

## List the routes

There should not be any routes yet - but let's check at the console:

```
    $ php bin/console debug:router
      Name   Method   Scheme   Host   Path
     ------ -------- -------- ------ ------
```

## Add the annotations bundle

Since we'll be defining routes using annotation comments, we need to ask Composer to download the annotations bundle into our `/vendor` directory (and register the bundle, and update the autoloader etc.):

1. Add Annotations :

    ```bash
        $ composer req annotations
        Using version ^5.1 for sensio/framework-extra-bundle
        ./composer.json has been updated
        Loading composer repositories with package information
        ...
        Some files may have been created or updated to configure your new packages.
        Please review, edit and commit them: these files are yours.
    ```

## Create a controller

We could write a new class for our homepage controller, but ... let's ask Symfony to make it for us. Typical pages seen by non-logged-in users like the home page, about page, contact details etc. are often referred to as 'default' pages, and so we'll name the controller class for these pages our `DefaultController`.

1. First we need to add the `make` bundle to our console tool (for our development environment):

    ```bash
        $ composer req --dev make
        Using version ^1.0 for symfony/maker-bundle
        ./composer.json has been updated
        Loading composer repositories with package information
        ...
    ```

2. Now let's ask Symfony to create a new homepage (default) controller:

```bash
    $ php bin/console make:controller Default
    created: src/Controller/DefaultController.php

    Success!
    Next: Open your new controller class and add some pages!
```

NOTE: Symfony controller classes are stored in directory `/src/Controller`.

Look inside the generated class `/src/Controller/DefaultController.php`. It should look something like this:

```php
    <?php
    
    namespace App\Controller;
    
    use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
    use Symfony\Component\Routing\Annotation\Route;
    
    class DefaultController extends AbstractController
    {
        /**
         * @Route("/default", name="default")
         */
        public function index()
        {
            return $this->json([
                'message' => 'Welcome to your new controller!',
                'path' => 'src/Controller/DefaultController.php',
            ]);
        }
    }
```

This default controller actually returns a JSON object: 

```json
    {
        "message":"Welcome to your new controller!",
        "path":"src\/Controller\/DefaultController.php",
    }
```

Let's change this to return a text response. Do the following:

1. Add a `use` statement for the `Symfony\Component\HttpFoundation\Response` class

2. Change the body of the `index()` method to output a simple text message response:

    ```php
        return new Response('Welcome to your new controller!');
    ```

So the listing of your `DefaultController` should look as follows:
```php
    namespace App\Controller;
    
    use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
    use Symfony\Component\Routing\Annotation\Route;
    use Symfony\Component\HttpFoundation\Response;
    
    class DefaultController extends AbstractController
    {
        /**
         * @Route("/default", name="default")
         */
        public function index()
        {
            return new Response('Welcome to your new controller!');
        }
    }
```



Learn more about the Maker bundle:

- [https://symfony.com/blog/introducing-the-symfony-maker-bundle](https://symfony.com/blog/introducing-the-symfony-maker-bundle)


## Run web server to visit new default route

Run the web sever and visit the home page at `http://localhost:8000/`.

But we see that default Symfony welcome page, not our custom response text message!

Since we **have** defined a route, we don't get the default page any more. However, since we named our controller `Default`, then this is the route that was defined for it:


```bash
  Name                       Method   Scheme   Host   Path
 -------------------------- -------- -------- ------ -----------------------------------
  default                    ANY      ANY      ANY    /default
```

If we look more closely at the generated code, we can see this route `/default` in the **annotation** comment preceding controller method `index()` in `src/Controllers/DefaultController.php`

```php
    @Route("/default", name="default")
```

So visit `http://localhost:8000/default` instead, to see the page generated by our `DefaultController->index()` method.


Figure \ref{generated_default} shows a screenshot of the message created from our generated default controller method.

![Screenshot of generated page for URL path `/default`. \label{generated_default}](./03_figures/part01/2_default_page_no_twig.png)

## Clearing the cache

Sometimes, when we've added a new route, we still get an error saying the route was not found. This can be a problem of the Symfony **cache**.

So clearing the cache is a good way to resolve this problem (you may get in the habit of clearing the cache each time you add/change any routes).

You can clear the cache in 2 ways:

1. Simply delete directory `/var/cache`

1. Use the CLI command to clear the cache:

    ```bash
        $ php bin/console cache:clear
        
        // Clearing the cache for the dev environment with debug true                                                          
        [OK] Cache for the "dev" environment (debug=true) was successfully cleared.   
        
        $
    ```
