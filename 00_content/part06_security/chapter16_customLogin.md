
# Custom login page

## A traditional login form  (project `security05`)

Symfony has a maker for login forms and security controllers, so let's use it. You can read more at:

- [https://symfony.com/doc/current/security/form_login_setup.html](https://symfony.com/doc/current/security/form_login_setup.html)

At the command line tell Symfony to generate some authentication code for us:

```bash
    $ php bin/console make:auth
```

Choose option 1, for a Login Form Authenticator:

```bash
    What style of authentication do you want? [Empty authenticator]:
      [0] Empty authenticator
      [1] Login form authenticator
     > 1
```

Next, enter a meaningful class name of `LoginFormAuthenticator`:

```bash
    The class name of the authenticator to create (e.g. AppCustomAuthenticator):
    > LoginFormAuthenticator
```

Accept the default name for a `SecurityController` (just press `<RETURN>`):

```bash
    Choose a name for the controller class (e.g. SecurityController) [SecurityController]:
    > 
```

Symfony should now update your `config/packages/security.yaml`, and generate a `SecurityController`, as well as a `Security/LoginFormAuthenticator`, and a login Twig form:

```bash
    created: src/Security/LoginFormAuthenticator.php
    updated: config/packages/security.yaml
    created: src/Controller/SecurityController.php
    created: templates/security/login.html.twig
       
    Success! 
```

## Simplifying the generated login Twig template

The generated Twig login page is fine - but you should become confident in making it your own. Replace the generated code with the following, simplified code - you should be able to see that the only complicated bits are the CSRF token, and the error messages:

```twig
    {% extends 'base.html.twig' %}
    
    {% block title %}Log in!{% endblock %}
    
    {% block body %}
    <form method="post">
        <input type="hidden" name="_csrf_token" value="{{ csrf_token('authenticate') }}">
    
        {% if error %}
            <div style="background-color: pink; padding: 1rem;">
                {{ error.messageKey|trans(error.messageData, 'security') }}
            </div>
        {% endif %}
    
        <h1>Login</h1>
    
        Username:
        <input value="{{ last_username }}" name="username" id="inputUsername" autofocus>
    
        <p>
            Password:
        <input type="password" name="password" id="inputPassword">
    
        <p>
        <button type="submit">Login</button>
    </form>
    {% endblock %}

```

Above we can see the following in our Login Twig template:

- the HTML `<form>` open tag, which we see submits via HTTP `POST` method

    - no action is given, so the form will submit to the same URL as displayed the form (`/login`), but with a `POST` method

- add the security CSRF token as a hidden form variable    
- display of any Twig `error` variable received
- the `username` label and text input field (and value of the `last_username` if any)
- the `password` label and password input field
- the submit button named `login`

## Custom login form when attempting to access `/admin`

See Figure \ref{custom_login_form} to see our custom login form in action.

![Screenshot of custom login form. \label{custom_login_form}](./03_figures/part06_security/5b_custom_login.png){ width=75% }


## Path for successful login

If the user visits the path `/login` directly in the browser, Symfony needs to know where to direct the user if login is successful. This is defined in method `onAuthenticationSuccess` in class `Security/LoginFormAuthenticator`. If no redirect is defined, then the `TODO` Exception will be thrown:

```php
        throw new \Exception('TODO: provide a valid redirect inside '.__FILE__);
```

Since we have a secure **admin** page, then let's redirect to route `admin`:

```php
    public function onAuthenticationSuccess(Request $reques§t, TokenInterface $token, $providerKey)
    {
        if ($targetPath = $this->getTargetPath($request->getSession(), $providerKey)) {
            return new RedirectResponse($targetPath);
        }

        return new RedirectResponse($this->urlGenerator->generate('staff'));
    }
```

If you want to redirect to different pages, depending on the **role** of the newly logged-in user, then do the following:

- get the array of string roles from `$token` with `$token->getRoles()`

- add `IF`-statement(s) returning a different named route depending on their role, e.g. something   like:

    ```php
        if(in_array('ROLE_ADMIN', $roles){        
            return new RedirectResponse($this->urlGenerator->generate('index_admin'));
        }        
        return new RedirectResponse($this->urlGenerator->generate('index_staff'));
    ```


## CSRF protection

CSRF = Cross Site Request Forgery

NOTE: For any public **production** site you should always implement CSRF protection. This is implemented using CSRF 'tokens' created on the server and exchanged with the web client and form submissions. CSRF tokens help protect web applications against cross-site scripting request forgery attacks and forged login attacks.

Add the following before the `<button>` in your `login.html.twig`:

```twig                
    <input type="hidden" name="_csrf_token"
           value="{{ csrf_token('authenticate') }}"
    >    
```

Learn more about CSRF threats and security:

- [Symfony CSRF protection](https://symfony.com/doc/current/security/csrf.html)
- [Wikipedia](https://en.wikipedia.org/wiki/Cross-site_request_forgery#Forging_login_requests)

When using the Symfony generated login form (as wee created in this chapter) the CSRF token protection is built-in automatically.

