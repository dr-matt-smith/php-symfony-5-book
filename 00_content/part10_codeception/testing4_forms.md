
# Filling out forms

## Setup database

Let's generate a `Recipe` Entity class, and its CRUD - with no security.

Generate a `Recipe` Entity class:

- title (string)

- steps (string)

- time (integer)

Also generate its CRUD, and migrate it to the database.

There should now be a new recipe form at URL `/recipe/new`.

## Cest to enter a new recipe (project `codeception04`)

Generate a new `Cest` to fill in the form for a new recipe:

```bash
    $ vendor/bin/codecept g:cest acceptance RecipeCest
```

Edit the skeleton as follows:

```php
    namespace App\Tests;    
    
    use App\Tests\AcceptanceTester;
    
    class RecipeCest
    {
        public function tryToTest(AcceptanceTester $I)
        {
            $I->amOnPage('/recipe/new');
            $I->fillField('#recipe_title', 'Boston Cheesecake');
            $I->fillField('#recipe_steps', 'buy packet - follow instructions');
            $I->fillField('#recipe_time', 60);
            $I->click('Save');
        }
    }
```

As you can see we select an HTML form input by its **id**, so `#recipe_title` is the form field for the `title` property of the new Recepie to be created etc.

When you run the server and run the acceptance tests you'll now see a new recipe in the database, based on your acceptance tests completion of the forms.

