
# Twig and logging 

## Getting reference to Twig and Logger objects

There are many useful service objects available in the Symfony system via the 'Service Container'. This is a design pattern known as **Dependency Injection**. In Symfony we get access to a servce object by **Type Hinting** with the server or interface class name, in the parameter parentheses of the method or constructor of the class.

In this chapter we'll use this technique to get a reference to the Twig and Logger service objects.

Learn more in the Symfony documentation:

- [https://symfony.com/doc/current/service_container.html](https://symfony.com/doc/current/service_container.html)

- [https://symfony.com/doc/current/components/dependency_injection.html](https://symfony.com/doc/current/components/dependency_injection.html)


## Using Twig for access denied message (project `security07`)

Let's improved our Access Denied exception handler in 2 ways:

- display a nice Twig template

- log the exception using the standard Monolog logging system

First add Monolog to our project with Composer:

```bash
    $ composer req logger
```

Now we will refactor class `AccessDeniedHandler` to

```php
    namespace App\Security;

    use Psr\Log\LoggerInterface;
    use Symfony\Component\DependencyInjection\ContainerInterface;
    use Symfony\Component\HttpFoundation\Request;
    use Symfony\Component\HttpFoundation\Response;
    use Symfony\Component\Security\Core\Exception\AccessDeniedException;
    use Symfony\Component\Security\Http\Authorization\AccessDeniedHandlerInterface;

    class AccessDeniedHandler implements AccessDeniedHandlerInterface
    {
        private $twig;
        private $logger;

        public function __construct(ContainerInterface $container, LoggerInterface $logger)
        {
            $this->twig = $container->get('twig');
            $this->logger = $logger;
        }

    }
```

Now we can re-write method `handle(...)` to log an error message, and

```php
    public function handle(Request $request, AccessDeniedException $accessDeniedException)
    {
        $this->logger->error('access denied exception');

        $template = 'error/accessDenied.html.twig';
        $args = [];
        $html = $this->twig->render($template, $args);
        return new Response($html);
    }
```

## The Twig page

Create Twig error page `/templates/error/accessDenied.html.twig`:

```twig
    {% extends 'base.html.twig' %}
    
    {% block title %}error{% endblock %}
    
    {% block body %}
        sorry - access is denied for your request
        <p>
            <a href="{{ url('homepage') }}">home</a>
        </p>
    {% endblock %}
```

See Figure \ref{denied_log} to see the error log register in the Symfony profiler footer, at the bottom of our custom error page.

![Screenshot of Custom Twig access denied page. \label{denied_log}](./03_figures/part06_security/9_twig_page.png){ width=75% }


If you click on the red error you'll see details of all logged messages during the processing of this request.  See Figure \ref{profiler_log}.

![Screenshot of Profiler log entries. \label{profiler_log}](./03_figures/part06_security/10_logs_in_profiler.png){ width=75% }

## Learn more about logger and exceptions

Learn more about Symfony and the Monolog logger:

- [Logging with Monolog](http://symfony.com/doc/current/logging.html)

Learn more about custom exception handlers and error pages:

- [Access Denied Handler](https://symfony.com/doc/current/security/access_denied_handler.html)
- [Custom Error pages](https://symfony.com/doc/current/controller/error_pages.html)
