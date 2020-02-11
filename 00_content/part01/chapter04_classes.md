

# Creating our own classes

## Goals

Our goals are to:

- create a simple Student entity class (by hand - not using the **make** tool)
- create a route / controller / template to show one student on a web page
- create a repository class, to manage an array of Student objects
- create a route / controller / template to list all students as a web page
- create a route / controller / template to show one student on a web page for a given Id

## Let's create an Entity `Student` (`basic03`)

Entity classes are declared as PHP classes in `/src/Entity`, in the namespace `App\Entity`. So let's create a simple `Student` class:

```php
    <?php
    namespace App\Entity;

    class Student
    {
        private $id;
        private $firstName;
        private $surname;
    }
```

That's enough typing - use your IDE (E.g. PHPStorm) to generate a public constructor (taking in values for all 3 properties), and also public getters/setters for each property.

## Create a StudentController class

Generate a StudentController class:

```bash
    $ php bin/console make:controller Student
```

It should look something like this (`/src/Controller/StudentController.php`):

```php
    <?php
    
    namespace App\Controller;
    
    use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
    use Symfony\Component\Routing\Annotation\Route;
    
    class StudentController extends AbstractController
    {
        /**
         * @Route("/student", name="student")
         */
        public function index()
        {
            ... default code here ...
        }
    }
```

NOTE!!!!: When adding new routes, it's a good idea to **CLEAR THE CACHE**, otherwise Symfony may not recognised the new or changed routes ... Either manually delete the `/var/cache` directory, or run the `cache:clear` console command:

```bash
    $ php bin/console cache:clear

    // Clearing the cache for the dev environment with debug true
    [OK] Cache for the "dev" environment (debug=true) was successfully cleared.
```


Let's make this create a student (`1`, `matt`, `smith`) and display it with a Twig template (which we'll write next!). We will also improve the route internal name, changing it to `student_show`, and change the method name to `show()`. So your class (with its `use` statements, especially for `App\Entity\Student`) looks as follows now:

```php
    namespace App\Controller;
    
    use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
    use Symfony\Component\Routing\Annotation\Route;
    use App\Entity\Student;
    
    class StudentController extends AbstractController
    {
        /**
         * @Route("/student", name="student")
         */
        public function index()
        {
            $student = new Student();
            $student->setId(99);
            $student->setFirstName('matt');
            $student->setSurname('Smith');
            
            $template = 'student/index.html.twig';
            $args = [
                'student' => $student
            ];
            return $this->render($template, $args);
        }
    }

```

NOTE:: Ensure your code has the appropriate `use` statement for the `App\Entity\Student` class - a nice IDE like PHPStorm will add this for you...

## The show student template `/templates/student/show.html.twig`

In folder `/templates/student` create a new Twig template `show.html.twig` containing the following:

```twig
    {% extends 'base.html.twig' %}

    {% block body %}
        <h1>Student SHOW page</h1>

        <p>
            id = {{ student.id }}
            <br>
            name = {{ student.firstName }} {{ student.surname }}
        </p>
    {% endblock %}
```

Do the following:

- Run the web server

    `symnfony serve`
 
- Visit `/student`

    -  you should see our student details displayed as a nice HTML page.


Figure \ref{student_show} shows a screenshot our student details web page.

![Screenshot of student show page. \label{student_show}](./03_figures/lab03_entities/1_show_student.png)

## Twig debug `dump(...)` function

A very useful feature of Twig is its `dump(...)` function. This outputs to the web page a syntax colored dump of the variable its passed. It's similar to the PHP `var_dump(...)` function. Figure \ref{twig_debug} shows a screenshot of adding the following to our `index.html.twig` template:

```twig
    {% block body %}
        <h1>Student SHOW page</h1>
        <p>
            id = {{ student.id }}
            <br>
            name = {{ student.firstName }} {{ student.surname }}
        </p>
    
        {{ dump (student) }}
    {% endblock %}
```

![Screenshot of student show page. \label{twig_debug}](./03_figures/part01/4_twigDebug.png)


## Creating an Entity Repository (`basic04`)

We will now move on to work with an **array** of `Student` objects, which we'll make easy to work with by creating a `Repository` class.
Let's create the `StudentRepository` class to work with collections of Student objects. Create PHP class file `StudentRepository.php` in directory `/src/Repository`:

```php
    <?php
    namespace App\Repository;

    use App\Entity\Student;

    class StudentRepository
    {
        private $students = [];

        public function __construct()
        {
            $id = 1;
            $s1 = new Student();
            $s1->setId(1);
            $s1->setFirstName('matt');
            $s1->setSurname('smith');
            $this->students[$id] = $s1;
    
            $id = 2;
            $s2 = new Student();
            $s2->setId(2);
            $s2->setFirstName('joelle');
            $s2->setSurname('murphy');
            $this->students[$id] = $s2;
    
            $id = 3;
            $s3 = new Student();
            $s3->setId(3);
            $s3->setFirstName('frances');
            $s3->setSurname('mcguinness');
            $this->students[$id] = $s3;
        }

        public function findAll()
        {
            return $this->students;
        }
    }
```

## The student list controller method

Let's replace the contents of our `index()` method in the `StudentController` class, with one that will retrieve the array of student records from an instance of `StudentRepository`, and pass that array to our Twig template. The Twig template will loop through and display each one. 

Replace the existing method `index()` of controller class `StudentController` with the following:

```php
    ... 

    class StudentController extends AbstractController
    {
        ...
    
        /**
         * @Route("/student", name="student")
         */
        public function index()
        {
            $studentRepository = new StudentRepository();
            $students = $studentRepository->findAll();
    
            $template = 'student/index.html.twig';
            $args = [
                'students' => $students
            ];
            return $this->render($template, $args);
        }
```

So our routes remain the same, with the URL pattern `/student` being routed to our `StudentController->index()` method:

```bash
    $ php bin/console debug:router

     -------------------------- -------- -------- ------ -----------------------------------
      Name                       Method   Scheme   Host   Path
     -------------------------- -------- -------- ------ -----------------------------------
      _... (lots of other debug profiler routes)
      homepage                   ANY      ANY      ANY    /
      student                    ANY      ANY      ANY    /student
```

## The list student template `/templates/student/index.html.twig`

In  directory `/templates/student` replace the contents of Twig template `index.html.twig` with the following:

```twig
    {% extends 'base.html.twig' %}

    {% block body %}
        <h1>Student LIST page</h1>

        <ul>
            {% for student in students %}
                <li>
                    id = {{ student.id }}
                    <br>
                    name = {{ student.firstName }} {{ student.surname }}
                </li>
            {% endfor %}
        </ul>
    {% endblock %}
```

Run the web server and visit `/student`, and you should see a list of all student details displayed as a nice HTML page.


Figure \ref{student_list} shows a screenshot our list of students web page.

![Screenshot of student list page. \label{student_list}](./03_figures/lab03_entities/2_list_student.png){width=60%}

## Refactor show action to show details of one Student object  (project `basic05`)

The usual convention for CRUD is that the **show** action will display the details of an object given its `id`. So let's write a new `StudentController` method `show()` to do this. We'll need to add a `findOne(...)` method to our repository class, that returns an object given an id.

The route we'll design will be in the form `/student/{id}`, where `{id}` will be the integer `id` of the object in the repository we wish to display. And, coincidentally, this is just the correct syntax for routes with parameters that we write in the annotation comments to define routes for controller methods in Symfony ...

NOTE: We'll give this **show** route the internal name `student_show` - these internal route names are used when we create links between pages in our Twig templates, and so it's important to name them meaninfully and consistently to make later coding straightforward.

```php
    /**
     * @Route("/student/{id}", name="student_show")
     */
    public function show($id)
    {
        $studentRepository = new StudentRepository();
        $student = $studentRepository->find($id);

        // we are assuming $student is not NULL....

        $template = 'student/show.html.twig';
        $args = [
            'student' => $student
        ];
        return $this->render($template, $args);
    }
```

While we are at it, we'll change the route for our list action, to make a list of students the default for a URL path starting with `/student`:

```php
        /**
         * @Route("/student", name="student_list")
         */
        public function list()
        {
            ... as before
        }
```

We can check these routes via the console:

- `/student/{id}` will invoke our `show($id)` method
- `/student` will invoke our `list()` method

```bash
     -------------------------- -------- -------- ------ -----------------------------------
      Name                       Method   Scheme   Host   Path
     -------------------------- -------- -------- ------ -----------------------------------
      _... (lots of other debug profiler routes)
      homepage                   ANY      ANY      ANY    /
      student_list               ANY      ANY      ANY    /student
      student_show               ANY      ANY      ANY    /student/{id}
```

If you have issues of Symfony not finding a new route you've added via a controller annotation comment, try the following.

It's a good idea to **CLEAR THE CACHE** when adding/changing routes, otherwise Symfony may not recognised the new or changed routes ... Either manually delete the `/var/cache` directory, or run the `cache:clear` console command:

```bash
    $ php bin/console cache:clear

    // Clearing the cache for the dev environment with debug true
    [OK] Cache for the "dev" environment (debug=true) was successfully cleared.
```

Symfony caches (stores) routing data and also rendered pages from Twig, to speed up response time. But if you have changed controllers and routes, sometimes you have to manually delete the cache to ensure all new routes are checked against new requests.


## Adding a `find($id)` method to the student repository

Let's add the find-one-by-id method to class `StudentRepository`:

```php
    public function find($id)
    {
        if(array_key_exists($id, $this->students)){
            return $this->students[$id];
        } else {
            return null;
        }
    }
```

If an object can be found with the key of `$id` it will be returned, otherwise `null` will be returned.

NOTE: At this time our code will fail if someone tries to show a student with an Id that is not in our repository array ...

## Make each item in list a link to show

Let's link our templates together, so that we have a clickable link for each student
listed in the list template, that then makes a request to show the details for the student
with that id.

In our list template `/templates/student/index.html.twig` we can get the id for the current student
with `student.id`. The internal name for our show route is `student_show`. We can use the `url(...)` Twig function to generate the URL path for a route, and in this case an `id` parameter.

So we update `list.html.twig` to look as follows, where we add a list `(details)` that will request a student's details to be displayed with our show route:

```twig
    {% extends 'base.html.twig' %}

    {% block body %}
        <h1>Student LIST page</h1>

        <ul>
            {% for student in students %}
                <li>
                    id = {{ student.id }}
                    <br>
                    name = {{ student.firstName }} {{ student.surname }}
                    <br>
                    <a href="{{ url('student_show', {id : student.id} ) }}">(details)</a>
                </li>
            {% endfor %}
        </ul>
    {% endblock %}
```

As we can see, to pass the `student.id` parameter to the `student_show` route we write a call to Twig function `url(...)` in the form:

```twig
    url('student_show', {<name:value-parameter-list>} )
```

We can represent a key-value array in Twig using the braces (curly brackets), and colons. So the PHP associative array (map):

```php
    $daysInMonth = [
        'jan' => 31,
        'feb' => 28
    ];
```

could be represented in Twig as:

```twig
    set daysInMonth = {'jan':31, 'feb':28}
```

Thus we can pass an array of parameter-value pairs to a route in Twig using the brace (curly bracket) syntax, as in:

```twig
    url('student_show', {id : student.id} )
```



Figure \ref{list_with_links} shows a screenshot our list of students web page, with a `(details)` hypertext link to the show page for each individual student obbject.

![Screenshot of student list page, with links to show page for each student object. \label{list_with_links}](./03_figures/lab03_entities/3_list_with_links.png)

## Dealing with not-found issues (project `basic06`)

If we attempted to retrieve a record, but got back `null`, we might cope with it in this way in our controller method, i.e. by throwing a Not-Found-Exception (which generates a 404-page in production):

```php
    if (!$student) {
        throw $this->createNotFoundException(
            'No product found for id '.$id
        );
    }
```

Or we could simply create an error Twig page, and display that to the user, e.g.:

```php
    public function showAction($id)
    {
        $studentRepository = new StudentRepository();
        $student = $studentRepository->find($id);

        $template = 'student/show.html.twig';
        $args = [
            'student' => $student
        ];

        if (!$student) {
            $template = 'error/404.html.twig';
        }

        return $this->render($template, $args);
    }
```

and a Twig template `/templates/error/404.html.twig` looking like this:

```twig
    {% extends 'base.html.twig' %}
    
    {% block title %}ERROR PAGE{% endblock %}

    {% block body %}
        <h1>Whoops! something went wrong</h1>

        <h2>404 - no found error</h2>

        <p>
            sorry - the item/page you were looking for could not be found
        </p>
    {% endblock %}
```

NOTE: We have overriden the `title` Twig block, so that the page title is `ERROR PAGE`...

Figure \ref{error404} shows a screenshot of our custom 404 error template for when no such student can be found for the given ID.

![Error page for non-existant student ID = 66. \label{error404}](./03_figures/part01/5_404error.png)
