# Models, Templates and Views {#chapter-mtv}
Now that we have the models set up and populated the database with some sample data, we can now start connecting the models, views and templates to serve up dynamic content. In this chapter, we will go through the process of showing categories on the main page, and then create dedicated category pages which will show the associated list of links.

## Workflow: A Data-Driven Page 
To do this, there are five main steps that you must undertake to create a data-driven webpage in Django.

1.  In the `views.py` module, `import` the models you wish to use.
2.  In the view function, query the model to get the data you want to present.
3.  Also in the view, pass the results from your model into the template's context.
4.  Create/modify the template so that it displays the data from the context.
5.  If you have not done so already, map a URL to your view.

These steps highlight how we need to work within Django's framework to bind models, views and templates together.

## Showing Categories on Rango's Homepage
One of the requirements regarding the main page was to show the top five categories present within your app's database. To fulfil this requirement, we will go through each of the above steps.

### Importing Required Models
First, we need to complete step one. Open `rango/views.py` and at the top of the file, after the other `import`s, `import` the `Category` model from Rango's `models.py` file.

{lang="python",linenos=off}
	# Import the Category model
	from rango.models import Category

### Modifying the Index View
Here we will complete steps two and step three, where we need to modify the view `index()` function. Remember that the `index()` function is responsible for the main page view. Modify `index()` as follows:

{lang="python",linenos=off}
	def index(request):
	    # Query the database for a list of ALL categories currently stored.
	    # Order the categories by the number of likes in descending order.
	    # Retrieve the top 5 only -- or all if less than 5.
	    # Place the list in our context_dict dictionary (with our boldmessage!)
	    # that will be passed to the template engine.
	    category_list = Category.objects.order_by('-likes')[:5]
	    
	    context_dict = {}
	    context_dict['boldmessage'] = 'Crunchy, creamy, cookie, candy, cupcake!'
	    context_dict['categories'] = category_list
	    
	    # Render the response and send it back!
	    return render(request, 'rango/index.html', context_dict)

Here, the expression `Category.objects.order_by('-likes')[:5]` queries the `Category` model to retrieve the top five categories. You can see that it uses the `order_by()` method to sort by the number of `likes` in descending order. The `-` in `-likes` denotes that we would like them in *descending* order (if we removed the `-` then the results would be returned in *ascending* order). Since a list of `Category` objects will be returned, we used Python's [list operators](https://www.quackit.com/python/reference/python_3_list_methods.cfm) to take the first five objects from the list (`[:5]`) to return a subset of `Category` objects.

With the query complete, we passed a reference to the list (stored as variable `category_list`) to the dictionary, `context_dict`. This dictionary is then passed as part of the context for the template engine in the `render()` call. Note that above, we still include our `boldmessage` in the `context_dict` -- this is still required for the existing template to work! This means our context dictionary now contains two key/value pairs: `boldmessage`, representing our `Crunchy, creamy, cookie, candy, cupcake!` message, and `categories`, representing our top five categories that have been extracted from the database.

W> ###Warning
W> For this to work, you will have had to complete the exercises in the previous chapter where you need to add the field `likes` to the `Category` model. Like we have said already, we assume you complete all exercises as you progress through this book.

### Modifying the Index Template
With the view updated, we can complete the fourth step and update the template `rango/index.html`, located within your project's `templates` directory. Change the HTML and Django template code so that it looks like the example shown below. Note that the major changes start at line 15.

{lang="html",linenos=on}
	<!DOCTYPE html>
	{% load staticfiles %}
	<html>
	<head>
	    <title>Rango</title>
	</head>
	
	<body>
	    <h1>Rango says...</h1>
	    <div>
	        hey there partner! <br/>
	        <strong>{{ boldmessage }}</strong>
	    </div>
	
	    <div>
	    {% if categories %}
	        <ul>
	        {% for category in categories %}
	            <li>{{ category.name }}</li>
	        {% endfor %}
	        </ul>
	    {% else %}
	        <strong>There are no categories present.</strong>
	    {% endif %}	
	    </div>
	    
	    <div>
	        <a href="/rango/about/">About Rango</a><br />
	        <img src="{% static "images/rango.jpg" %}" alt="Picture of Rango" /> 
	    </div>
	</body>
	</html>

Here, we make use of Django's template language to present the data using `if` and `for` control statements. Within the `<body>` of the page, we test to see if `categories` -- the name of the context variable containing our list of (a maximum of) five categories -- actually contains any categories (`{% if categories %}`).

If so, we proceed to construct an unordered HTML list (within the `<ul>` tags). The `for` loop (`{% for category in categories %}`) then iterates through the list of results, and outputs each category's name (`{{ category.name }})` within an `<li>` element to denote that it is a *list element.*

If no categories exist, a message is displayed instead indicating that no categories are present -- `There are no categories present.`. As this is wrapped in a `<strong>` element, it will appear in **bold**.

As the example also demonstrates Django's template language, all template commands are enclosed within the tags `{%` and `%}`, while variables whose values are to be placed on the page are referenced within `{{` and `}}` brackets. *Everything* within these tags and brackets is interpreted by the Django templating engine before sending a completed response back to the client.

Now, save the template file and head over to your web browser. Refresh Rango's homepage at `http://127.0.0.1:8000/rango/`, and you show then see a list of categories underneath the page title and your bold message, just like in the [figure below](#fig-ch6-rango-categories-index). Well done, this is your first *data-driven webpage!*

{id="fig-ch6-rango-categories-index"}
![The Rango homepage -- now dynamically generated -- showing a list of categories.](images/ch6-rango-categories-index.png)

##Creating a Details Page
According to the [specifications for Rango](#overview-design-brief-label), we also need to show a list of pages that are associated with each category. To accomplish this, there are several different challenges that we need to overcome. First, a new view must be created. *This new view will have to be parameterised.* We also need to create URL patterns and URL strings that encode category names.

### URL Design and Mapping
Let's start by considering the URL problem. One way we could handle this problem is to use the unique ID for each category within the URL. For example, we could create URLs like `/rango/category/1/` or `/rango/category/2/`, where the numbers correspond to the categories with unique IDs 1 and 2 respectively. However, it is not possible to infer what the category is about just from the ID.

Instead, we could use the category name as part of the URL. For example, we can imagine that the URL `/rango/category/python/` would lead us to a list of pages related to Python. This is a simple, readable and meaningful URL. If we go with this approach, we'll also have to handle categories that have multiple words, like `Other Frameworks`, for example. **Putting spaces in URLs is generally regarded as bad practice** (as we describe below). Spaces need to be [*percent-encoded*](https://www.w3schools.com/tags/ref_urlencode.asp). For example, the percent-encoded string for `Other Frameworks` would read `Other%20Frameworks`. This looks messy and confusing!

T> ### Clean your URLs
T> Designing clean and readable URLs is an important aspect of web design. See [Wikipedia's article on Clean URLs](http://en.wikipedia.org/wiki/Clean_URL) for more details.

To solve this problem, we will make use of the so-called `slugify()` function provided by Django.

### Update the Category Table with a Slug Field
To make readable URLs, we need to include a [slug](https://prettylinks.com/2018/03/url-slugs/) field in the `Category` model. First, we need to import the function `slugify` from Django that will replace whitespace with hyphens, circumnavigating the percent-encoded problem. For example, `"how do i create a slug in django"` turns into `"how-do-i-create-a-slug-in-django"`.

W> ### Unsafe URLs
W> While you can use spaces in URLs, it is considered to be unsafe to use them. Check out the [Internet Engineering Task Force Memo on URLs](http://www.ietf.org/rfc/rfc1738.txt) to read more.

Next, we need to override the `save()` method of the `Category` model. This overridden function will call the `slugify()` function and update the new `slug` field with it. Note that every time the category name changes, the slug will also change -- the `save()` method is always called when creating or updating an instance of a Django model. Update your `Category` model as shown below.

{lang="python",linenos=off}
	class Category(models.Model):
	    name = models.CharField(max_length=128, unique=True)
	    views = models.IntegerField(default=0)
	    likes = models.IntegerField(default=0)
	    slug = models.SlugField()
	    
	    def save(self, *args, **kwargs):
	        self.slug = slugify(self.name)
	        super(Category, self).save(*args, **kwargs)
	        
	    class Meta:
	        verbose_name_plural = 'categories'
	    
	    def __str__(self):
	        return self.name

Don't forget to add in the following `import` at the top of the module, either.

{lang="python",linenos=off}
	from django.template.defaultfilters import slugify

The overriden `save()` method is relatively straightforward to understand. When called, the `slug` field is set by using the output of the `slugify()` function as the new field's value. Once set, the overriden `save()` method then calls the parent (or `super`) `save()` method defined in the base `django.db.models.Model` class. It is this call that performs the necessary logic to take your changes and save the said changes to the correct database table.

Now that the model has been updated, the changes must now be propagated to the database. However, since data already exists within the database from previous chapters, we need to consider the implications of the change. Essentially, for all the existing category names, we want to turn them into slugs (which is performed when the record is initially saved). When we update the models via the migration tool, it will add the `slug` field and provide the option of populating the field with a default value. Of course, we want a specific value for each entry -- so we will first need to perform the migration, and then re-run the population script. This is because the population script will explicitly call the `save()` method on each entry, triggering the `save()` as implemented above, and thus update the slug accordingly for each entry.

To perform the migration, issue the following commands (as detailed in the [Models and Databases Workflow](#section-models-databases-workflow)).

{lang="text",linenos=off}
	$ python manage.py makemigrations rango

Since we did not provide a default value for the slug and we already have existing data in the model, the `makemigrations` command will give you two options. Select the option to provide a default (option `1`), and enter an empty string -- denoted by two quote marks (i.e. `''`).

You should then see output that confirms that the migrations have been created.

{lang="text",linenos=off}
	You are trying to add a non-nullable field 'slug' to category without a default;
	    we can't do that (the database needs something to populate existing rows).
	Please select a fix:
	 1) Provide a one-off default now (will be set on all existing rows with a null
	        value for this column)
	 2) Quit, and let me add a default in models.py
	Select an option: 1
	Please enter the default value now, as valid Python
	The datetime and django.utils.timezone modules are available, so you can do
	    e.g. timezone.now
	Type 'exit' to exit this prompt
	>>> ''
	Migrations for 'rango':
	  rango/migrations/0003_category_slug.py
	    - Add field slug to category

From there, you can then migrate the changes, and run the population script again to update the new slug fields.

{lang="text",linenos=off}
	$ python manage.py migrate
	$ python populate_rango.py

Now run the development server with the command `$ python manage.py runserver`, and inspect the data in the models with the admin interface. Remember that the admin interface is reached by pointing your browser to `http://127.0.0.1:8000/admin/`.

If you go to add in a new category via the admin interface you may encounter a problem -- or two!

1. Let's say we added in the category `Python User Groups`. If you try to save the record, Django will not let you save it unless you also fill in the slug field too. While we could type in `python-user-groups`, relying on users to fill out fields will always be error-prone, and wouldn't provide a good user experience. It would be better to have the slug automatically generated.
2. The next problem arises if we have one category called `Django` and one called `django`. Since the `slugify()` makes the slugs lower case it will not be possible to identify which category corresponds to the `django` slug.

To solve the first problem, there are two possible solutions. The easiest solution would be to update our model so that the slug field allows blank entries, i.e.: 

{lang="python",linenos=off}
	slug = models.SlugField(blank=True)

However, this is less than desirable. A blank slug would probably not make any sense, and at that, you could define multiple blank slugs! How could we then identify what category a user is referring to? **A much better solution** would be to customise the admin interface so that it automatically pre-populates the slug field as you type in the category name. To do this, update `rango/admin.py` with the following code.

{lang="python",linenos=off}
	from django.contrib import admin
	from rango.models import Category, Page
	...
	# Add in this class to customise the Admin Interface
	class CategoryAdmin(admin.ModelAdmin):
	    prepopulated_fields = {'slug':('name',)}
	
	# Update the registration to include this customised interface
	admin.site.register(Category, CategoryAdmin)
	...

Note that with the above `admin.site.register()` call, you should ensure that you replace the existing call to `admin.site.register(Category)` with the one provided above. Once completed, try out the admin interface and add in a new category. Note that the `slug` field automatically populates as you type into the `name` field. Try adding a space and see what happens!

Now that we have addressed the first problem, we can ensure that the slug field is also unique by adding the constraint to the slug field. Update your `models.py` module to reflect this change with the inclusion of `unique=True`.

{lang="python",linenos=off}
	slug = models.SlugField(unique=True)

Now that we have added in the slug field, we can now use the slugs to guarantee a unique match to an associated category. We could have added the unique constraint earlier. However, if we had done that and then performed the migration (and thus setting everything to be an empty string by default), the migration would have failed. This is because the unique constraint would have been violated. We could have deleted the database and then recreated everything -- but that is not always desirable.

W> ### Migration Woes
W> It's always best to plan out your database in advance and avoid changing it. Making a population script means that you easily recreate your database if you find that you need to delete it and start again.
W>
W> If you find yourself completely stuck, it is sometimes just more straightforward to delete the database and recreate everything from scratch, rather than trying to work out where the issue is coming from.

### Category Page Workflow
Now we need to figure out how to create a page for individual categories. This will allow us to then be able to see the pages associated with a category. To implement the category pages so that they can be accessed using the URL pattern `/rango/category/<category-name-slug>/`, we need to undertake the following steps.

1. Import the `Page` model into `rango/views.py`.
2. Create a new view in `rango/views.py` called `show_category()`. The `show_category()` view will take an additional parameter, `category_name_slug`, which will store the encoded category name.
	- We will need helper functions to encode and decode `category_name_slug`.
3.  Create a new template, `templates/rango/category.html`.
4.  Update Rango's `urlpatterns` to map the new `category` view to a URL pattern in `rango/urls.py`.

We'll also need to update the `index()` view and `index.html` template to provide links to the category page view.

### Category View
In `rango/views.py`, we first need to import the `Page` model. This means we must add the following import statement at the top of the file.

{lang="python",linenos=off}
	from rango.models import Page

Next, we can add our new view, `show_category()`.

{lang="python",linenos=off}
	def show_category(request, category_name_slug):
	    # Create a context dictionary which we can pass 
	    # to the template rendering engine.
	    context_dict = {}
	    
	    try:
	        # Can we find a category name slug with the given name?
	        # If we can't, the .get() method raises a DoesNotExist exception.
	        # The .get() method returns one model instance or raises an exception.
	        category = Category.objects.get(slug=category_name_slug)
	        
	        # Retrieve all of the associated pages.
	        # The filter() will return a list of page objects or an empty list.
	        pages = Page.objects.filter(category=category)
	        
	        # Adds our results list to the template context under name pages.
	        context_dict['pages'] = pages
	        # We also add the category object from 
	        # the database to the context dictionary.
	        # We'll use this in the template to verify that the category exists.
	        context_dict['category'] = category
	    except Category.DoesNotExist:
	        # We get here if we didn't find the specified category.
	        # Don't do anything - 
	        # the template will display the "no category" message for us.
	        context_dict['category'] = None
	        context_dict['pages'] = None
	    
	    # Go render the response and return it to the client.
	    return render(request, 'rango/category.html', context_dict)

Our new view follows the same basic steps as our `index()` view. We first define a context dictionary. Then, we attempt to extract the data from the models and add the relevant data to the context dictionary. We determine which category has been requested by using the value passed `category_name_slug` to the `show_category()` view function (in addition to the `request` parameter).

If the category slug is found in the `Category` model, we can then pull out the associated pages, and add this to the context dictionary, `context_dict`. If the category requested was not found, we set the associated context dictionary values to `None`. Finally, we `render()` everything together, using a new `category.html` template.

### Category Template
In your `<workspace>/tango_with_django_project/templates/rango/` directory, create a new template called `category.html`. In the new file, add the following code.

{lang="html",linenos=on}
	<!DOCTYPE html>
	<html>
	    <head>
	        <title>Rango</title>
	    </head>
	
	    <body>
	        <div>
	        {% if category %}
	            <h1>{{ category.name }}</h1>
	            {% if pages %}
	            <ul>
	                {% for page in pages %}
	                <li><a href="{{ page.url }}">{{ page.title }}</a></li>
	                {% endfor %}
	            </ul>
	            {% else %}
	            <strong>No pages currently in category.</strong>
	            {% endif %}
	        {% else %}
	            The specified category does not exist!
	        {% endif %}
	        </div>
	    </body>
	</html>

The HTML code example again demonstrates how we utilise the data passed to the template via its context through the tags `{{ }}`. We access the `category` and `pages` objects and their fields -- such as `category.name` and `page.url`, for example.

If the `category` exists, then we check to see if there are any pages in the category. If so, we iterate through the returned pages using the `{% for page in pages %}` template tags. For each `page` in the `pages` list, we present their `title` and `url` attributes as listed hyperlink (e.g. within `<li>` and `<a>` elements). These are displayed in an unordered HTML list (denoted by the `<ul>` tags). If you are not too familiar with HTML, then have a look at the [HTML Tutorial by W3Schools.com](http://www.w3schools.com/html/) to learn more about the different tags.

I> ### Note on Conditional Template Tags
I> The Django template conditional tag -- represented with `{% if condition %}` -- is a really neat way of determining the existence of an object within the template's context. Make sure you check the existence of an object to avoid errors when rendering your templates, especially if your associated view's logic doesn't populate the context dictionary in all possible scenarios.
I>
I> Placing conditional checks in your templates -- like `{% if category %}` in the example above -- also makes sense semantically. The outcome of the conditional check directly affects how the rendered page is presented to the user. Remember, presentational aspects of your Django apps should be encapsulated within templates.

### Parameterised URL Mapping
Now let's have a look at how we pass the value of the `category_name_url` parameter to our function `show_category()` function. To do so, we need to modify Rango's `urls.py` file and update the `urlpatterns` tuple as follows.

{lang="python",linenos=off}
	urlpatterns = [
	    path('', views.index, name='index'),
	    path('about/', views.about, name='about'),
	    path('category/<slug:category_name_slug>/', 
	         views.show_category, name='show_category'),
	]

A parameter, represented by `<slug:category_name_slug>`, is added to a new mapping. This indicates to Django that we want to match a string which is a `slug`, and to assign it to variable `category_name_slug`. You will notice that this variable name is what we pass through to the view `show_category()`. If these two names do not match exactly, Django will get confused and raise an error. Instead of slugs, you can also extract out other variables like strings and integers. Refer to the [Django documentation on URL paths](https://docs.djangoproject.com/en/2.1/ref/urls/) for more details. If you need to parse more complicated expressions, you can use `re_path()` instead of `path()`. This will allow you to match all sorts of regular (and irregular) expressions. Luckily for us, Django provides matches for the most common patterns.

<!-->
We have added in a rather complex entry that will invoke `view.show_category()` when the URL pattern `r'^category/(?P<category_name_slug>[\w\-]+)/$'` is matched. 

There are two things to note here. First, we have added a parameter name with in the URL pattern, i.e. `<category_name_slug>`, which we will be able to access in our view later on. When you create a parameterised URL you need to ensure that the parameters that you include in the URL are declared in the corresponding view.
The next thing to note is that the regular expression `[\w\-]+)` will look for any sequence of alphanumeric characters e.g. `a-z`, `A-Z`, or `0-9` denoted by `\w` and any hyphens (-) denoted by `\-`, and we can match as many of these as we like denoted by the `[ ]+` expression.

The URL pattern will match a sequence of alphanumeric characters and hyphens which are between the `rango/category/` and the trailing `/`. This sequence will be stored in the parameter `category_name_slug` and passed to `views.show_category()`. For example, the URL `rango/category/python-books/` would result in the `category_name_slug` having the value, `python-books`. However, if the URL was `rango/category/££££-$$$$$/` then the sequence of characters between `rango/category/` and the trailing `/` would not match the regular expression, and a `404 not found` error would result because there would be no matching URL pattern.
-->

All view functions defined as part of a Django app *must* take at least one parameter. This is typically called `request` -- and provides access to information related to the given HTTP request made by the user. When parameterising URLs, you supply additional named parameters to the signature for the given view. That's why `show_category()` was defined as `def show_category(request, category_name_slug)`.

<!--
It's not the position of the additional parameters that matters, it's
the *name* that must match anything defined within the URL pattern.
 Note how `category_name_slug` defined in the URL pattern matches the
 `category_name_slug` parameter defined for our view. 
		Using  `category_name_slug` in our view will give `python-books`, or whatever value was supplied as that part of the URL.
-->
		
I> ###Regex Hell
I> "Some people, when confronted with a problem, think *'I know, I'll use regular expressions.'* Now they have two problems."
I> [Jamie Zawinski](http://blog.codinghorror.com/regular-expressions-now-you-have-two-problems/)
I>
I> Django's `path()` method means you can generally avoid Regex Hell -- but if you need to use a regular expression (with the `re_path()` function, for instance), this [cheat sheet](http://cheatography.com/davechild/cheat-sheets/regular-expressions/) is really useful.

### Modifying the Index Template
Our new view is set up and ready to go -- but we need to do one more thing. Our index page template needs to be updated so that it links to the category pages that are listed. We can update the `index.html` template to now include a link to the category page via the slug.

{lang="html",linenos=on}
	<!DOCTYPE html>
	{% load staticfiles %}
	<html>
	    <head>
	        <title>Rango</title>
	    </head>
	    
	    <body>
			hey there partner! <br />
			<strong>{{ boldmessage }}</strong>
	        
	        <div>
	            hey there partner!
	        </div>
	        
	        <div>
	        {% if categories %}
	        <ul>
	            {% for category in categories %}
	            <!-- The following line is changed to add an HTML hyperlink -->
	            <li>
	            <a href="/rango/category/{{ category.slug }}/">{{ category.name }}</a>
	            </li>
	            {% endfor %}
	        </ul>
	        {% else %}
	            <strong>There are no categories present.</strong>
	        {% endif %}
	        </div>
	        
	        <div>
	            <a href="/rango/about/">About Rango</a><br />
	            <img src="{% static "images/rango.jpg" %}" alt="Picture of Rango" /> 
	        </div>
	    </body>
	</html>

Again, we used the HTML tag `<ul>` to define an unordered list. Within the list, we create a series of list elements (`<li>`), each of which in turn contains an HTML hyperlink (`<a>`). The hyperlink has an `href` attribute, which we use to specify the target URL defined by `/rango/category/{{ category.slug }}` which, for example, would turn into `/rango/category/other-frameworks/` for the category `Other Frameworks`.

### Demo
Let's try everything out now by visiting the Rango homepage. You should see *up to* five categories on the index page. The categories should now be links. Clicking on `Django` should then take you to the `Django` category page, as shown in the [figure below](#fig-ch6-rango-links). If you see a list of links like `Official Django Tutorial`, then you've successfully set up the new page. 

What happens when you visit a category that does not exist? Try navigating a category which doesn't exist, like `/rango/category/computers/`. Do this by typing the address manually into your browser's address bar. You should see a message telling you that the specified category does not exist. Look at your template's logic and work through it to understand what is going on.

{id="fig-ch6-rango-links"}
![The links to Django pages. Note the mouse is hovering over the first link -- you can see the corresponding URL for that link at the bottom left of the Google Chrome window.](images/ch6-rango-links.png)

X> ## Exercises
X> Reinforce what you've learnt in this chapter!
X> 
X> * Update the population script to add some value to the `views` count for each **page**. Pick whatever integers you want -- as long as each page receives a whole (integer) number greater than zero.
X> * Modify the index page to also include the top five most viewed pages. When no pages are present, you should include a friendly message in place of a list, stating: `There are no pages present.` This message should be bolded -- wrap it around `<strong>...</strong>` tags.
X> * Leading on from the exercise above, include a heading for the `Most Liked Categories` and `Most Viewed Pages`. These must be placed as second-level headers, using the `<h2>` tag. Place each of the headers above their respective list.
X> * Include a link back to the index page from the category page.
X> * Undertake [part three of official Django tutorial](https://docs.djangoproject.com/en/2.1/intro/tutorial03/) if you have not done so already to reinforce what you've learnt here.

{id="fig-ch6-exercises"}
![The index page after you complete the exercises. Your output may vary slightly.](images/ch6-exercises.png)

T> ### Hints
T> * When updating the population script, you'll essentially follow the same process as you went through in the [previous chapter's](#chapter-models-databases) exercises. You will need to update the data structures for each page, and also update the code that makes use of them.
T>      * Update the `python_pages`, `django_pages` and `other_pages` data structures. Each page has a `title` and `url` -- they all now need a count of how many `views` they see, too.
T>      * Look at how the `add_page()` function is defined in your population script. Does it allow for you to pass in a `views` count? Do you need to change anything in this function?
T>      * Finally, update the line where the `add_page()` function is *called*. If you called the views count in the data structures `views`, and the dictionary that represents a page is called `p` in the context of where `add_page()` is called, how can you pass the views count into the function?
T> * Remember to re-run the population script so that the database is updated with your new counts.
T>      * You will need to edit both the `index` view and the `index.html` template to put the most viewed (i.e. popular pages) on the index page.
T>      * Instead of accessing the `Category` model, you will have to ask the `Page` model for the most viewed pages.
T>      * Remember to pass the list of pages through to the context.
T>      * If you are not sure about the HTML template code to use, you can draw inspiration from the `category.html` template markup. The markup that you need to write is essentially the same.

T> ### Model Tips
T> For more tips on working with models you can take a look through the following blog posts:
T> 
T> 1. [Best Practices when working with models](http://steelkiwi.com/blog/best-practices-working-django-models-python/) by Kostantin Moiseenko. In this post, you will find a series of tips and tricks when working with models.
T> 2. [How to make your Django Models DRYer](https://medium.com/@raiderrobert/make-your-django-models-dryer-4b8d0f3453dd#.ozrdt3rsm) by Robert Roskam. In this post, you can see how you can use the `property` method of a class to reduce the amount of code needed when accessing related models.

X> ### Test your Implementation
X> Like in the previous chapter, we've implemented a series of unit tests to allow you to check your implementation up until this point. [Follow the guide we provided earlier](#section-getting-ready-tests), using the test module `tests_chapter6.py`. How does your implementation stack up against our tests? Remember that your implementation should have fully completed the exercises listed above for the tests to pass.
X>
X> Some of these tests may seem overly harsh -- but remember, this book is a specification for the product we want you to develop. If you don't develop software *exactly* as specified, it can produce undesirable results when your bit of code is plugged into a larger framework. By following specifications to the letter, you'll be learning a valuable lesson that you can take forward in a future development career.