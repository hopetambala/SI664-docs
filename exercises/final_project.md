# Final Project
You are responsible for the following final project deliverables:

### Deliverables
* 1.0 [Github repo](#github_repo)
* 2.0 [Logical data model](#data_model)
* 3.0 [MySQL data base](#mysql_db)
* 4.0 [Django app](#django_app)
* 5.0 [Django REST API](#django_rest_api)
* 6.0 [Extra Credit](#extra_credit)

## <a name="github_repo"></a>1.0 Github repo
Create a [Github](http://github.com) repo. You are free to name it what you will, although I 
recommend a name that reflects the theme of your final project.  The following requirements also 
apply:  

* __.gitignore:__ Your repo *must* include a `.gitignore` file that lists Python, Django, local IDE 
and other directories and files to be excluded from your commits.
* __LICENSE:__ Optional.
* __README.md:__ Your repo *must* include a `README.md` file. See [assignment 9.3](
./assignment_v9p3.md) for a description of the required structure and 
content of the `README.md` file. 
* __No `/venv` directory:__ Do *not* push to your repo the project's local Python virtual 
environment `/venv` directory environment (e.g., IDE config files). Exclude your Python virtual environment 
`/venv` directory.
* __Other files to exclude:__  hidden system files (e.g., `.DS_Store`, `Thumbs.db`), personal IDE 
config files, files generated at runtime, compiled code, build output directories.

:bulb: If you have previously pushed your virtual environment directory and files inadvertently 
to your repo, do the following:
* Deactivate your virtual environment 
* Move your `/venv` directory out of your project temporarily.
* Check your `.gitignore` file and make sure it excludes your project's `/venv`.
* Stage, commit and push all changes to your repo.  This will remove the `/venv` 
directory from your repo. 
* Move the `/venv` directory back to your project directory. Activate it.
* Issue a `git status` and confirm that the `/venv` directory is now excluded from future commits.

## <a name="github_repo"></a>2.0 Logical data model
You must embed in your final project's `README.md` file an *updated* screenshot of the 
final project logical data model initially created for [Assignment 8.3](./assignment_v8p3.md).

__Composition__: The updated logical data model should include approximately 7-10 tables and 
*must* define *at least one* many-to-many relationship between two of the entities described in the model (e.g., 
`artist` - `artist_artwork` - `artwork`).

__Visibility__: The logical data model screenshot *must* be rendered visible in the `README.md` 
file as an image rather than as a link.

__Location__: Place the data model image file in the root level `static/img` directory.

## <a name="mysql_db"></a>3.0 MySQL database

1. data import script (place in static/sql)
2. database
3. database dump file (place in static/sql)

Include references to assignments

## <a name="django_app"></a>4.0 Django app
Minimally viable project.  Free to mimic the `heritagesites` app as well as borrow code from it.

4.1 Home/About page 
4.2 Login page (+Social Login)
4.3 List page / Detail pages
4.4 Add / Update / Delete forms
4.5 Filter form (basic list search)
4.6 Navigation bar
4.7 Bootstrap 4 styling

Include references to assignments

## <a name="django_rest_api"></a>5.0 Django REST API

5.1 POST/ PUT/DELETE (one entity)
5.2 Token Authentication
5.3 Swagger Doc

Include references to assignments

## <a name="extra_credit"></a>6.0 Extra Credit

6.1 Additional CRUD Forms
6.2 Additional REST endpoint


 

