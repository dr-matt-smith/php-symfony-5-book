# Adding a campus entity and relating them 

## Create a new class: Campus


![Class diagram for Campus class.](./03_figures/app_crud/crud11_campusClass.png){ width=75% }


Create entity Campus with single property 'location' (string)
add a    `__toString()`  method to Campus class (`src/Entity/Campus.php`) containing the following
```php
    public function __toString()
    {
        return $this->location;
    }
```
(we'll need this toString method in Campus later, so that when creating/editing Students we can choose the related Campus object from a drop down menu - which needs a string description of each Campus)

## Create CRUD for this Campus class

generate CRUD for Campus

```bash
php bin\console make:crud Campus
```

## Create relationship between Student and Campus (each student linked to one campus)


![Class diagram for Student-Campus multiplicity.](./03_figures/app_crud/crud12_studentCampus.png){ width=75% }



To create this relationship we are going to add a 'campus' property to the Student class, that is a reference to a Campus object. Here is our detailed new Student class diagram:

![Detailed Student class diagram.](./03_figures/app_crud/crud13_studentDetailedDesign.png){ width=75% }



Here is how to add a related property to a class:

add a property 'campus' to the Student class, of type relation, that is ManyToOne to the Campus class
i.e. many students linked to one campus
to ADD a property to an existing class, we need to run the `make:entity` console command again:

```bash
php bin\console make:entity Student
```
the console should see the entity already exists, and invite us to add a new property...

## update Database Structure (since we changed our classes)

1. Create and run new DB migration

1. Create a migration by typing:

    ```bash
    php bin\console make:migration
    ```
1. now run the migration by typing:

    ```bash
    php bin\console doctrine:migrations:migrate
    ```

## Delete old CRUD and generate new CRUD for both classes

1. delete the old Student CRUD 
    - FILE:     `src/Controller/StudentController.php`
    - FILE:     `src/Form/StudentType.php`
    - folder:    `templates/student`
1. generate CRUD for Student

```bash
php bin\console make:crud Student
```

## Run server add some related records:

1. now run server:

    ```bash
    php bin\console make:crud Student
    ```
1.  visit the Campus CRUD pages and create record for 2 campuses 
    - e.g. Blanch and Tallaght)

1. visit the Student CRUD pages, and edit / create Student's related to your new Campuses


![Screenshot showing list of Campus objects.](./03_figures/app_crud/crud14_campusScreenshot.png){ width=75% }

When we create/edit a student, we now get a dropdown menu of the Campus objects (the text in the dropdown menu is from the `__toString()` method we created for the Campus class).

![Screenshot showing new Student form with Campus choice dropdown menu](./03_figures/app_crud/crud15_dropDownScreenshot.png){ width=75% }



