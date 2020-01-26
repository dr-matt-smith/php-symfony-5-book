

# Creating our own classes

## Goals

Our goals are to:

- create a simple Student entity class
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
            $student = new Student(1, 'matt', 'smith');
    
            $template = 'student/show.html.twig';
            $args = [
                'student' => $student
            ];
            return $this->render($template, $args);
        }
    }

```

NOTE:: Ensure your code has the appropriate `use` statement for the `App\Entity\Student` class - a nice IDE like PHPStorm will add this for you...

## The show student template `/templates/student/show.html.twig`

If it does't already exist, create the directory `/templates/student`. If it did exist, you may need to delete a default generated `student/index.html.twig` - we'll create our own Twig templates from scratch.
 
In that directory create a new Twig template named `show.html.twig`. Write the following Twig code for the template:

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

Run the web server and visit `/student`, and you should see our student details displayed as a nice HTML page.


Figure \ref{student_show} shows a screenshot our student details web page.

![Screenshot of student show page. \label{student_show}](./03_figures/lab03_entities/1_show_student.png)

## Creating an Entity Repository (`basic04`)

Let's create a repository class to work with collections of Student objects. So let's create class `StudentRepository` in a new directory `/src/Repository`:

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
            $s1 = new Student($id, 'matt', 'smith');
            $this->students[$id] = $s1;
            $id = 2;
            $s2 = new Student($id, 'joelle', 'murphy');
            $this->students[$id] = $s2;
            $id = 3;
            $s3 = new Student($id, 'frances', 'mcguinness');
            $this->students[$id] = $s3;
        }

        public function findAll()
        {
            return $this->students;
        }
    }
```

## The student list controller method

Now we have a repository that can supply a list of students, let's created a new route `/student/list` that will retrieve the array of student records from an instance of `StudentRepository`, and pass that array to a Twig template, to loop through and display each one. We'll give this route the internal name `student_list` in our annotation comment.

Add method `listAction()` to the controller class `StudentController`:

```php
    use App\Repository\StudentRepository;

    ...

    /**
     * @Route("/student/list", name="student_list")
     */
    public function listAction()
    {
        $studentRepository = new StudentRepository();
        $students = $studentRepository->findAll();

        $template = 'student/list.html.twig';
        $args = [
            'students' => $students
        ];
        return $this->render($template, $args);
    }
```

We should see this new route in our list of routes:

```bash
     -------------------------- -------- -------- ------ -----------------------------------
      Name                       Method   Scheme   Host   Path
     -------------------------- -------- -------- ------ -----------------------------------
      homepage                   ANY      ANY      ANY    /
      student_show               ANY      ANY      ANY    /student
      student_list               ANY      ANY      ANY    /student/list
      ... and the debug / profile routes ...
```

## The list student template `/templates/student/list.html.twig`

In  directory `/templates/student` create a new Twig template named `list.html.twig`. Write the following Twig code for the template:

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

Run the web server and visit `/student/list`, and you should see a list of all student details displayed as a nice HTML page.


Figure \ref{student_list} shows a screenshot our list of students web page.

![Screenshot of student list page. \label{student_list}](./03_figures/lab03_entities/2_list_student.png)

## Refactor show action to show details of one Student object  (project `basic05`)

The usual convention for CRUD is that the **show** action will display the details of an object given its `id`. So let's refactor our method `showAction()` to do this, and also we'll need to add a `findOne(...)` method to our repository class, that returns an object given an id.

The route we'll design will be in the form `/student/{id}`, where `{id}` will be the integer `id` of the object in the repository we wish to display. And, conincidentally, this is just the correct syntax for routes with parameters that we write in the annotation comments to define routes for controller methods in Symfony ...

```php
    /**
     * @Route("/student/{id}", name="student_show")
     */
    public function showAction($id)
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
        public function listAction()
        {
            ... as before
        }
```

We can check these routes via the console:

- `/student/{id}` will invoke our `showAction($id)` method
- `/student` will invoke our `listAction()` method

```bash
     -------------------------- -------- -------- ------ -----------------------------------
      Name                       Method   Scheme   Host   Path
     -------------------------- -------- -------- ------ -----------------------------------
      homepage                   ANY      ANY      ANY    /
      student_show               ANY      ANY      ANY    /student/{id}
      student_list               ANY      ANY      ANY    /student
```

If you have issues of Symfony not finding a new route you've added via a controller annotation comment, try the following.

It's a good idea to **CLEAR THE CACHE** when addeding/changing routes, otherwise Symfony may not recognised the new or changed routes ... Either manually delete the `/var/cache` directory, or run the `cache:clear` console command:

```bash
    $ php bin/console cache:clear

    // Clearing the cache for the dev environment with debug true
    [OK] Cache for the "dev" environment (debug=true) was successfully cleared.
```

Symfony caches (stores) routing data and also rendered pages from Twig, to speed up response time. But if you have changed controllers and routes, sometimes you have to manually delete the cache to ensure all new routes are checked against new requests.

## Make each item in list a link to show

Let's link our templates together, so that we have a clickable link for each student
listed in the list template, that then makes a request to show the details for the student
with that id.

In our list template `/templates/student/list.html.twig` we can get the id for the current student
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

    {% block body %}
        <h1>Whoops! something went wrong</h1>

        <h2>404 - no found error</h2>

        <p>
            sorry - the item/page you were looking for could not be found
        </p>
    {% endblock %}
```


