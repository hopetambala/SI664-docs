# Final Project
You are responsible for the following final project deliverables:

## Deliverables
* 1.0 [Github repo](#github_repo)
* 2.0 [Logical data model](#data_model)
* 3.0 [MySQL data base](#mysql_db)
* 4.0 [Django app](#django_app)
* 5.0 [Django REST API](#django_rest_api)
* 6.0 [Extra Credit](#extra_credit)

## <a name="github_repo"></a>1.0 Github repo
Create a [Github](http://github.com) repo. You are free to name it what you will, although I 
recommend a name that reflects the theme of your final project. See my ["Setting up a Github Remote 
Repository"](https://github.com/UMSI-SI664-2018Fall/SI664-docs/blob/master/github/github-mac.md) 
if you need a refresher on setting up a repo. The following requirements also apply:  

### 1.1 README
Your repo *must* include a `README.md` file. See [assignment 9.3](./assignment_v9p3.md) for a 
description of the required structure and content of the final project `README.md` file.

### 1.2 LICENSE (optional)
A LICENSE file is optional. If you add one the [MIT license](https://opensource.org/licenses/MIT)
 is recommended.
 
### 1.3 .gitignore
Your repo *must* include a `.gitignore` file that lists Python, Django, local IDE 
and other directories and files to be excluded from your commits. 

### 1.4 Directory/file exclusions
Do *not* push to your repo the final project's local Python virtual environment `/venv` directory environment (e.g., IDE config files). In other words, exclude your Python virtual environment `/venv` directory in your `.gitignore`. Other files that you `must` exclude: hidden system files (e.g., `.DS_Store`, `Thumbs.db`), personal IDE config files, files generated at runtime, compiled code, and build output directories.

:bulb: If you have previously pushed your virtual environment directory and files inadvertently 
to your repo, do the following:
* Deactivate your virtual environment 
* Move your `/venv` directory out of your project temporarily.
* Check your `.gitignore` file and make sure it excludes your project's `/venv`.
* Stage, commit and push all changes to your repo.  This will remove the `/venv` 
directory from your repo. 
* Move the `/venv` directory back to your project directory. Activate it.
* Issue a `git status` and confirm that the `/venv` directory is now excluded from future commits.

### 1.5 Related assignment(s)
* [Exercise 9.3](./assignment_v9p3.md)

## <a name="github_repo"></a>2.0 Logical data model
Create a logical data model that defines the entities, their attributes, and the relationships 
between entities that describe your problem domain. The following requirements also apply: 

### 2.1 Design
The design of your logical data model must reflect a data normalization strategy that adheres to the requirements imposed by third normal form (3NF). Identify candidate entities and establish entity relationships in terms of their cardinality (e.g. one-to-many, many-to-many). Ensure that data you identify as an attribute of a particular entity (i.e., a property) express a fact about the entity and no other entity. The goal is a logical data model that eliminates data duplication and ensures referential integrity.

### 2.2 Size
The logical data model should comprise *approximately* 7-10 tables. 

### 2.3 Entity relationships
It is expected that 3-4 one-to-many relationships will be expressed by the logical data model. In
 addition the logical data model *must* define *at least one* many-to-many relationship between two
  of the entities described in the model (e.g., `artist` - `artist_artwork` - `artwork`).  
  Utilize Crow's foot notation to illustrate the relationships.

### 2.4 *.mwb file
Place a copy of the MySQL Workbench logical data model `*.mwb` file in the final project's root 
level `static/sql` directory.

### 2.5 Screenshot
Take a screenshot of the logical data model and *render* the image in the `README.md` file (a link to the image file is not sufficient). Locate the screenshot image file in the final project's root level `static/img` directory.

### 2.6 Related assignment(s)
* [Exercise 3.2](./assignment_v3p2.md)
* [Exercise 8.3](./assignment_v8p3.md)

## <a name="mysql_db"></a>3.0 MySQL database
Write a data import script that transforms your flat file data set into a MySQL relational 
database. You can embed your SQL statements in a Python script or write a MySQL script (`*.sql`) 
script that creates and populates your database with data. 

:bulb: Feel free to utilize the `run_mysql_script.py` script available in the 
[SI664-repo](https://github.com/UMSI-SI664-2018Fall/SI664-scripts/tree/master/scripts) to execute 
your *.sql script.

### 3.1 Python/SQL scripts
Place a copy of the script(s) used to create and populate your database in the final project's root 
level `static/sql` directory.

### 3.2 Database dump file
Generate a *.sql dump file of your database and place a copy of it in the final project's root 
level `static/sql` directory. I will use the dump file to create a local instance of your 
database for evaluation purposes.

### 3.3 Related assignment(s)
* [Exercise 4.2 (macOS)](./assignment_v4p2_mac.md)
* [Exercise 4.2 (Windows)](./assignment_v4p2_win.md)
* [Exercise 5.2 (macOS)](./assignment_v5p2_mac.md)
* [Exercise 5.2 (Windows)](./assignment_v5p2_win.md)
* [Exercise 10.2](./assignment_v10p2.md)

## <a name="django_app"></a>4.0 Django app
Design and implement a Django app that interacts with your MySQL database. This will require you 
to create models, views, templates, URL routes as well as one or more forms and filters. Feel 
free to mimic [heritagesites app](https://github.com/UMSI-SI664-2018Fall/heritagesites) 
capabilities (repo now public) as well as borrow code from it. Also consider using Django's 
`inspectdb` utility to auto-generate your models as described in Exercise 4, [section 2.6]
(https://github.com/UMSI-SI664-2018Fall/SI664-docs/blob/master/exercises/assignment_v4p2_mac.md#26-auto-generate-unescounsd-models). You will need to review and likely edit the auto-generated `models.py` file but it will save you time. The following requirements also apply:

### 4.1 Bootstrap 4 styling
Style your app with [Bootstrap 4](https://getbootstrap.com/docs/4
.1/getting-started/introduction/). See the `heritagesites` app `base.html` file for how to add 
the required Bootstrap 4 CDN `<link>` and JQuery, Popper, and Boostrap `<script>` tags to your 
`base.html` file. Consider re-purposing your local `heritagesites.css` file for your final project app. Stay 
with the color palette you chose for your `heritagesites` app or choose 3-5 new colors. See Exercise 6, 
[section 6.0](https://github.com/UMSI-SI664-2018Fall/SI664-docs/blob/master/exercises
/assignment_v6p2_mac.md#60-static-assets) regarding how to set up your app's `/static` directory 
structure.

### 4.2 Home/About page
Create a home or about page that provides a brief description of your app. 

### 4.3 Login page (+Social Login)
Install the `social-auth-app-django` package and build a login page that also permits a user to 
authenticate using their Google credentials. Utilize the Google credentials you created for 
[Exercise 7.2](./assignment_v7p2.md).

:warning: do not place your secret values in `/mysite/settings.py`. Locate them elsewhere as 
described in "Setting up a Github Remote Repository", [section 3.5](https://github.com/UMSI-SI664-2018Fall/SI664-docs/blob/master/github/github-mac.md#35-hide-secret-key-values).

### 4.4 List and detail pages
Create at least one list view (`ListView`) and supporting template that displays a list of the 
principal entities described in your model (e.g., artworks, hospitals, movies, super heroes, wines, 
etc.). Each list item *must* provide a clickable link to an associated "detail" view (`DetailView`) and template. The detail page *must* display a set of entity attributes. Attributes lacking values (e.g. NULL) *must* be hidden from view.

:bulb: Implementing a *single* list page with linked detail pages will satisfy this requirement. 

### 4.5 Add/Update/Delete forms
Install the `django-crispy-forms` package and implement a single set of Add/Update/Delete forms 
that operate on your principle entity. Create a `forms.py` files and use the Django helper class (`ModelForm`)
 to create a new `Form` class based on your chosen model.  Add links to the forms (displayed as buttons) on your 
list page (Add) and your detail page (Update, Delete) as described in [Exercise 8.2](./assignment_v8p2.md).

### 4.6 Filters
Install the `django-filter` package and implement a minimum *two* filters. Add a `filters.py` 
file and implement at least one `CharFilter` (text) and one `ModelChoiceFilter` (dropdown) that are 
based on your principal model as described in [Exercise 9.2](./assignment_v9p2.md). You may embed your filter 
in your list page or embed it in a navigation bar or place it elsewhere.  

### 4.7 Navigation
Provide a means to navigate between the various views provided by your app. I recommend that 
implement Bootstrap 4's navigation header, the [navbar](https://getbootstrap.com/docs/4
.0/components/navbar/).

### 4.8 Related assignment(s)
* [Exercise 4.2 (macOS)](./assignment_v4p2_mac.md)
* [Exercise 4.2 (Windows)](./assignment_v4p2_win.md)
* [Exercise 5.2 (macOS)](./assignment_v5p2_mac.md)
* [Exercise 5.2 (Windows)](./assignment_v5p2_win.md)
* [Exercise 6.2 (macOS)](./assignment_v6p2_mac.md)
* [Exercise 6.2 (Windows)](./assignment_v6p2_win.md)
* [Exercise 7.2](./assignment_v7p2.md)
* [Exercise 8.2](./assignment_v8p2.md)
* [Exercise 9.2](./assignment_v9p2.md)

## <a name="django_rest_api"></a>5.0 Django REST API

5.1 POST/ PUT/DELETE (one entity)
5.2 Token Authentication
5.3 Swagger Doc

Include references to assignments

## <a name="extra_credit"></a>6.0 Extra Credit

6.1 Additional CRUD Forms
6.2 Additional REST endpoint


 

