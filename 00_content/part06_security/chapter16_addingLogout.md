
# Allowing user to logout  (project `security03`)

## Adding a `/logout` route

We can define a route to **logout** very easily in Symfony, with no need for any controller method. In `/config/routes.yaml` we add our login route, and its redirect to the website home page `/`. These are the only 2 lines we need in this file (for our simple demo application):

```yaml
    app_logout:
      path: /logout
```

We also need to define the logout route as part of our statement:

```yaml
    firewalls:
        dev:
            pattern: ^/(_(profiler|wdt)|css|images|js)/
            security: false
        main:
            anonymous: true

            http_basic: true
            logout:
                path:   app_logout
                invalidate_session: true
```
 
NOTE: The last line above explicitly asks Symfony to invalidate the user token in the session, as soon as the logout route is visited. This is meant to be the default, but sometimes this extra line seems to be needed to avoid the user still being logged in for the next request.


We can  enter the route directly in the browser address bar, e.g. via URL:

```
    http://localhost:8000/logout
```

Figure \ref{logout_link} shows that we can see the logout route is also available from the Symfony profile toolbar. 

![Symfony profiler user logout action. \label{logout_link}](./03_figures/part06_security/6_logout_profiler.png){ width=75% }


In either case we'll logout any currently logged-in user, and return the anonymously authenticated user `anon` with no defined authentication roles.

