
# Twig templating

## Add the debug-profiler bundle (`basic02`)


When developing we want all the error/warning/debugging information we can get. Let's add the Symfony profiler, which tells us lots about how thing are, and are not working with out site in development mode.


```bash
    $ composer req debug
    
    Using version ^1.0 for symfony/debug-pack
    ./composer.json has been updated
    Loading composer repositories with package information
    Updating dependencies (including require-dev)

    ...
```

<!--
![Screenshot of 404 error for URL path `/`. \label{404_error}](./03_figures/lab02/1_404_error.png)
-->

NOTE: The debug bundle makes use of (requires) the Twig templating bundle. This will impact:

- the look of error pages

- the code generated for new controllers

Try this (now we have Twig added):

1. Delete the controller class file `/src/Controller/DefaultController.php`^[That's one great thing about working with generated code - we can delete it and regenerate it with little or no work.]

2. Generate a new Default controller class with `php bin/console make:controller Default`

3. Look at the generated code:

    ```php
        /**
         * @Route("/default", name="default")
         */
        public function index()
        {
            // replace this line with your own code!
            return $this->render('@Maker/demoPage.html.twig', [
                'path' => str_replace($this->getParameter('kernel.project_dir').'/', '', __FILE__)
            ]);
        }
    ```

As you can see, the controller method now returns the output of method `$this->render(...)` rather than directly creating a `Response` object. With the Twig bundle added, each controller class now has access to the Twig `render(...)` method.

Figure \ref{generated_default_twig} shows a screenshot of the message created from our generated default controller method with Twig.

NOTE: The actual look of the default generated Twig content may be a little different (e.g. 19 Feb 2019 it now says `Hello DefaultController!`)...

![Screenshot of generated page for URL path `/default`. \label{generated_default_twig}](./03_figures/lab02/2_generated_page.png)

## View the routes added by the profiler

View the route list now - since our profile has added some (with the underscore `_` prefix):

```bash
      Name                       Method   Scheme   Host   Path
     -------------------------- -------- -------- ------ -----------------------------------
      default                    ANY      ANY      ANY    /default
      _twig_error_test           ANY      ANY      ANY    /_error/{code}.{_format}
      _wdt                       ANY      ANY      ANY    /_wdt/{token}
      _profiler_home             ANY      ANY      ANY    /_profiler/
      _profiler_search           ANY      ANY      ANY    /_profiler/search
      _profiler_search_bar       ANY      ANY      ANY    /_profiler/search_bar
      _profiler_phpinfo          ANY      ANY      ANY    /_profiler/phpinfo
      _profiler_search_results   ANY      ANY      ANY    /_profiler/{token}/search/results
      _profiler_open_file        ANY      ANY      ANY    /_profiler/open
      _profiler                  ANY      ANY      ANY    /_profiler/{token}
      _profiler_router           ANY      ANY      ANY    /_profiler/{token}/router
      _profiler_exception        ANY      ANY      ANY    /_profiler/{token}/exception
      _profiler_exception_css    ANY      ANY      ANY    /_profiler/{token}/exception.css
```

## Specific URL path and internal name for our default route method

Let's change the URL path to the website root (`/`) and name the route `homepage` by editing the annotation comments preceding method `index()` in `src/Controllers/DefaultController.php`.


```php
    class DefaultController extends Controller
    {
        /**
         * @Route("/", name="homepage")
         */
        public function index()
```

Now the route is:
```bash
  Name                       Method   Scheme   Host   Path
 -------------------------- -------- -------- ------ -----------------------------------
  homepage                   ANY      ANY      ANY    /
```

Finally, let's replace that default message with an HTTP response that **we** have created - how about the message `hello there!`. We can generate an HTTP response by creating an instance of the `Symfony\Component\HttpFoundation\Response` class.

Luckily, if we are using a PHP-friendly editor like PHPStorm, as we start to type the name of a class, the IDE will popup a suggestion of namespaced classes to choose from.
Figure \ref{phpstorm_response} shows a screenshot of PHPStorm offering up a list of suggested classes after we have typed the letters `new Re`. If we accept a suggested class from PHPStorm, then an appropriate `use` statement will be inserted before the class declaration for us.

![Screenshot of PHPStorm IDE suggesting namespaces classes. \label{phpstorm_response}](./03_figures/lab02/4_phpstorm_class_suggester.png)


Here is a complete `DefaultController` class:
```php
    namespace App\Controller;

    use Symfony\Component\Routing\Annotation\Route;
    use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
    use Symfony\Component\HttpFoundation\Response;

    class DefaultController extends AbstractController
    {
        /**
         * @Route("/", name="homepage")
         */
        public function indexAction()
        {
            return new Response('Hello there!');
        }
    }
```


Figure \ref{hello_there} shows a screenshot of the message created from our `Response()` object.

![Screenshot of page seen for `new Response('hello there!')`. \label{hello_there}](./03_figures/lab02/3_hello_there.png)


## Adding Twig directly

If we hadn't added the profiler, we could have added just the Twig bundle as follows:

- Add Twig :

    ```bash
        composer req twig
    ```

## Secruty checker bundle

Another general bundle to always include is the Symfony security checkr:

1. Add the Symfony security checker - a good one to **always** have

    ```bash
        composer req sec-checker
    ```


## Development Symfony 4 recipes

Libaries installed with `--dev` are only for use in our development setup - that software isn't used (or installed) for public deloployment of our `production` website that will actually run live on the internet.

Here is the list of the most common development libraries we'll need:

1. the Server recipe

    ```bash
        composer req --dev server
    ```

1. the Maker recipe (for development setup):

    ```bash
        composer req --dev make
    ```

1. Add the Symfony PHPUnit bridge(for development setup):

    ```bash
        composer req --dev phpunit
    ```

1. Add the Symfony web profiler (with great `dump()` functions!)

    ```bash
        composer req --dev profiler
    ```

1. Add the Symfony debugging libraries

    ```bash
        composer req --dev debug
    ```

## Install multple libraries in a single `composer` commad

We can install all our non-development libraries with one command:

```bash
    composer req twig annotations sec-checker
```

and all our development libraries with another command (with the `--dev` option):

```bash
    composer req --dev server make phpunit debug profiler
```

## Let's create a nice Twig hom page  

We are (soon) going to create Twig template in `templates/default/homepage.html.twig`. So we need to ask the `Twig` object in our Symfony project to create an HTTP response via its `render()` method. Part of the 'magic' of PHP Object-Orienteted inheritance (and the **Dependancy Injection** design pattern), is that since our controller class is a subclass of `Symfony\Bundle\FrameworkBundle\Controller\Controller`, then objects of our controller automatically have access to a `render(...)` method for an automatically generated Twig object.

In a nutshell, to output an HTTP response generated from Twig, we just have to specify the Twig template name, and relative location^[The 'root' of Twig template locations is, by default, `/templates`. To keep files well-organised, we should create subdirectories for related pages. For example, if there is a Twig template `/templates/admin/loginForm.html.twig`, then we would need to refer to its location (relative to `/templates`) as `admin/loginForm.html.twig`. ], and supply an array of any parameters we want to pass to the template.

So we can simply write the following to ask Symfony to generate an HTTP response from Twig's text output from rendering the template that can (will soon!) be found in `/tempaltes/default/homepage.html.twig`:

```php
        /**
         * @Route("/", name="homepage")
         */
        public function indexAction()
        {
            $template = 'default/homepage.html.twig';
            $args = [];
            return $this->render($template, $args);
        }
```

Now let's create that Twig template in `/templates/default/homepage.html.twig`:

1. Create new directory  `/templates/default`

2. Create new file `/templates/default/homepage.html.twig`:

```twig
    {% extends 'base.html.twig' %}

    {% block body %}
        <h1>home page</h1>

        <p>
            welcome to the home page
        </p>
    {% endblock %}
```

Note that Twig paths searches from the Twig root location of `/templates`, not from the location of the file doing the inheriting, so do **NOT** write `{% extends 'default/base.html.twig' %}`...

Figure \ref{twig_page} shows a screenshot our Twig-generated page in the web browser.

![Screenshot of page from our Twig template. \label{twig_page}](./03_figures/lab02/5_twig_page.png)



