

# Codeception Symfony DB testing

## Adding Symfony and Doctrine to the settings (project `codeception05`)

Let's configure Codeception to use Doctrine for DB work, and to recognise Symfony controllers.

Note - by stating `cleanup:true` for the Doctrine2 module, we are asking Codeception to create a database **transaction** before each test runs, and then to roll-back the translation after the test runs. So the database should be in the same state **after** each test as it was **before**. Thus, our tests are not linked and do not rely on running in a set sequence.

We need to edit file `/tests/acceptqance.yml.suite` to add the following:

```yaml
    - Symfony:
        app_path: 'src'
        environment: 'test'
    - Doctrine2:
        depends: Symfony
        cleanup: true
    - Asserts
```

So it should now contain:

```yaml
    actor: AcceptanceTester
    modules:
        enabled:
            - PhpBrowser:
                url: http://localhost:8000
            - \App\Tests\Helper\Acceptance
            - Symfony:
                app_path: 'src'
                environment: 'test'
            - Doctrine2:
                depends: Symfony
                cleanup: true
            - Asserts
```

## Test Users in DB from Fixtures

Generate a new Acceptance test `UserCest`:

```bash
codecept g:cest acceptance UserCest
```

Edit this new file `/tests/acceptance/UserCest.php` to contain the following:

```php
    <?php 
    namespace App\Tests;
    use App\Tests\AcceptanceTester;
    
    class UserCest
    {
    
        public function testUsersInDb(AcceptanceTester $I)
        {
            $I->seeInRepository('App\Entity\User', [
                'username' => 'user'
            ]);
            $I->seeInRepository('App\Entity\User', [
                'username' => 'admin'
            ]);
            $I->seeInRepository('App\Entity\User', [
                'username' => 'matt'
            ]);
    
        }
    }
```

When you run the test should pass (assuming these users are in the DB from the Fixtures setup).

## Check DB reset after each test

Add 2 more test methods, one to create a new user, and test it exists, and then one to check that that new user was removed when its test finished runing:

```php
    
    public function testAddToDatabase(AcceptanceTester $I)
    {
        $I->haveInRepository('App\Entity\User', [
            'username' => 'userTEMP',
            'password' => 'sdf',
            'roles' => ['ROLE_USER']
        ]);

        $I->seeInRepository('App\Entity\User', [
            'username' => 'userTEMP'
        ]);
    }

    public function testTEMPNoLongerInDatabase(AcceptanceTester $I)
    {

        $I->dontSeeInRepository('App\Entity\User', [
            'username' => 'userTEMP'
        ]);
    }
```

## Add DB counts to our form-filling Acceptance test

Let's add code to count number of recipes int he DB before and after filling in the form to cfreatge a new recipe.

First, let's ensure there is one recipe in the data through a fixture:

```php
    namespace App\DataFixtures;
    
    use Doctrine\Bundle\FixturesBundle\Fixture;
    use Doctrine\Common\Persistence\ObjectManager;
    
    use App\Entity\Recipe;
    
    class RecipeFixtures extends Fixture
    {
        public function load(ObjectManager $manager)
        {
             $recipe = new Recipe();
             $recipe->setTitle('Boston Cheesecake');
             $recipe->setSteps('open packge - follow instructions');
             $recipe->setTime(120);
    
             $manager->persist($recipe);
    
            $manager->flush();
        }
    }
```

And load the fixtures via the Symfony console Doctrine command.

Now we can add to our Recipe form-filling acceptance test to count 1 recipe initially, then 2 after the new one has been created by filling in the form:

```php
    public function tryToTest(AcceptanceTester $I)
    {
        // --- expect 1 in DB from fixturs
        $expectedCount = 1;
        $users = $I->grabEntitiesFromRepository('App\Entity\Recipe');
        $numRecipes = count($users);

        // assert
        $I->assertEquals($expectedCount, $numRecipes);

        // --- create new recipe via FORM
        // title suffix RANDOM numbner: 'Boston Cheesecake<randNum>'
        $randomNumber = rand(1,100);
        $recipeTitle = 'Boston Cheesecake' . $randomNumber;

        $I->amOnPage('/recipe/new');
        $I->fillField('#recipe_title', $recipeTitle);
        $I->fillField('#recipe_steps', 'buy packet - follow instructions');
        $I->fillField('#recipe_time', 60);
        $I->click('Save');

        // ---- check added to repository
        $I->seeInRepository('App\Entity\Recipe', [
            'title' => $recipeTitle
        ]);


        // --- now should be 2 in DB
        $expectedCount = 2;
        $users = $I->grabEntitiesFromRepository('App\Entity\Recipe');
        $numRecipes = count($users);

        // assert
        $I->assertEquals($expectedCount, $numRecipes);
    }
```

<!--
## Setup database

using Doctrine and Symfony

https://codeception.com/docs/05-UnitTests#Interacting-with-the-Framework


In this case you can use the methods from the Doctrine2 module, while Doctrine itself uses the Symfony module to establish connections to the database. In this case a test might look like:

<?php
function testUserNameCanBeChanged()
{
    // create a user from framework, user will be deleted after the test
    $id = $this->tester->haveInRepository(User::class, ['name' => 'miles']);
    
    // get entity manager by accessing module
    $em = $this->getModule('Doctrine2')->em;

    // get real user
    $user = $em->find(User::class, $id);
    $user->setName('bill');
    $em->persist($user);
    $em->flush();
    $this->assertEquals('bill', $user->getName());

    // verify data was saved using framework methods
    $this->tester->seeInRepository(User::class, ['name' => 'bill']);
    $this->tester->dontSeeInRepository(User::class, ['name' => 'miles']);
}

In both examples you should not be worried about the data persistence between tests. The Doctrine2 and Laravel5 modules will clean up the created data at the end of a test. This is done by wrapping each test in a transaction and rolling it back afterwards.



## Faker

1. Generate a `Helper` class:

    ```bash
         vendor/bin/codecept generate:helper Factories
    ```

    - this should create `tests/_support/Helper/Factories.php `    
    
1. Installt the Faker **Facade**:



    ```bash
        composer req league/factory-muffin-faker
    ```
    
    
1. Install version 2.1 of the PHP League `FactoryMuffin`:

    ```bash
        composer req league/factory-muffin
    ```
-->
