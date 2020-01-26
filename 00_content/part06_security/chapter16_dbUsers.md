
# Symfony Database Users

## Creating `User` Entity Class (project `security04`)

Much of the security features of Symfony have been automated since Symfony 4.2.

First let's generate an approprite `User` Entity class:

```bash
    $ php bin/console make:user
```

Choose the default class name `User`, and to store users in the database (just hit `<RETURN>`):
```bash
     The name of the security user class (e.g. User) [User]:
     > 
     Do you want to store user data in the database (via Doctrine)? (yes/no) [yes]:
     >     
```

Rather than email, enter `username` for the unique user property:
```bash
     Enter a property name that will be the unique "display" name for the user (e.g. email, username, uuid) [email]:
     > username
```


Choose the default to hash passwords, and to store users in the database (just hit `<RETURN>`):    
```bash
     Will this app need to hash/check user passwords? Choose No if passwords are not needed or will be checked/hashed by some other system (e.g. a single sign-on server).
    
     Does this app need to hash/check user passwords? (yes/no) [yes]:
     > 
```

At the command line Symfony should confirm that `/config/packages/security.yaml` has been updated, and the `User` and `UserRepository` classes created:

```bash
     created: src/Entity/User.php
     created: src/Repository/UserRepository.php
     updated: src/Entity/User.php
     updated: config/packages/security.yaml
```

If you look at `security.yaml` you'll see that Entity class `User` has been declared as a **provider** of authenticated users, and that passwords for this class are to be encoded using the `bcrypt` hashing algorithm:

```yaml
    security:
        encoders:
            App\Entity\User:
                algorithm: bcrypt
    
    
        providers:
            # used to reload user from session & other features (e.g. switch_user)
            app_user_provider:
                entity:
                    class: App\Entity\User
                    property: username
```

## Create/migrate your database

Ensure the database settings are correct in the project's `.env` file, and migrate this new `User` Entity class with:

```bash
    php bin/console doctrine:migtrations:diff
    php bin/console doctrine:migtrations:migrate
```

You now have a `user` table in your database, ready for some fixtures...

## User fixtures - adding some users in the database

First generate a new Fixtures class `UserFixtures`:

```bash
    $ php bin/console make:fixtures
    
    The class name of the fixtures to create (e.g. AppFixtures):
    > UserFixtures
    
    created: src/DataFixtures/UserFixtures.php
```

Since we are encoding passwords, we need to add a `use` statement in class `` to allow us to make use of the Symfony passsword encoder interface class. We also need to add a `use` statement since we'll be creating `User` objects to be inserted into the database:

```php
    use Symfony\Component\Security\Core\Encoder\UserPasswordEncoderInterface;
    use App\Entity\User;
```

We will need an object that can encode passwords for us. In the class declare a private property `$passwordEncoder`:
```php
    class UserFixtures extends Fixture
    {
         private $passwordEncoder;    
```

We can use Symfony's ParaConverter to create a `$passwordEncoder` object for us. Next create a constructor method, that declares a parameter `UserPasswordEncoderInterface $passwordEncoder`, we can then store the created object in the private property:

```php
    class UserFixtures extends Fixture
    {
         private $passwordEncoder;
    
         public function __construct(UserPasswordEncoderInterface $passwordEncoder)
         {
             $this->passwordEncoder = $passwordEncoder;
         }
```

We can now write the content of the `load()` method to create a new `User` object, with userame `matt`, role 'ROLE_ADMIN' and an encoded password `smith`:

```php
    public function load(ObjectManager $manager)
    {
        $user1 = new User();
        $user1->setUsername('matt');
        $user1->setRoles(['ROLE_ADMIN']);
        $plainTextPassword = 'smith';
        $encodedPassword = $this->passwordEncoder->encodePassword($user1, $plainTextPassword);
        $user1->setPassword($encodedPassword);
        
        $manager->persist($user1);
        $manager->flush();
    }
```

Let's load these fixtures:

```bash
    php bin/console doctrine:fixtures:load
```

You should now be able to visit `/admin`, be presented with the built-in HTTP login screen, and log in with username `matt` and password `smith`.

## Make life easier with method `createUser()`

Since we may with so create several users, let's put that logic into a separate method:

```php
    private function createUser($username, $plainPassword, $roles = ['ROLE_USER']):User
    {
        $user = new User();
        $user->setUsername($username);
        $user->setRoles($roles);

        // password - and encoding
        $encodedPassword = $this->passwordEncoder->encodePassword($user, $plainPassword);
        $user->setPassword($encodedPassword);

        return $user;
    }
```

Now we can create several users very clearly in our `load()` method:

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

Re-run the fixtures loading, and you should have 3 users in your database.

You should now be able to access `/admin` with either the `matt/smith` or `admin/admin` users. You will get an Access Denied exception if you login with `user/user`, since that only has `ROLE_USER` privileges, and `ROLE_ADMIN` is required to visit `/admin`.

## Using SQL from CLI to see users in DB

To double check your fixtures have been created correctly in the database, you could run an SQL query from the CLI:

```bash
    $ php bin/console doctrine:query:sql "SELECT * FROM user"

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