# SI664: Final Project (2000 points)
You are responsible for the following final project deliverables:

## Deliverables
* 1.0 [Github repo](#github_repo)
* 2.0 [Logical data model](#data_model)
* 3.0 [MySQL data base](#mysql_db)
* 4.0 [Django app](#django_app)
* 5.0 [Django REST API](#django_rest_api)
* 6.0 [Extra credit](#extra_credit) (optional)
* 7.0 [Admin user](#admin_user)
* 8.0 [Database dump file](#dump_file)
* 9.0 [Final project evaluation](#evaluation)

:bulb: Feel free to mimic [heritagesites app](https://github.com/UMSI-SI664-2018Fall/heritagesites) capabilities as well as borrow code from it. The repo that houses the code I wrote is now public.

## <a name="github_repo"></a>1.0 Github repo
Create a [Github](http://github.com) repo. You are free to name it what you will, although I 
recommend a name that reflects the theme of your final project. The following requirements apply:  

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
Do *not* push to your repo the final project's local Python virtual environment `/venv` directory
 environment (e.g., IDE config files). In other words, exclude your Python virtual environment 
 `/venv` directory in your `.gitignore`. Other files that you *must* exclude: hidden system files (e.g., `.DS_Store`, `Thumbs.db`), personal IDE config files, files generated at runtime, compiled code, and build output directories.

:bulb: If you have previously pushed your virtual environment directory and files inadvertently 
to your repo, do the following:
* Deactivate your virtual environment 
* Move your `/venv` directory out of your project temporarily.
* Check your `.gitignore` file and make sure it excludes your project's `/venv`.
* Stage, commit and push all changes to your repo.  This will remove the `/venv` 
directory from your repo. 
* Move the `/venv` directory back to your project directory. Activate it.
* Issue a `git status` and confirm that the `/venv` directory is now excluded from future commits.

### 1.5 Related resources
In addition to the weekly readings and other online resources, the following exercises are also relevant:

* A. Whyte, ["Setting up a Github Remote Repository"](https://github.com/UMSI-SI664-2018Fall/SI664-docs/blob/master/github/github-mac.md) 
* [Exercise 9.3](./assignment_v9p3.md)

## <a name="github_repo"></a>2.0 Logical data model
Create a logical data model that defines the entities, their attributes, and the relationships 
between entities that describe your problem domain. The following requirements apply:

### 2.1 Naming conventions
Table and property names *must* follow the naming conventions established in Simon Holywell's 
[SQL style code](https://www.sqlstyle.guide/).

:warning: Django is rather particular regarding the naming of primary keys. Implement this format: 
`<table_name>_id` (e.g., table=`director`, pk=`director_id`; table=`artist_artwork`; pk=`artist_artwork_id`). 
If you adopt a different format (e.g., table=`director`, pk=`dir_id`) the Django framework will 
ignore your primary key in favor of one of its own making (`director_id`) resulting in runtime 
errors due to key mismatches in the model layer.

### 2.2 Design
The design of your logical data model must reflect a data normalization strategy that adheres to the requirements imposed by third normal form (3NF). Identify candidate entities and establish entity relationships in terms of their cardinality (e.g. one-to-many, many-to-many). Ensure that data you identify as an attribute of a particular entity (i.e., a property) express a fact about the entity and no other entity. The goal is a logical data model that eliminates data duplication and ensures referential integrity.

### 2.3 Size
The logical data model should comprise *approximately* 7-10 tables. 

### 2.4 Entity relationships
It is expected that 3-4 one-to-many relationships will be expressed by the logical data model. In
 addition the logical data model *must* define *at least one* many-to-many relationship between 
 two of the entities described in the model (e.g., `artist` - `artist_artwork` - `artwork`). Utilize Crow's foot notation to illustrate the relationships.

### 2.5 *.mwb file
Place a copy of the MySQL Workbench logical data model `*.mwb` file in the final project's root 
level `static/sql` directory.

### 2.6 Screenshot
Take a screenshot of the logical data model and *render* the image in the `README.md` file (a link to the image file is not sufficient). Place the screenshot image file in the final project's root level `static/img` directory.

### 2.7 Related resources
In addition to the weekly readings and other online resources, the following exercises are also relevant:

* [Exercise 3.2](./assignment_v3p2.md)
* [Exercise 8.3](./assignment_v8p3.md)

## <a name="mysql_db"></a>3.0 MySQL database
Write a data import script that transforms your flat file data set into a MySQL relational database. You can embed your SQL statements in a Python script or write a MySQL script (`*.sql`) that creates and populates your database with data. Handle your csv files carefully otherwise file encoding issues or string length mismatches between the original source data and data you extract into new flat files [can occur](https://umich.instructure.com/courses/245664/discussion_topics/628836).

### 3.1 Foreign Key constraints
Make sure that all foreign key constraints that you define set both the `ON DELETE` and `ON UPDATE` values to `CASCADE` in order to ensure that any parent table row deletions that you initiate are properly cascaded through your database.

```mysql
CREATE TABLE IF NOT EXISTS heritage_site_jurisdiction
  (
    heritage_site_jurisdiction_id INTEGER NOT NULL AUTO_INCREMENT UNIQUE,
    heritage_site_id INTEGER NOT NULL,
    country_area_id INTEGER NOT NULL,
    PRIMARY KEY (heritage_site_jurisdiction_id),
    FOREIGN KEY (heritage_site_id) REFERENCES heritage_site(heritage_site_id)
    ON DELETE CASCADE ON UPDATE CASCADE,
    FOREIGN KEY (country_area_id) REFERENCES country_area(country_area_id)
    ON DELETE CASCADE ON UPDATE CASCADE
  )
ENGINE=InnoDB
CHARACTER SET utf8mb4
COLLATE utf8mb4_0900_ai_ci;
```

### 3.2 Python/SQL scripts
Place a copy of the script(s) used to create and populate your database in the final project's root 
level `static/sql` directory.

:bulb: Feel free to utilize the `run_mysql_script.py` script available in the 
[SI664-repo](https://github.com/UMSI-SI664-2018Fall/SI664-scripts/tree/master/scripts) to execute 
your *.sql script.

### 3.3 Related resources
In addition to the weekly readings, MySQL 8.0 [documentation](https://dev.mysql.com/doc/refman/8
.0/en/), and other online resources, the following exercises are also relevant:

* A. Whyte, ["Creating a Database using Flat Files"](https://github.com/UMSI-SI664-2018Fall/SI664-docs/blob/master/sql/mysql_create_database.md)
* A. Whyte, ["Reading Local Files (Windows)"](https://github.com/UMSI-SI664-2018Fall/SI664-docs/blob/master/sql/mysql-load_local_file_setup-win.md)
* A. Whyte, ["SELECT Statement Examples](https://github.com/UMSI-SI664-2018Fall/SI664-docs/blob/master/sql/mysql_select_examples.md)
* [Exercise 4.2 (macOS)](./assignment_v4p2_mac.md)
* [Exercise 4.2 (Windows)](./assignment_v4p2_win.md)
* [Exercise 5.2 (macOS)](./assignment_v5p2_mac.md)
* [Exercise 5.2 (Windows)](./assignment_v5p2_win.md)
* [Exercise 10.2](./assignment_v10p2.md)

## <a name="django_app"></a>4.0 Django app
Design and implement a Django app that interacts with your MySQL database. This will require you 
to create and activate a virtual environment, install package dependencies, use Django to start a project and start an app, create models, views, templates, URL routes as well as one or more forms and filters. The following requirements apply:

### 4.1 Models
Install the `mysqlclient` package and configure your database connection in `mysite/settings.py`.
 Consider using Django's `inspectdb` utility to auto-generate your models as described in Exercise 4, [section 2.6](https://github.com/UMSI-SI664-2018Fall/SI664-docs/blob/master/exercises/assignment_v4p2_mac.md#26-auto-generate-unescounsd-models). You will need to review and likely edit the auto-generated `models.py` file but it will save you time.

### 4.2 Bootstrap 4 styling
Style your app with [Bootstrap 4](https://getbootstrap.com/docs/4.1/getting-started/introduction/). See the `heritagesites` app `base.html` file for how to add the required Bootstrap 4 CDN `<link>` and JQuery, Popper, and Boostrap `<script>` tags to your `base.html` file. Consider re-purposing your local `heritagesites.css` file for your final project app. Stay with the color palette you chose for your `heritagesites` app or choose 3-5 new colors. See Exercise 6, [section 6.0](https://github.com/UMSI-SI664-2018Fall/SI664-docs/blob/master/exercises/assignment_v6p2_mac.md#60-static-assets) regarding how to set up your app's `/static` directory structure.

### 4.3 Home/About page
Create a home or about page that provides a brief description of your app. 

### 4.4 Login page (+Social Login)
Install the `social-auth-app-django` package and build a login page that also permits a user to 
authenticate using their Google credentials. Utilize the Google OAuth2 key and secret that you 
created for [Exercise 7.2](./assignment_v7p2.md).

:warning: Do not place your secret values in `/mysite/settings.py`. Locate them elsewhere as 
described in "Setting up a Github Remote Repository", [section 3.5](https://github.com/UMSI-SI664-2018Fall/SI664-docs/blob/master/github/github-mac.md#35-hide-secret-key-values).

### 4.5 List and detail pages
Create at least *one* list view (`ListView`) and supporting template that displays an instance 
list of the *principal* entity described in your model (e.g., artworks, hospitals, movies, super heroes, wines, etc.). Each list item instance *must* provide a clickable link to an associated detail view (`DetailView`) and template. The detail page *must* display a set of entity attributes. Attributes lacking values (e.g., blank or NULL) *must* be hidden from view.

### 4.6 Create/Update/Delete web forms
Install the `django-crispy-forms` package and implement a *single set* of create/update/delete web forms that operate on your *principal* entity (e.g., artwork, game, hospital, movie, wine). Create a `forms.py` files and use the Django helper class (`ModelForm`) to create a new `Form` class based on your chosen model.  In addition to the form submission buttons, add additional navigation buttons on your list page (add -> add form page), your detail page (update -> update form page, delete -> delete form page) and your delete page (cancel -> detail page) as described in [Exercise 8.2](./assignment_v8p2.md). 

Refer to the `heritagesites/urls.py` file to establish your web form routes.

#### heritagesites app urls.py
```python
urlpatterns = [
	...
	path('sites/', views.SiteFilterView.as_view(), name='site'),
	path('sites/new/', views.SiteCreateView.as_view(), name='site_new'),
	path('sites/<int:pk>/', views.SiteDetailView.as_view(), name='site_detail'),
	path('sites/<int:pk>/delete/', views.SiteDeleteView.as_view(), name='site_delete'),
	path('sites/<int:pk>/update/', views.SiteUpdateView.as_view(), name='site_update'),
]
```

:warning: Test your web forms and navigation buttons in order to ensure that they perform as 
intended.

### 4.7 Filter
Install the `django-filter` package and implement at least *one* filter. Add a `filters.py` file 
and implement a `CharFilter` (string search) or a `ModelChoiceFilter` (dropdown item search) that is based on your *principal* entity as described in [Exercise 9.2](./assignment_v9p2.md). You may embed your filter in your list page or place it in a navigation bar or elsewhere.  

If the output of your `FilterView` is a long list implement pagination. Grab the updated `heritagesite` app's [pagination.html](https://github.com/UMSI-SI664-2018Fall/heritagesites/blob/master/heritagesites/templates/heritagesites/pagination.html) template. URL query string values are now correctly passed between page clicks. Also see `views.py` and add the `PaginatedFilterView(generic.View)` class to your final project's `views.py`. It is a generic view mixin that returns a query string as a context [template](https://docs.djangoproject.com/en/2.1/ref/templates/api/) variable. Add this object to your the `FilterView` class to handle paginated filtering (see the `SiteFilterView(PaginatedFilterView, FilterView)` class).

:warning: Test your filter(s) in order to ensure that they perform as intended.

### 4.8 Navigation
Provide a means to navigate between the various views/pages provided by your app. I recommend 
that you implement Bootstrap 4's [navbar](https://getbootstrap.com/docs/4.0/components/navbar/) 
navigation header as was done for the `heritagesites` app, 

### 4.9 Related resources
In addition to the weekly readings, Django project [documentation](https://docs.djangoproject.com/en/2.1/), and other online resources, the following resources are also relevant:

* A. Whyte, ["Django ORM Queryset Examples"](https://github.com/UMSI-SI664-2018Fall/SI664-docs/blob/master/orm/django_orm.md)
* django-crispy-forms [documentation](https://django-crispy-forms.readthedocs.io/en/latest/)
* django-filters [documentation](https://django-filter.readthedocs.io/en/master/)
* social-auth-app-django [documentation](https://python-social-auth-docs.readthedocs.io/en/latest/configuration/django.html)
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
Create a Django `api` app and implement a *single set* of [OpenAPI](https://swagger.io/docs/specification/about/) complaint 
REST API GET/POST/PUT/DELETE endpoints for your final project.

### 5.1 package installs
Follow the directions laid out in [Exercise 10](https://github.com/arwhyte/SI664-docs/blob/master/exercises/assignment_v10p2.md) and install the following 
packages:

* `django-rest-framework`
* `django-cors-headers`
* `django-rest-auth`
* `django-allauth`
* `coreapi`
* `pyyaml`
* `django-rest-swagger`

Configure the newly installed apps in `mysite/settings.py` as described in the exercise.

### 5.2 api app
Create a final project Django `api` app. Follow the steps outlined in Exercise 10.2, [section 5.0](https://github.com/arwhyte/SI664-docs/blob/master/exercises/assignment_v10p2.md#50-create-a-django-api-app). 

Base your `api/views.py`, `api/serializers.py` and `api/urls.py` on the `heritagesites` `api` app [code](https://github.com/UMSI-SI664-2018Fall/heritagesites/tree/master/api). 

:warning: Once the `api` app is created and configured don't forget to run a database migration as outlined in Exercise 10.2, [section 5.8](https://github.com/arwhyte/SI664-docs/blob/master/exercises/assignment_v10p2.md#58-run-migrations).

### 5.3 Target entity
Choose an entity that has a *many-to-many* relationship with your principal entity. You will 
target this entity with your REST API GET/POST/PUT/DELETE endpoints.

An example:

* Principal entity: `artwork` (create/update/delete via web form-based CRUD operations)
* Associative table: `artwork_artist` (many-to-many relationships stored)
* Secondary entity: `artist` (create/update/delete via REST API CRUD operations)   

You are free to decide which entity to consider as the principal entity and which to consider as 
a secondary entity. Let the your data model's problem domain be your guide. 

*bulb*: In the above example, one might argue that the artist rather than the artwork (which 
represents the output of an artist's artistic genius) should be considered the principal 
entity. Yet art museums collect artworks not artists; an observation that determined behind my 
choice.

### 5.4 Endpoints
Refer to the `heritagesites` project files when establishing your API-related routes:

#### mysite/urls.py   

```python
urlpatterns = [
    ...
    path('api-auth/', include('rest_framework.urls')),
    ...
    path('heritagesites/api/', include('api.urls')),
    path('heritagesites/api/rest-auth/', include('rest_auth.urls')),
    path('heritagesites/api/rest-auth/registration/', include('rest_auth.registration.urls'))
]
```

#### api/urls.py
```python
urlpatterns = [
	path('', include(router.urls)),
	path('docs/', docs_view),
	path('swagger-docs/', schema_view)
]
```

#### Example REST API endpoints
```
GET /<final_project_app_name>/api/<entity_name_plural_form>/

example: GET /heritagesites/api/sites/

GET /<final_project_app_name>/api/<entity_name_plural_form>/{entity_id}

example: GET /heritagesites/api/sites/25/
```

```
POST /<final_project_app_name>/api/<entity_name_plural_form>/

example: POST /heritagesites/api/sites/
```

```
PUT /<final_project_app_name>/api/<entity_name_plural_form>/{entity_id}

example: PUT /heritagesites/api/sites/25/
```

```
DELETE /<final_project_app_name>/api/<entity_name_plural_form>/{entity_id}
       
example: DELETE /heritagesites/api/sites/25/
```

:warning: Use [Postman](https://www.getpostman.com/) to test your endpoints. Bad endpoint URLs or
 endpoints that fail to perform as advertised will impact your score negatively.
 
### 5.5 Token Authentication
The POST/PUT/DELETE endpoints *must* be protected by token authentication.

### 5.5 Swagger API documentation
I will refer to your Swagger API documentation when testing your endpoints. I'll check your 
`api/urls.py` file to confirm the path to your final project's Swagger docs. If you mimic the 
`heritagesites` api app I'll find them here:

```
http://localhost:8000/<final_project_app_name>/api/swagger-docs/
```

### 5.7 Related resources
In addition to the weekly readings and other online resources, the following exercises are also relevant:

* Django REST Framework [documentation](https://www.django-rest-framework.org/)
* Postman [documentation](https://learning.getpostman.com/docs/postman/launching_postman/installation_and_updates/)
* [Exercise 10.2](./assignment_v10p2.md)

## <a name="extra_credit"></a>7.0 Extra credit opportunities (optional)
The final project includes opportunities for extra credit:

### 6.1 Additional set of CRUD web forms (125 points)
Choose another entity and implement a second `ListView`/`DetailView` set of views along with 
accompanying templates and routes together with a set of create/update/delete web forms and earn 
up to 150 additional points. You can target the secondary entity chosen for your required REST API 
implementation.  Implement the second set of web forms in a manner similar to that described 
in section [4.0](#django_app).  

### 6.2 Additional Filters (50 points)
Add up to two additional filters to your `filters.py` `FilterSet` class and earn up to an 
additional 25 per points per filter added (50 points max). Each filter included in your final project *must* implement a *different* [filter](https://django-filter.readthedocs.io/en/master/ref/filters.html) type. See the `heritagesites` app [filters.py](https://github.com/UMSI-SI664-2018Fall/heritagesites/blob/master/heritagesites/filters.py) 
`HeritageSiteFilter(django_filters.FilterSet)` class for working examples. Implement your additional filters in a manner similar to that described in section [4.0](#django_app).

### 6.3 Additional set of REST API Endpoints (125 points)
Choose another entity and implement a *second set* of [OpenAPI](https://swagger.io/docs/specification/about/) complaint REST API GET/POST/PUT/DELETE endpoints for your final project. You can target the principal entity chosen for your required web forms implementation. Implement your extra credit endpoints in a manner similar to that described in section [5.0](#django_rest_api).
 
## <a name="admin_user"></a>7.0 Admin user
Using the Django admin site create a user account with the following attributes:

* user name: arwhyte
* password: goblue2018
* first name: Anthony
* last name: Whyte
* email: arwhyte@umich.edu

Once the `arwhyte` user is created, click the "Users" link under "AUTHENTICATION AND 
AUTHORIZATION", and click the username "arwhyte". In the "Change User" form under "Permissions" 
check the following boxes:

* Active (yes)
* Staff status (yes)
* Superuser status (yes)

Next, under "Auth Tokens", click "Tokens" and generate a token for the user `arwhyte`.

:bulb: I need this user in order to evaluate your apps.

## <a name="admin_user"></a>8.0 Database dump file
After creating the new `arwhyte` user account, applying the proper permissions, and assigning an 
authorization token, generate a MySQL dump file of your database named 
`<uniqname>-<database_name>-dump-YYYYMMDDhhmm.sql`.

Place the dump file your final project's root level `static/sql` directory. I will use the dump file to create a new 
instance of your database for evaluation purposes.

## <a name="evaluation"></a>9.0 Final project evaluation
Stage, commit and push all final project code and supporting files to your Github final project 
repo *no later than* _Thursday, 20 December 2018, 11:59 PM_. I will fork your Github repo, clone it 
locally, and install and run your final project apps, testing required capabilities and the REST 
API as part of my evaluation. 

As with the weekly assignments and the midterm partial credit will be awarded at the discretion 
of the instructor. 