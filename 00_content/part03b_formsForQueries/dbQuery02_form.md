# Custom database queries

## Search Form for exact property value (project `query02`)

Searching by having to type values in the URL isn't ideal. So let's add an HTML form in the list of projects page, allowing users to enter the category that way.  Figure \ref{query6}illustrates what we are going to create.

![Form to search for category. \label{query6}](./03_figures/part03/q6_simpleForm.png){width=80%}

## The form in Twig templat

Let's write the HTML code for the submission form for the Products list Twig template in `/templates/product/index.html.twig`.

At present we have a table `<thead>` with a row of column headers, and then a loop for each Product:

```twig    
    {% extends 'base.html.twig' %}
    
    {% block title %}Product index{% endblock %}
    
    {% block body %}
        <h1>Product index</h1>
    
        <table class="table">
            <thead>
                <tr>
                    <th>Id</th>
                    <th>Description</th>
                    <th>Price</th>
                    <th>Category</th>
                    <th>actions</th>
                </tr>
            </thead>
            <tbody>

    <<<< TABLE ROW WITH FORM TO GO HERE >>>>
    
            {% for product in products %}
                <tr>
                    <td>{{ product.id }}</td>    
```

We need to add a new table row between the table headers and the loop of Products:

```twig    
    <tr>
        <th></th>
        <th></th>
        <th></th>
        <th>
            <form action="{{ url('search_category') }}" method="post">
                <input name="category">
                <input type="submit">
            </form>
        </th>
        <th></th>
    </tr>

    {% for product in products %}
        <tr>
            ... as before
```

The row has empty cells, except for the 4th cell (the Category column), where we create a simple form. The form has:

- a method of `post`

- an action of `url('search_category')`

    - we'll have to create this new route in the `ProductController` to process submission of this form
    
- a text box named `category`

    - since this text box will appear in the Categorty column, we don't need to give a text prompt
    
    - the HTML default `<input>` type is `text`, so we don't need to specify this either 
    
- a Submit button

## Controller method to process form submission

Here is the new method in `ProductController` to process submission of this form - implementing the route `search_category`:

```php
    /**
     * @Route("/searchCategory", name="search_category", methods={"POST"})
     */
    public function searchCategory(Request $request): Response
    {
        $category = $request->request->get('category');

        if(empty($category)){
            return $this->redirectToRoute('product_index');
        }

        return $this->redirectToRoute('product_search', ['category' => $category]);
    }
```

The annotation comments specify the URL route `/searchCategory`, the internal route name `search_category`, and that we expect the request to be submitted using the `POST` method:

```php
    /**
     * @Route("/searchCategory", name="search_category", methods={"POST"})
     */
```

We need to extract the `category` variable submitted in the HTTP Request, so we need access to the Symfony `Request` object. The simnplest way to get a reference to this object is via the Symfony **param converter**, by adding `(Request $request)` as a method parameter. This means we now have Request object variable `$request` available to use in our method:

```php
    public function searchCategory(Request $request): Response
```

We can retrieve a value from the submitted `POST` variables int the request using the `get->` method naming the variable `category`. NOTE: In this instance `get` is a **getter** (accessor method) - not to be confused with the HTTP `GET` Request method...

```php
  $category = $request->request->get('category');
```


Finally, we can do some logic based on the value of form submitted variable `$category`. If this variable is an emptuy string, let's just redirect Symfonhy to run the method to list all products, route `product_index`:

```php
    if(empty($category)){
        return $this->redirectToRoute('product_index');
    }
```

If `$category` was **not** empty then we can redirect to our category search route, passing the value to this route:

```php
    return $this->redirectToRoute('product_search', ['category' => $category]);
```

## Getting rid of the URL search route

If we no longer wanted the URL search route, we could replace the final statement in our `searchCategory(...)` method to the following (and remove method `search(...)` altogether):

```php
    /**
     * @Route("/searchCategory", name="search_category", methods={"POST"})
     */
    public function searchCategory(Request $request): Response
    {
        $category = $request->request->get('category');

        if(empty($category)){
            return $this->redirectToRoute('product_index');
        }
    
        // if get here, not empty - so use value to search...
        $productRepository = $this->getDoctrine()->getRepository('App:Product');
        $products =  $productRepository->findByCategory($category);

        $template = 'product/index.html.twig';
        $args = [
            'products' => $products,
            'category' => $category
        ];

        return $this->render($template, $args);
    }
```
