
# Quickstart Symfony security

## Learn about Symfony security

There are several key Symfony reference pages to read when starting with security. These include:

- [Introduction to security](http://symfony.com/doc/current/security.html)

- [How to build a traditional login form](http://symfony.com/doc/current/security/form_login_setup.html)

- [Using CSRF protection](http://symfony.com/doc/current/security/csrf_in_login_form.html)



## Create a new project from scratch (project `security01`)

Create a new project, adding the usual packages for database and CRUD generation:

- server
- make
- twig
- annotations
- doctrine
- form
- validation
- debug

Also add following package to allow us to work with security and its annotations:

- security

NOTE: You may also need these 2 - but try without, and let me know :-) - they all be part of the `security` collection.

```bash
    $ composer req sensio/framework-extra-bundle symfony/expression-language
```

## Make a Default controller

Let's make a Default controller `/src/Controller/DefaultController.php`:

```bash
    $ php bin/console make:controller Default
```

Edit the route to be `/` and the internal name to be `homepage`:

```php
    /**
     * @Route("/", name="homepage")
     */
    public function indexAction()
    {
        $template = 'default/index.html.twig';
        $args = [];
        return $this->render($template, $args);
    }
```

Change the template to be something like:

```twig
    {% extends 'base.html.twig' %}

    {% block body %}
        welcome to the home page
    {% endblock %}
```

This will be accessible to everyone.

## Make a secured Admin controller

Let's make a Admin controller:

```bash
    $ php bin/console make:controller Admin
```

This will be accessible to only to users logged in with `ROLE_ADMIN` security.

Edit the new `AdminController` in `/src/Controller/AdminController.php`. Add a `use` statement, to let us use the `@IsGranted` annotation:

```php
    use Sensio\Bundle\FrameworkExtraBundle\Configuration\IsGranted;
```

Now we'll restrict access to the index action of our Admin controller using the `@IsGranted` annotation:

```php
    /**
     * @Route("/admin", name="admin")
     * @IsGranted("ROLE_ADMIN")
     */
    public function index()
    {
        $template = 'admin/index.html.twig';
        $args = [];
        return $this->render($template, $args);
    }
```


Change the template to be something like:

```twig
    {% extends 'base.html.twig' %}

    {% block body %}
        <h1>Admin home</h1>

        here is the secret code to the safe:
        007123
    {% endblock %}
```

That's it!

Run the web sever:

- visiting the Default page at `/` is fine, even though we have not logged in ag all

- however, visiting the the `/admin` page should result in an HTTP 401 error (Unauthorized) due to insufficient authentication. See Figure \ref{not_authorised}.

![Screenshot of error attempting to visit `/admin`. \label{not_authorised}](./03_figures/part06_security/1_401_error.png){ width=75% }

Of course, we now need to add a way to login and define different user credentials etc...

## Core features about Symfony security

There are several related features and files that need to be understood when using the Symnfony security system. These include:

- **firewalls**
- **providers** and **encoders**
- **route protection** (we met this with `@IsGranded` controller method annotation comment above...)
- user **roles** (we met this as part of `@IsGranded` above `("ROLE_ADMIN")` ...)

Core to Symfony security are the **firewalls** defined in `/config/packages/security.yml`. Symfony firewalls declare how route patterns are protected (or not) by the security system. Here is its default contents (less comments - lines starting with hash `#` character):

```yaml
    security:
        providers:
          in_memory: { memory: ~ }

        firewalls:
            dev:
                pattern: ^/(_(profiler|wdt)|css|images|js)/
                security: false
            main:
                anonymous: true
```

Symfony considers **every** request to have been authenticated, so if no login action has taken place then the request is considered to have been authenticated to be **anonymous** user `anon`. We can see in this `anon` user in Figure \ref{anon_user} this looking at the user information from the Symfony debug bar when visiting the default home page.

![Symfony profiler showing anonymous user authentication. \label{anon_user}](./03_figures/part06_security/4_anon_authentication.png){ width=50% }


A Symfony **provider** is where the security system can access a set of defined users of the web application. The default is simply `in_memory` - although usually larger applications have users in a database or from a separate API. We see that the `main` firewall simply states that users are permitted (at present) any request route pattern, and anonymous authenticated users (i.e. ones who have not logged in) are permitted.

The `dev` firewall allows Symfony development tools (like the profiler) to work without any authentication required. Leave it in `security.yml` and just ignore the `dev` firewall from this point onwards.

## Defining some users and HTTP basic login (project `security02`)

We control security through file `/config/packages/security.yml'.

Let's define 3 users^[Ensure you always prefix security roles with `ROLE_`, to ensure they are processed by Symfony's security system.]:

- `user` has password `user`, and the security role ROLE_USER

- `admin` has password `admin`, and the security role ROLE_ADMIN

- `matt` has password `smith`, and the security role ROLE_ADMIN

The simplest way is to define them in `security.yml` as 'in memory' users:


replace:

```yaml
    in_memory: { memory: ~ }
```

with:
```yaml
    in_memory:
          memory:
              users:
                  user:
                      password: user
                      roles: 'ROLE_USER'
                  admin:
                      password: admin
                      roles: 'ROLE_ADMIN'
                  matt:
                      password: smith
                      roles: 'ROLE_ADMIN'
```

We also must state how these user's passwords are encoded (or not):

```yaml
    security:
        encoders:
            Symfony\Component\Security\Core\User\User: plaintext
    
        # https://symfony.com/doc/current/security.html#where-do-users-come-from-user-providers
        providers:
            ... as before
```

Finally, we need some kind of login form. The simplest is the basic HTTP login form built into web browsers.
So remove the hash `#` comment in line:

```yaml
  http_basic: true
```

So your complete `security.yml` should look as follows:

```yaml
    security:
        encoders:
            Symfony\Component\Security\Core\User\User: plaintext
    
        providers:
            in_memory:
                memory:
                    users:
                        user:
                            password: user
                            roles: 'ROLE_USER'
                        admin:
                            password: admin
                            roles: 'ROLE_ADMIN'
                        matt:
                            password: smith
                            roles: 'ROLE_ADMIN'
        firewalls:
            dev:
                pattern: ^/(_(profiler|wdt)|css|images|js)/
                security: false
            main:
                anonymous: true
    
                http_basic: true
```

See Figure \ref{basic_login} to see the Chrome browser's basic HTTP built-in login form.

![Screenshot of Chrome HTTP basic login form. \label{basic_login}](./03_figures/part06_security/3_basic_login.png){ width=75% }

See Figure \ref{admin_profiler} to see the logged-in user in the Symfony page footer profiler.

![Screenshot of Profiler showing admin login. \label{admin_profiler}](./03_figures/part06_security/2_profiler.png){ width=75% }

## Click user in Profile bar to see ROLE

If you click the user in the Symfony profiler footer, the Profiler will show you details of the logged-in user, including their `ROLE_`. See Figure \ref{user_role}.

![Screenshot of Profiler showing user role details. \label{user_role}](./03_figures/part06_security/7_user_role.png){ width=75% }

