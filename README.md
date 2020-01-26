# php-symfony4-book

An introduction to Symfony4

have fun coding

.. matt ..



composer req  / --dev

Typical new project needs:

server make twig annotations doctrine form validation annotations security-csrf

Also handy:
 koff/crud-maker-bundle

When working with Security & Routes also add the Framework Extra Bundle (This gives @Security and @Method extra annotations, & Expression Lanaguge
):
 security
 sensio/framework-extra-bundle
 symfony/expression-language

1. First add the basics

```
    composer req server make twig annotations doctrine form validation annotations security-csrf koff/crud-maker-bundle
```

2. then add security adn framrwork bundle - otherwise may get older version?

```
    composer req security sensio/framework-extra-bundle  symfony/expression-language
```
