# Bootstrapping Rango {#chapter-bootstrap}
In this chapter, we will be styling Rango using the *Twitter Bootstrap 4* toolkit. Bootstrap is the most popular HTML, CSS, JS Framework, which we can use to style our application. The toolkit lets you design and style responsive web applications, and is pretty easy to use once you get familiar with it.

I> ### Cascading Style Sheets
I> If you are not familiar with CSS, have a look at the [CSS crash course](#chapter-css). We provide a quick guide on the basic of Cascading Style Sheets.

Now take a look at the [Bootstrap 4.0 website](http://getbootstrap.com/) - it provides you with sample code and examples of the different components and how to style them by added in the appropriate style tags, etc. On the Bootstrap website they provide a number of [example layouts](http://getbootstrap.com/examples/) which we can base our design on.

To style Rango we have identified that the [dashboard style](http://getbootstrap.com/examples/dashboard/) more or less meets our needs in terms of the layout of Rango, i.e. it has a menu bar at the top, a side bar (which we will use to show categories) and a main content pane. 

Download and save the HTML source for the Dashboard layout to a file called, `bootstrap-base.html` and save it to your `templates/rango` directory.

Before we can use the template, we need to modify the HTML so that we can use it in our application. The changes that we performed are listed below along with the updated HTML (so that you don't have to go to the trouble and a copy of the HTML is in our [github repository] 
(https://raw.githubusercontent.com/leifos/tango_with_django_2/master/code/tango_with_django_project/templates/rango/bootstrap-base.html).

- Replaced all references of `../../` to be `http://getbootstrap.com/docs/4.2/` as we are using version 4.2.
- Replaced `dashboard.css` with the absolute reference:
	- `http://getbootstrap.com/docs/4.2/examples/dashboard/dashboard.css`
- Removed the search form from the top navigation bar.
- Stripped out all the non-essential content from the HTML and replaced it with:
	- `{% block body_block %}{% endblock %}`
- Set the title element to be:
	- `<title>
           Rango - {% block title %}How to Tango with Django!{% endblock %}
       </title>`
- Changed `project name` to be `Rango`.
- Added the links to the index page, login, register, etc to the top nav bar.
- Added in a side block, i.e., `{% block side_block %}{% endblock %}`
- Added in `{% load staticfiles %}`  and `{% load rango_template_tags %}` after the `DOCTYPE` tag.



## Template

W> ### Copying and Pasting
W> In the introductory chapter, we said not the copy and paste - but this is an exception. 
W> However, if you directly cut and paste you will end up bringing additional text you do not want. So go to our github page and get the [base template] 
(https://raw.githubusercontent.com/leifos/tango_with_django_2/master/code/tango_with_django_project/templates/rango/bootstrap-base.html) shown below.
W> 
W> Also, If you don't understand what the specific Bootstrap classes do, then you can check out the [ Bootstrap Documentation](http://getbootstrap.com/css/).


{lang="html",linenos=off}
	<!DOCTYPE html>
	{% load staticfiles %}
	{% load rango_template_tags %}
	<html lang="en">
	  <head>
	    <meta charset="utf-8">
	    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
	    <meta name="description" content="">
	    <meta name="author" content="Mark Otto, Jacob Thornton, and Bootstrap contributors">
	    <meta name="generator" content="Jekyll v3.8.5">
	    <link rel="icon" href="{% static 'images/favicon.ico' %}">
	    <title>
	      Rango - {% block title %}How to Tango with Django!{% endblock %}
	    </title>

	    <!-- Bootstrap core CSS -->
		<!-- Bootstrap core CSS -->
		<link href="https://getbootstrap.com/docs/4.2/dist/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-GJzZqFGwb1QTTN6wy59ffF1BuGJpLSa9DkKMp0DgiMDm4iYMj70gZWKYbI706tWS" crossorigin="anonymous">
	
	    <!-- Custom styles for this template -->
	    <link href="https://getbootstrap.com/docs/4.2/examples/dashboard/dashboard.css" rel="stylesheet">

	  </head>

	  <body>
		  <header>
	    <nav class="navbar navbar-expand-md navbar-dark fixed-top bg-dark p-0">
	  <a class="navbar-brand p-2" href="{% url 'rango:index' %}">Rango</a>
	  <button class="navbar-toggler" type="button" data-toggle="collapse" data-target="#navbarCollapse" aria-controls="navbarCollapse" aria-expanded="false" aria-label="Toggle navigation">
	        <span class="navbar-toggler-icon"></span>
	      </button>
  
	<div class="collapse navbar-collapse" id="navbarCollapse">
	  <ul class="navbar-nav mr-auto">
	    <li class="nav-item"><a class="nav-link" href="{% url 'rango:index' %}">Home</a></li>
	    <li class="nav-item "><a class="nav-link" href="{% url 'rango:about' %}">About</a></li>
		{% if user.is_authenticated %}
	    <li class="nav-item "><a class="nav-link" href="{% url 'rango:restricted' %}">Restricted</a></li>
	    <li class="nav-item"><a class="nav-link" href="{% url 'rango:add_category' %}">Add Category</a></li>
	    <li class="nav-item"><a class="nav-link" href="{% url 'auth_logout' %}?next=/rango/">Logout</a></li>
		{% else %}
	    <li class="nav-item"><a class="nav-link" href="{% url 'registration_register' %}">Register Here</a></li>
	    <li class="nav-item "><a class="nav-link" href="{% url 'auth_login' %}">Login</a></li>
		{% endif %}
	
	  </ul>
	</div>
	</nav>

	</header>


	    <div class="container-fluid">
	      <div class="row">
			  <nav class="col-md-2 d-none d-md-block bg-light sidebar">
			       <div class="sidebar-sticky">
		
		        {% block sidebar_block %}
		            {% get_category_list category %}
		        {% endblock %}
			
			 </div>
		 </nav>
		 
	 <main role="main" class="col-md-9  ml-sm-auto col-lg-10 px-4">		
		 {% block body_block %}{% endblock %}
	 
	  <!-- FOOTER -->
	  <footer>
	    <p class="float-right"><a href="#">Back to top</a></p>
	    <p>&copy; 2019 Tango With Django 2 &middot; <a href="#">Privacy</a> &middot; <a href="#">Terms</a></p>
	  </footer>
	</main>


	    <!-- Bootstrap core JavaScript -->
	    <!-- Placed at the end of the document so the pages load faster -->
	 <script src="https://code.jquery.com/jquery-3.3.1.slim.min.js" integrity="sha384-q8i/X+965DzO0rT7abK41JStQIAqVgRVzpbzo5smXKp4YfRvH+8abtTE1Pi6jizo" crossorigin="anonymous"></script>
	       <script>window.jQuery || document.write('<script src="https://getbootstrap.com/docs/4.2/assets/js/vendor/jquery-slim.min.js"><\/script>')</script><script src="https://getbootstrap.com/docs/4.2/dist/js/bootstrap.bundle.min.js" integrity="sha384-zDnhMsjVZfS3hiP7oCBRmfjkQC4fzxVxFhBx8Hkz2aZX8gEvA/jsP3eXRCvzTofP" crossorigin="anonymous"></script>
	         <script src="https://cdnjs.cloudflare.com/ajax/libs/feather-icons/4.9.0/feather.min.js"></script>
        
	         <script src="https://getbootstrap.com/docs/4.2/examples/dashboard/dashboard.js"></script>
		</body>
	</html>

	
	

Once you have the new template setup, you can download the [Rango Favicon](https://raw.githubusercontent.com/leifos/tango_with_django_2/master/code/tango_with_django_project/static/images/favicon.ico) and save it to `static/images/`.

If you take a close look at the modified Dashboard HTML source, you'll notice it has a lot of structure in it created by a series of `<div>` tags. Essentially the page is broken into two parts - the top navigation bar which is contained by `<header>` tags, and the main content pane denoted by the `<main ... >` tag. Within the main content pane, there is a `<div>` which houses two other divs for the `sidebar_block` and the `body block`.
	
	
The code above assumes that you have completed the chapters on User Authentication and used the Django Regisration Redux Package. If not you will need to update the template and remove/modify the references to those links in the navigation bar in the header. 
	
Also of note is that the HTML template makes references to external websites to request the required `css` and `js` files. So you will need to be connected to the internet for the style to be loaded when you run the application.

I> ###Working Offline?
I> Rather than including external references to the `css` and `js` files, you could download all the associated files and store them in your static directory. If you do this, simply update the base template to reference the static files stored locally. 

	
## Quick Style Change
To give Rango a much needed facelift, we can replace the content of the existing `base.html` with the HTML template code in `base_bootstrap.html`. You might want to first copy out the existing code in `base.html` and then copy in the `base_bootstrap.html` code.

Now reload your application. Pretty nice! Well, sort of.

You should notice that your application looks about a hundred times better already. Below we have some screen shots of the about page showing the before and after.

Flip through the different pages. Since they all inherit from base, they will all be looking better, but not perfect! In the remainder of this chapter, we will go through a number of changes to the templates and use various Bootstrap classes to improve the look and feel of Rango.

<!--
## Page Headers
Now that we have the `base.html` all set up and ready to go, we can do a
really quick face lift to Rango by going through the Bootstrap
components and selecting the ones that suit the pages.

Let's start by updating all our templates by adding in the class `page-header` to the `<h1>` title tag at the top of each page. For example the `about.html` would be updated as follows.


{lang="html",linenos=off}
	{% extends 'rango/base.html' %}
	{% load staticfiles %}
	{% block title_block %}
		About
	{% endblock %}
	
	{% block body_block %}
		<div>
		<h1 class="page-header">About Page</h1>			
			This tutorial has been put together by: leifos and maxwelld90 
		</div>	
		<img src="{% static "images/rango.jpg" %}" alt="Picture of Rango" /> 
	{% endblock %}

This doesn't visually appear to change the look and feel, but it informs the toolkit what is the title text, and if we change the theme then it will be styled appropriately.

-->

![A screenshot of the About page without styling.](images/ch12-about-nostyling.png)

![A screenshot of the About page with Bootstrap Styling applied.](images/ch12-about-bootstrap.png)



### Sidebar Categories
For of all the sidebar categories are not displaying very nicely. If we take a look at the HTML source code for the [Dashboard page](https://getbootstrap.com/docs/4.2/examples/dashboard/), we can notice that a few classes have been added to the `<ul>` and `<li>` tags to denote that they are nav-items and nav-links respectively. So update the template as shown below.


{lang="html",linenos=off}
	<ul class="nav flex-column">
	{% if cats %}
	    {% for c in cats %}
	    {% if c == act_cat %}
	        <li  class="nav-item">
	              <a  class="nav-link active" href="{% url 'rango:show_category' c.slug %}">
					  <span data-feather="archive"></span>
					  {{ c.name }}</a>
	        </li>
	    {% else  %}
	        <li class="nav-item">
	            <a  class="nav-link" href="{% url 'rango:show_category' c.slug %}">
					<span data-feather="archive"></span>{{ c.name }}</a>
	        </li>
	    {% endif %}
	{% endfor %}
	{% else %}
	    <li class="nav-item">No Categories Yet!</li>
	{% endif %}
	</ul>


Also, rather than using `<strong>` to show that the category page has been selected, we have added the `active` class to the active category. We can also add in a `feather-icon` using the `<span. ... >' tag. Here we chose the `archive` icon, but you can see that there are loads of icons to choose from at Feather Icons (https://feathericons.com/)


### The Index Page
For the index page it would be nice to show the top categories and top pages in two separate columns, while the title is at the top.

 Looking at the Bootstrap examples, we can see that in the [Jumbotron](https://getbootstrap.com/docs/4.2/examples/jumbotron/)  example, they have a neat header element (i.e. the jumbotron) which we can put our title message in. And so we can update the index.html as follows:
 
 {lang="html",linenos=off}
 	<div class="jumbotron p-4">
 	<div class="container">
  		<h1 class="jumbotron-heading">Rango says...</h1>
 		<div>
 			<h2 class="h2">
 				{% if user.is_authenticated %}
 		 	   		Howdy {{ user.username }}!
 			   {% else %}
 	    	   		Hey there partner!
 			   {% endif %}
 			</h2>
         	<strong>{{ boldmessage }}</strong>
     	</div>
 	</div>
 	</div>
	
	
You might notice that after the `jumbotron` we have put `p-4`. The `p-4` controls the [spacing](https://getbootstrap.com/docs/4.2/utilities/spacing/) around the jumbotron. Try changing the padding to be `p-6` or `p-1`. You can also control the space of the top, bottom, left and right by specifically setting `pt`, `pb`, `pr` and `pl`. 
  
  
 X> ###Exercise 
 X> - Update all other templates so that the page heading is encapsulated within a jumbotron.
 X> This will make the whole application have a consistent look and feel.
  
 
 Then to create the two columns, we draw upon the [Album](https://getbootstrap.com/docs/4.2/examples/album/) example. While it has three columns, called cards, we only need two. Most, if not all, CSS frameworks use a [grid layout](https://getbootstrap.com/docs/4.2/layout/grid/)  consisteing of 12 blocks. If you inspect the HTML souce for the Album you will see that within a row there is a `<div>` which sets the size of the cards `<div class="col-md-4">` followed by `<div class="card mb-4 shadow-sm">`. This sets each card to be 4 units in length relative to the width (and 4 by 3 is 12). Since we want two cards (one for the most popular pages and most popular categories) then we can change the 4 to a 6 (i.e. 2 by 6 is 12). And so you can update the `index.html` with the following HTML.
 
 {lang="html",linenos=off}
	 <div class="container">
	 <div class="row">
	 <div class="col-md-6">
	 <div class="card mb-6">
	 <div class="card-body">
	 	<h2 >Most Liked Categories</h2>
	    <p class="card-text">
	 	{% if categories %}
	 		<ul>
	 	  	{% for category in categories %}
	 	  	<li>
	 	  	<a href="{% url 'rango:show_category' category.slug %}">{{ category.name }}</a>
	 	  	</li>
	 	  	{% endfor %}
	 	  	</ul>
	 	  	{% else %}
	 	  	<strong>There are no categories present.</strong>
	 	  	{% endif %}
	 	</p>
	</div>
	</div>
	</div>
	<div class="col-md-6">
	<div class="card mb-6">
	<div class="card-body">
		<h2>Most Viewed Pages</h2>
	    <p class="card-text">
	 	{% if pages %}
	 	<ul>
	 		{% for page in pages %}
	 		<li>
	 		<a href="{{ page.url }}">{{ page.title }}</a>
	 		</li>
	 		{% endfor %}
	 		</ul>
	 		{% else %}
	 		<strong>There are no pages present.</strong>
	 		{% endif %}
	        </p>
	</div>
	</div>
	</div>
	</div>
	</div>
 
Once you have updated the template, reload the page - it should look a lot better now, but the way the list items are presented is pretty horrible. 

Let's use the [list group styles provided by Bootstrap](https://getbootstrap.com/docs/4.2/components/list-group/) to improve how they look. We can do this quite easily by changing the `<ul>` elements to `<ul class="list-group">` and the `<li>` elements to `<li class="list-group-item">`. Reload the page, any better?

![A screenshot of the Index page with a Jumbotron and Columns.](images/ch12-styled-index.png)

###The Login Page
Now let's turn our attention to the login page. On the Bootstrap website you can see they have already made a [nice login form](https://getbootstrap.com/docs/4.2/examples/signin/). If you take a look at the source, you'll notice that there are a number of classes that we need to include to stylise the basic login form. Update the `body_block` in the `login.html` template as follows:

{lang="html",linenos=off}
	{% block body_block %}
	<link href="https://getbootstrap.com/docs/4.0/examples/signin/signin.css"
	    rel="stylesheet">
	<div class="jumbotron p-4">
	    <h1 class="jumbotron-heading">Login</h1>
	</div>
	<div class="container">
		
	<form class="form-signin" role="form" method="post" action=".">
	    {% csrf_token %}
	    <h2 class="form-signin-heading">Please sign in</h2>
	    <label for="inputUsername" class="sr-only">Username</label>
	    <input type="text" name="username" id="id_username" class="form-control" 
	           placeholder="Username" required autofocus>
	    <label for="inputPassword" class="sr-only">Password</label>
	    <input type="password" name="password" id="id_password" class="form-control"
	           placeholder="Password" required>
	    <button class="btn btn-lg btn-primary btn-block" type="submit" 
	            value="Submit" />Sign in</button>
	</form>

	</div>
	{% endblock %}

Besides adding in a link to the bootstrap `signin.css`, and a series of changes to the classes associated with elements, we have removed the code that automatically generates the login form, i.e. `form.as_p`. Instead, we took the elements, and importantly the `id` of the elements generated and associated them with the elements in this bootstrapped form. To find out what these `id`s were, we ran Rango, navigated to the page, and then inspected the source to see what HTML was produced by the `form.as_p` template tag. 

In the button, we have set the class to `btn` and `btn-primary`. If you check out the [Bootstrap section on buttons](https://getbootstrap.com/docs/4.2/components/buttons/) you can see there are lots of different colours, sizes and styles that can be assigned to buttons.

![A screenshot of the login page with customised Bootstrap Styling.](images/ch12-styled-login.png)

### Other Form-based Templates
You can apply similar changes to `add_cagegory.html` and `add_page.html` templates. For the `add_page.html` template, we can set it up as follows.

{lang="html",linenos=off}
	{% extends "rango/base.html" %}
	{% block title %}Add Page{% endblock %}
	
	{% block body_block %}
		<div class="jumbotron p-4">
			<div class="container">
		 		<h1 class="jumbotron-heading">Add Page to {{category.name}}</h1>
			</div>
		</div>

		<div class="container">
			<div class="row">
		       <form role="form" id="page_form" method="post" 
		             action="/rango/category/{{category.slug}}/add_page/">
		       {% csrf_token %}
		       {% for hidden in form.hidden_fields %}
		           {{ hidden }}
		       {% endfor %}
		       {% for field in form.visible_fields %}
		           {{ field.errors }}
		           {{ field.help_text }}<br/>
		           {{ field }}<br/>
				   <div class="p-2"></div>
		       {% endfor %}
		       <br/>
		       <button class="btn btn-primary"
		               type="submit" name="submit">
		           Add Page
		       </button>
			   <div class="p-5"></div>
		       </form>
		   </div>
		</div>
	{% endblock %}

X> ###Exercise 
X> - Create a similar template for the Add Category page called `add_category.html`.

###The Registration Template
For the `registration_form.html`, we can update the form as follows:

{lang="python",linenos=off}
    {% extends "rango/base.html" %}
    {% block body_block %}

	<div class="jumbotron p-4">
		<div class="container">
	 		<h1 class="jumbotron-heading">Register Here</h1>
		
	</div>
	</div>

	<div class="container">
		<div class="row">

			<div class="form-group" >
		    <form role="form"  method="post" action=".">
		        {% csrf_token %}
		        <div class="form-group" >
		        <p class="required"><label class="required" for="id_username">
		            Username:</label>
		            <input class="form-control" id="id_username" maxlength="30" 
		                 name="username" type="text" />
		            <span class="helptext">
		            Required. 30 characters or fewer. Letters, digits and @/./+/-/_ only.
		            </span>
		        </p>
		        <p class="required"><label class="required" for="id_email">
		            E-mail:</label>
		            <input class="form-control" id="id_email" name="email" 
		                 type="email" />
		        </p>
		        <p class="required"><label class="required" for="id_password1">
		            Password:</label>
		            <input class="form-control" id="id_password1" name="password1"
		                type="password" />
		        </p>
		        <p class="required">
		            <label class="required" for="id_password2">
		                Password confirmation:</label>
		            <input class="form-control" id="id_password2" name="password2" 
		                 type="password" />
		            <span class="helptext">
		                 Enter the same password as before, for verification.
		            </span>
		        </p>
		        </div>
		        <button type="submit" class="btn btn-primary">Submit</button>
		    </form>
		</div>
		
	</div>
	</div>
	
    {% endblock %}

Again we have manually transformed the form created by the `{{ form.as_p }}` template tag, and added the various bootstrap classes.

W> ###Bootstrap, HTML and Django Kludge
W> This is not the best solution - we have kind of kludged it together. 
W> It would be much nicer and cleaner if we could instruct Django when building the HTML for the form to insert the appropriate classes. But we will leave that to you to figure out :-)


###Next Steps
In this chapter we have described how to quickly style your Django application using the Bootstrap toolkit. Bootstrap is highly extensible and it is relatively easy to change themes - check out the [StartBootstrap Website](http://startbootstrap.com/) for a whole series of free themes. Alternatively, you might want to use a different CSS toolkit like: [Zurb](http://zurb.com), [Gridism](http://cobyism.com/gridism/), [Pure](https://purecss.io) or [GroundWorkd](https://groundworkcss.github.io/groundwork/). Now that you have an idea of how to hack the templates and set them up to use a responsive CSS toolkit, we can now go back and focus on finishing off the extra functionality that will really pull the application together.

![A screenshot of the Registration page with customised Bootstrap Styling.](images/ch12-styled-register.png)

X> ### Another Style Exercise
X> While this tutorial uses Bootstrap, an additional, and optional exercise, would be to style Rango using one of the other responsive CSS toolkits.  If you do create your own style, let us know and we can link to it to show others how you have improved Rango's styling!
