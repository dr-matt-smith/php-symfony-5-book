

# Security users from database

## Create a `User` entity (project `security04`)

We need to create a `User` entity, that implements the Symfony Security User Interface.

Create Entity class `/src/Entity/User.php`:

```php
    namespace App\Entity;

    use Doctrine\ORM\Mapping as ORM;
    use Symfony\Component\Security\Core\User\UserInterface;

    /**
     * @ORM\Table(name="app_users")
     * @ORM\Entity(repositoryClass="App\Repository\UserRepository")
     */
    class User implements UserInterface, \Serializable
    {
        // properties and methods go here ...
    }
```

Here are all the properties needed:

```php
    /**
     * @ORM\Column(type="integer")
     * @ORM\Id
     * @ORM\GeneratedValue(strategy="AUTO")
     */
    private $id;

    /**
     * @ORM\Column(type="string", length=25, unique=true)
     */
    private $username;

    /**
     * @ORM\Column(type="string", length=64)
     */
    private $password;

    /**
     * @ORM\Column(type="json_array")
     */
    private $roles = [];
```

We need 2 special methods relating to security:

```php
    public function getSalt()
    {
        // no salt needed since we are using bcrypt
        return null;
    }

    public function eraseCredentials()
    {
    }
```

We need 2 special methods relating to serialization:

```php
    /** @see \Serializable::serialize() */
    public function serialize()
    {
        return serialize(array(
            $this->id,
            $this->username,
            $this->password,
        ));
    }

    /** @see \Serializable::unserialize() */
    public function unserialize($serialized)
    {
        list (
            $this->id,
            $this->username,
            $this->password,
        ) = unserialize($serialized);
    }
```

We need 2 special methods relating to user ROLEs:

```php
    public function getRoles()
    {
        $roles = $this->roles;
        // ensure always contains ROLE_USER
        $roles[] = 'ROLE_USER';

        return array_unique($roles);
    }

    public function setRoles($roles)
    {
        $this->roles = $roles;
        return $this;
    }
```

You can now generate standard getters and setters for all remaining properties.

## Create database and migrate

Now you can create a database based on your `.env` database settings, and migrate the Entity structure to your database schema:

```bash
    $ php bin/console doctrine:database:create
    $ php bin/console doctrine:migrations:diff
    $ php bin/console doctrine:migrations:migrate
```

## Configure security to use DB users as 'provider'

In `/config/packages/security.yml` we specify our firewalls, and also:

- where our security users are provided from `providers`

- how passwords are hashed/encrypted for security users `encoders`

We will replace our in memory provider with a database provider of objects of our `App\Entity\User` class:

```yaml
    providers:
        our_db_provider:
            entity:
                class: App\Entity\User
                property: username
```

We will replace the plaintext encoder with a `bcrypt` algorithm for objects of our `App\Entity\User` class:

```yaml
    encoders:
        App\Entity\User:
            algorithm: bcrypt
```

So our complete `security.yml` file now reads as follows:


```yaml
    security:
        providers:
            our_db_provider:
                entity:
                    class: App\Entity\User
                    property: username

        encoders:
            App\Entity\User:
                algorithm: bcrypt

        firewalls:
            dev:
                pattern: ^/(_(profiler|wdt)|css|images|js)/
                security: false
            main:
                anonymous: true
                provider: our_db_provider
                form_login:
                    login_path: login
                    check_path: login

                logout:
                    path:   /logout
                    target: /
```

## Initial values for your project database

Fixtures play two roles:

- inserting initial values into your database (e.g. the first `admin` user)
- setting up the database to a known state for **testing** purposes

Doctrine provides a Symfony fixtures **bundle** that makes things very straightforward.

Learn more about Symfony fixtures at:

- [Symfony website fixtures page](https://symfony.com/doc/master/bundles/DoctrineFixturesBundle/index.html)

## Instal Doctrine fixtures package

Use Composer to add the Doctrine fixtures package to the project:

```bash
    composer req doctrine/doctrine-fixtures-bundle
```

## Writing the fixture classes

We need to locate our fixtures in directory `/src/DataFixtures`.

We need to write a class that extends the Doctrine `Fixtures` class.

We now need to create class `LoadUsers` in file `/src/DataFixtures/LoadUsers.php`:

```php
    namespace App\DataFixtures;

    use App\Entity\User;
    use Doctrine\Bundle\FixturesBundle\Fixture;
    use Doctrine\Common\Persistence\ObjectManager;
    use Symfony\Component\Security\Core\Encoder\UserPasswordEncoderInterface;

    class LoadUsers extends Fixture
    {
        // properties and methods go here ...
    }
```

We need a private property `$encoder` that will allow us to hash passwords before their storage:
```php
    /**
     * @var UserPasswordEncoderInterface
     */
    private $encoder;

    public function __construct(UserPasswordEncoderInterface $encoder)
    {
        $this->encoder = $encoder;
    }
```

We need a `load(...)` method, that gets invoked when we are loading fixtures from the CLI. This method creates objects for the entities we want in our database, and the saves (persists) them to the database:
```php
    public function load(ObjectManager $manager)
    {
        // create objects
        $userUser = $this->createUser('user', 'user');
        $userAdmin = $this->createUser('admin', 'admin', ['ROLE_ADMIN']);
        $userMatt = $this->createUser('matt', 'smith', ['ROLE_ADMIN']);

        // store to DB
        $manager->persist($userUser);
        $manager->persist($userAdmin);
        $manager->persist($userMatt);
        $manager->flush();
    }
```

Rather than put all the work in the `load(...)` method, we can create a helper method to create each new object. Method `createUser(...)` creates and returns a reference to a new `User` object given some parameters:

```php
    private function createUser($username, $plainPassword, $roles = ['ROLE_USER']):User
    {
        $user = new User();
        $user->setUsername($username);
        $user->setRoles($roles);

        // password - and encoding
        $encodedPassword = $this->encodePassword($user, $plainPassword);
        $user->setPassword($encodedPassword);

        return $user;
    }
```

NOTE: The default role is `ROLE_USER` if none is provided.


Finally, to make it clear how we are encoding the password, we have method `encodePassword(...)`, returning an encoded password given a `User` object and a plain text password:

```php
    private function encodePassword($user, $plainPassword):string
    {
        $encodedPassword = $this->encoder->encodePassword($user, $plainPassword);
        return $encodedPassword;
    }
```

## Loading the fixtures

Loading fixtures involves deleting all existing database contents and then creating the data from the fixture classes - so you'll get a warning when loading fixtures. At the CLI type:

```bash
    php bin/console doctrine:fixtures:load
```

That's it!

You should now be able to access `/admin` with either the `matt/smith` or `admin/admin` users. You will get an Access Denied exception if you login with `user/user`, since that only has `ROLE_USER` privileges, and `ROLE_ADMIN` is required to visit `/admin`.

See Figure \ref{denied_exception} to see the default Symfony (dev mode) Access Denied exception page.

![Screenshot of Default Symfony access denied page. \label{denied_exception}](./03_figures/part06_security/8_access_denied.png){ width=75% }

The next chapter will show you how to deal with (and log) access denied exceptions ...

## Using SQL from CLI to see users in DB

To double check your fixtures have been created correctly in the database, you could run an SQL query from the CLI:

```bash
    $ php bin/console doctrine:query:sql "SELECT * FROM app_users"

    /../security04_db_users/vendor/doctrine/common/lib/Doctrine/Common/Util/Debug.php:71:
    array (size=3)
      0 =>
        array (size=6)
          'id' => string '1' (length=1)
          'username' => string 'user' (length=4)
          'password' => string '$2y$13$uLxKuVGLJnnKzXmlmCizf.scKM5rm87w9WPlatk2g8KXrCDOtSIvy' (length=60)
          'roles' => string '["ROLE_USER"]' (length=13)
      1 =>
        array (size=6)
          'id' => string '2' (length=1)
          'username' => string 'admin' (length=5)
          'password' => string '$2y$13$xTIs6Fmt9ZPeKU0RUWWIkO6Wt9SlFZEZnrhbbE3yw5BCx5aLwgE1a' (length=60)
          'roles' => string '["ROLE_ADMIN"]' (length=14)
      2 =>
        array (size=6)
          'id' => string '3' (length=1)
          'username' => string 'matt' (length=4)
          'password' => string '$2y$13$wSciYVsT5HwAws69wwe//ObWfj3RufGVuhw01hvjLkkSqCR5hWaha' (length=60)
          'roles' => string '["ROLE_ADMIN"]' (length=14)

```