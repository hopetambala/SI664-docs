# Meeting 8 Exercise

In this assignment you will

* Install and configue django-crispy-forms
* Create a set of CRUD templates for adding, updating, and deleting Heritage Sites
* Stage, commit, and push your changes to your Github `heritagesites` repo

## 1.0 Installation

### 1.1 Activate the heritagesites virtual environment

##### macOS
```commandline
$ source venv/bin/activate
(venv) $
```

##### Windows
```commandline
> venv\Scripts\activate
(venv) >
```

### 1.2 Get the django-crispy-forms package
Install the [django-crispy-forms/](https://pypi.org/project/django-crispy-forms/) package.

##### macOS
```commandline
(venv) $ pip3 install django-crispy-forms
```

##### Windows
```commandline
(venv) > pip install django-crispy-forms
```

## 2.0 crispy_forms setup
Make the following changes to `mysite/settings.py`:

### 2.1 Register crispy_forms
Add the `crispy_forms` app to `INSTALLED_APPS`:

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'heritagesites.apps.HeritagesitesConfig',
    'crispy_forms'                                       # <-- Add
    'social_django',
    'test_without_migrations',
]
```

### 2.2 Add template pack key/value
Add the `CRISPY_TEMPLATE_PACK` key/value setting:

```python
# django-crispy-forms
CRISPY_TEMPLATE_PACK = 'bootstrap4'
```

## 3.0 CRUD forms, views, routes, and templates
Now the fun begins.  You will add new functionality to the `heritagesites` app that will permit 
authenticated users to add, modify, and delete Heritage Sites.

### 3.1 Back up database
Back up the `unesco_heritage_sites` database. Consider adding a date suffix to the dump file 
name (*-201811\[DD\]\[HH\]\[mm\].sql) to help distinguish the file from previous and future backups.

```commandline
$ mysqldump -uarwhyte --databases unesco_heritage_sites > unesco_heritage_sites-dump-201811060939.sql
```

### 3.2 Update models.py
Adjust the `HeritageSite` model in `models.py`. Change the `date_inscribed` field 
type from `models.TextField` to `models.IntegerField`:

```python
date_inscribed = models.IntegerField(blank=True, null=True)
```

Next, provide a canonical URL for the `HeritageSite` model.  The Django community considers it a best practice to add both a `get_absolute_url()` and `__str__()` method to every model (you will update the other models later). 

```python
from django.urls import reverse
```

```python
def get_absolute_url(self):
		# return reverse('site_detail', args=[str(self.id)])
		return reverse('site_detail', kwargs={'pk': self.pk})
```

### 3.3 Add forms.py
Create a new file named `forms.py` and place the file in your `heritagesites/` app directory. Use
 the Django helper class (`ModelForm`) to create a new `Form` class based on the `HeritageSite` model. Name the class `HeritageSiteForm`.  

##### forms.py
```python
from django import forms
from crispy_forms.helper import FormHelper
from crispy_forms.layout import Submit
from heritagesites.models import HeritageSite


class HeritageSiteForm(forms.ModelForm):
	class Meta:
		model = HeritageSite
		fields = '__all__'

	def __init__(self, *args, **kwargs):
		super().__init__(*args, **kwargs)
		self.helper = FormHelper()
		self.helper.form_method = 'post'
		self.helper.add_input(Submit('submit', 'submit'))
```

### 3.4 Add views
Add the following views to `views.py`. You are responsible for adding any missing imports.

##### SiteCreateView(generic.View)
```python
@method_decorator(login_required, name='dispatch')
class SiteCreateView(generic.View):
	model = HeritageSite
	form_class = HeritageSiteForm
	success_message = "Heritage Site created successfully"
	template_name = 'heritagesites/site_new.html'
	# fields = '__all__' <-- superseded by form_class
	# success_url = reverse_lazy('heritagesites/site_list')

	def dispatch(self, *args, **kwargs):
		return super().dispatch(*args, **kwargs)

	def post(self, request):
		form = HeritageSiteForm(request.POST)
		if form.is_valid():
			site = form.save(commit=False)
			site.save()
			for country in form.cleaned_data['country_area']:
				HeritageSiteJurisdiction.objects.create(heritage_site=site, country_area=country)
			return redirect(site) # shortcut to object's get_absolute_url()
			# return HttpResponseRedirect(site.get_absolute_url())
		return render(request, 'heritagesites/site_new.html', {'form': form})

	def get(self, request):
		form = HeritageSiteForm()
		return render(request, 'heritagesites/site_new.html', {'form': form})
```

##### SiteUpdateView()
```python
@method_decorator(login_required, name='dispatch')
class SiteUpdateView(generic.UpdateView):
	model = HeritageSite
	form_class = HeritageSiteForm
	# fields = '__all__' <-- superseded by form_class
	context_object_name = 'site'
	# pk_url_kwarg = 'site_pk'
	success_message = "Heritage Site updated successfully"
	template_name = 'heritagesites/site_update.html'

	def dispatch(self, *args, **kwargs):
		return super().dispatch(*args, **kwargs)

	def form_valid(self, form):
		site = form.save(commit=False)
		# site.updated_by = self.request.user
		# site.date_updated = timezone.now()
		site.save()

		# Current country_area_id values linked to site
		old_ids = HeritageSiteJurisdiction.objects\
			.values_list('country_area_id', flat=True)\
			.filter(heritage_site_id=site.heritage_site_id)

		# New countries list
		new_countries = form.cleaned_data['country_area']

		# TODO can these loops be refactored?

		# New ids
		new_ids = []

		# Insert new unmatched country entries
		for country in new_countries:
			new_id = country.country_area_id
			new_ids.append(new_id)
			if new_id in old_ids:
				continue
			else:
				HeritageSiteJurisdiction.objects \
					.create(heritage_site=site, country_area=country)

		# Delete old unmatched country entries
		for old_id in old_ids:
			if old_id in new_ids:
				continue
			else:
				HeritageSiteJurisdiction.objects \
					.filter(heritage_site_id=site.heritage_site_id, country_area_id=old_id) \
					.delete()

		return HttpResponseRedirect(site.get_absolute_url())
		# return redirect('heritagesites/site_detail', pk=site.pk)
```

##### SiteDeleteView(generic.DeleteView)
```python
@method_decorator(login_required, name='dispatch')
class SiteDeleteView(generic.DeleteView):
	model = HeritageSite
	success_message = "Heritage Site deleted successfully"
	success_url = reverse_lazy('site')
	context_object_name = 'site'
	template_name = 'heritagesites/site_delete.html'

	def dispatch(self, *args, **kwargs):
		return super().dispatch(*args, **kwargs)

	def delete(self, request, *args, **kwargs):
		self.object = self.get_object()

		# Delete HeritageSiteJurisdiction entries
		HeritageSiteJurisdiction.objects \
			.filter(heritage_site_id=self.object.heritage_site_id) \
			.delete()

		self.object.delete()

		return HttpResponseRedirect(self.get_success_url())
```

#### 3.5 Add new routes
Add three new `HeritageSite` routes.  Nothing broken here; just add the routes to 
`heritagesites/urls.py`:

* `path('sites/new/', views.SiteCreateView.as_view(), name='site_new')`
* `path('sites/<int:pk>/delete/', views.SiteDeleteView.as_view(), name='site_delete')`
* `path('sites/<int:pk>/update/', views.SiteUpdateView.as_view(), name='site_update')`


### 3.6 Add new templates
Add the following new templates to your `heritagesites` app. Then inspect each new file, 
restoring missing template tags and/or field values. 

:bulb: check `models.py` to confirm that you have restored all missing fields for the `HeritageSite` model.

##### site_new.html
Restore missing fields:

```html
{% extends 'heritagesites/base.html' %}

{% load crispy_forms_tags %}

{% block content %}

  <header>
    <h2>New Site</h2>
  </header>

  <!-- novalidate attribute required to render validation messages -->
  <form method="post" novalidate>
    {% csrf_token %}

    <div class="row">
      <div class="col-6">
        {{ form.heritage_site_category|as_crispy_field }}
      </div>
    </div>

    <div class="row">
      <div class="col-6">
        {{ form.country_area|as_crispy_field }}
      </div>
    </div>


    <!-- Restore missing fields rendered using Bootstrap rows and columns classes --> 
    
    
    <div class="row">
      <div class="col-3">
        {{ form.longitude|as_crispy_field }}
      </div>
      <div class="col-3">
        {{ form.latitude|as_crispy_field }}
      </div>
    </div>

    <div class="row">
      <div class="col-6">
        {{ form.area_hectares|as_crispy_field }}
      </div>
    </div>

    <div class="row">
      <div class="col-6">
        {{ form.transboundary|as_crispy_field }}
      </div>
    </div>

    <button type="submit" class="btn btn-outline-success">Add</button>
    &nbsp;<a class="btn btn-outline-secondary" href="{% url 'site' %}">cancel</a>
  </form>

{% endblock content %}
```

##### site_update.html
Restore missing template tags, fields and a missing button that the user can click to `cancel` 
the intended operation and return to `site.html`:

```html
<!-- Restore missing template tags -->

{% block content %}

  <header>
    <h2>Site Update</h2>
  </header>

  <!-- novalidate attribute required to render validation messages -->
  <form method="post" novalidate>
  {% csrf_token %}
  

  <!-- Restore missing fields rendered using Bootstrap rows and columns classes -->


  <div class="row">
    <div class="col-6">
      {{ form.justification|as_crispy_field }}
    </div>
  </div>

  <div class="row">
    <div class="col-6">
      {{ form.date_inscribed|as_crispy_field }}
    </div>
  </div>


  <!-- Restore missing fields rendered using Bootstrap rows and columns classes --> 


  <div class="row">
    <div class="col-6">
      {{ form.transboundary|as_crispy_field }}
    </div>
  </div>

  <button type="submit" class="btn btn-outline-success">Update</button>
  
  <!-- Restore missing button -->

</form>

{% endblock content %}
```

##### site_delete.html
Restore missing template tags and repair the broken `cancel` button that redirects the user to 
`site_detail.html`, adding the correct URL lookup value and fixing the missing primary key object 
reference with the appropriate `context_object_name` value.

```html
{% extends 'heritagesites/base.html' %}



  <form method="post">
    {% csrf_token %}
    <p>Are you sure you want to delete the Heritage Site "{{ site.site_name }}"?</p>
    <button type="submit" class="btn btn-outline-danger">delete</button>
    <a class="btn btn-outline-secondary" href="{% url 'some name' some_context_object_name.pk 
    %}">cancel</a>
  </form>


```

### 3.7 Update templates
Update the following templates, restoring missing template tags and/or field values as directed. 

##### site.html
Replace the `<header>` with the following:

```html
<header>
  <div class="row">
    <div class="col-sm-11">
      <h2>UNESCO Heritage Sites</h2>
    </div>
    <div class="col-sm-1">
      {% if user.is_authenticated %}
        <a class="btn btn-outline-secondary" href="{% url 'site_new' %}">new</a>
      {% endif %}
    </div>
  </div>
</header>
```

##### site_detail.html
Replace the contents of `site_detail.html` with the following code:

```html
{% extends 'heritagesites/base.html' %}
   
   {% load heritagesites_extras %}
   
   <!-- safe filter on for raw HTML stored in database -->
   {% block content %}
     <header>
       <div class="row">
         <div class="col-sm-10">
           <h2>{{ site.site_name | safe }}</h2>
         </div>
         <div class="col-xs-1">
           {% if user.is_authenticated %}
             <a class="btn btn-outline-secondary" href="{% url 'site_update' site.pk %}">edit</a>
           {% endif %}
         </div>
         <div class="col-xs-1">
           {% if user.is_authenticated %}
             &nbsp;<a class="btn btn-outline-warning" href="{% url 'site_delete' site.pk %}">delete</a>
           {% endif %}
         </div>
       </div>
     </header>
   
     {% if site.country_area.all %}
       <div class="row">
         <div class="col-sm-2">
           <p>Region</p>
         </div>
         <div class="col-sm-10">
           {% spaceless %}
           <p>
             {% for country_area in site.country_area.all %}
               {% if country_area.location.intermediate_region %}
                 {% ifchanged country_area.location.intermediate_region.intermediate_region_id %}
                   {% if forloop.first %}
                     {{ country_area.location.intermediate_region.intermediate_region_name.strip }}
                   {% else %}
                     {{ country_area.location.intermediate_region.intermediate_region_name.strip|add_leading_comma }}
                   {% endif %}
                 {% endifchanged %}
               {% elif country_area.location.sub_region %}
                 {% ifchanged country_area.location.sub_region.sub_region_id %}
                   {% if forloop.first %}
                     {{ country_area.location.sub_region.sub_region_name.strip }}
                   {% else %}
                     {{ country_area.location.sub_region.sub_region_name.strip|add_leading_comma }}
                   {% endif %}
                 {% endifchanged %}
               {% elif country_area.location.region %}
                 {% ifchanged country_area.location.region.region_id %}
                   {% if forloop.first %}
                     {{ country_area.location.region.region_name.strip }}
                   {% else %}
                     {{ country_area.location.region.region_name.strip|add_leading_comma }}
                   {% endif %}
                 {% endifchanged %}
               {% else %}
                 {% ifchanged country_area.location.planet_id %}
                   {{ country_area.location.planet.unsd_name.strip }}
                 {% endifchanged %}
               {% endif %}
             {% endfor %}
           </p>
           {% endspaceless %}
         </div>
       </div>
   
       <div class="row">
         <div class="col-sm-2">
           <p>Country / Area</p>
         </div>
         <div class="col-sm-10">
           {% spaceless %}
           <p>
             {% for country_area in site.country_area.all %}
               {% if forloop.last %}
                 {{ country_area.country_area_name.strip }} ({{ country_area.iso_alpha3_code.strip }})
               {% else %}
                 {{ country_area.country_area_name.strip }} {{ country_area.iso_alpha3_code.strip|add_parens|add_trailing_comma }}
               {% endif %}
             {% endfor %}
           </p>
           {% endspaceless %}
         </div>
       </div>
   
     {% endif %}
   
     {% if site.heritage_site_category.category_name %}
       <div class="row">
         <div class="col-sm-2">
           <p>Category</p>
         </div>
         <div class="col-sm-10">
           <p>{{ site.heritage_site_category.category_name }}</p>
         </div>
       </div>
     {% endif %}
   
     <!-- SHOW STUDENTS THAT TEMPLATE NEEDS FIXING: EXTRA <p> -->
     {% if site.description %}
       <div class="row">
         <div class="col-sm-2">
           <p>Description</p>
         </div>
         <div class="col-sm-10">
           {{ site.description | safe }}
           <!-- <p>{{ site.description | safe }}</p> -->
         </div>
       </div>
     {% endif %}
   
     <!-- SHOW STUDENTS THAT TEMPLATE NEEDS FIXING: EXTRA <p> -->
     {% if site.justification %}
       <div class="row">
         <div class="col-sm-2">
           <p>Justification</p>
         </div>
         <div class="col-sm-10">
           {{ site.justification | safe }}
           <!-- <p>{{ site.justification | safe }}</p> -->
         </div>
       </div>
     {% endif %}
   
     {% if site.date_inscribed %}
       <div class="row">
         <div class="col-sm-2">
           <p>Date inscribed</p>
         </div>
         <div class="col-sm-10">
           <p>{{ site.date_inscribed }}</p>
         </div>
       </div>
     {% endif %}
   
     {% if site.latitude and site.longitude %}
       <div class="row">
         <div class="col-sm-2">
           <p>Geo coordinates</p>
         </div>
         <div class="col-sm-4">
           <p>{{ site.latitude }}, {{ site.longitude }} (<span style="font-style:italic">lat</span>.,
             <span style="font-style:italic">long</span>.)</p>
         </div>
       </div>
     {% endif %}
   
     {% if site.area_hectares %}
       <div class="row">
         <div class="col-sm-2">
           <p>Area</p>
         </div>
         <div class="col-sm-10">
           <p>{{ site.area_hectares }} hectares</p>
         </div>
       </div>
     {% endif %}
   {% endblock content %}
   ```

### 3.8 custom template tags and filters
Django's template engine can be extended with [custom tags and filters](https://docs
.djangoproject.com/en/2.1/howto/custom-template-tags/). The `site_detail.html` includes a `{% load heritagesites_extras %}` tag. This tag loads a couple of custom filters that add either leading or trailing commas to rendered string values. 
 
##### add_leading_comma custom filter 
 ```html
{{ country_area.location.region.region_name.strip|add_leading_comma }}
```

Add a `templatetags/` directory to the `heritagesites` app.  You must add a `__init__.py` file 
(file with no content) that signals that the directory is to be treated as a Python package.


```html
heritagesites/
    templatetags/
        __init__.py
        heritagesites_extras.py
```

Next, add a `heritagesites_extra.py` to `templatetags/` composed of the following custom filters:

##### heritagesites_extras.py
```python
from django import template
from django.template.defaultfilters import stringfilter

register = template.Library()


@register.filter(name='add_leading_comma')
@stringfilter
def add_leading_comma(value):
	return ''.join([', ', value])


@register.filter(name='add_trailing_comma')
@stringfilter
def add_trailing_comma(value):
	return ''.join([value, ','])


@register.filter(name='add_parens')
@stringfilter
def add_parentheses(value):
	return ''.join(['(', value, ')'])
```

:warning: After adding the `templatetags` modules, you must stop and restart the Django 
development server in order to activate the custom tags and filters. 
 
## 4.0 Styling
The `heritagesites` app rendered text tends to rub up against the sides of the browser window.  
There are a few other issues worth addressing that help clean up the look-and-feel of the app:

### 4.1 base.html
First, adjust the Bootstrap 4 [spacing](https://getbootstrap.com/docs/4.1/utilities/spacing/) in `base.html` Add `px-*` (left, right padding) and `py-*` (top, bottom padding) classes to the `<nav>`, `<main>`, and `<footer>` elements.  

```html
<nav class="navbar navbar-expand-sm navbar-dark navbar-custom px-4 py-2">
```

```html
<main class="px-4 py-3">
```

```html
<footer class="px-4 py-2">
```

Second, add the Bootstrap [sticky-top](https://getbootstrap.com/docs/4.0/utilities/position/#sticky-top) class to the `<nav>` element.

```html
<nav class="navbar navbar-expand-sm navbar-dark navbar-custom sticky-top px-4 py-2">

```

Third, put the `navbar` items inside a row `<div>` to even out the formatting.  Combine list 
items (`<li>`) into a single unordered list (`<ul>`) and remove all references to the `mr-auto` 
class.

```html
<div class="row">
  <a class="navbar-brand" href="{% url 'home' %}">SI664 Heritage Sites</a>
    <ul class="navbar-nav">
      <li class="nav-item">
        <a class="nav-link" href="{% url 'about' %}">about</a>
    </li>
    <li class="nav-item">
      <a class="nav-link" href="{% url 'site' %}">sites</a>
    </li>
    <li class="nav-item">
      <a class="nav-link" href="{% url 'country_area' %}">countries/areas</a>
    </li>
    <li class="nav-item">
      <a class="nav-link" href="{% url 'login' %}">login</a>
    </li>
  </ul>
</div>
```

### 4.2 heritagesites.css
Comment out all `line-height`, `margin-top`, `margin-bottom`, and `padding` attributes in 
`heritagesites.css`.

```css
/* example: comment out padding */
.container-fluid {
    background-color: transparent;
    /* padding: 0.5rem 0.5rem; */
}
```

### 4.3 login.html
Fifth, adjust the `login.html` button styling to match button styling elsewhere in the app by 
changing the Bootstrap classes from `btn btn-success` to `btn btn-outline-success`:

```html
<button type="submit" class="btn btn-outline-success">Sign in</button>
```

## 5.0 Document your work
Choose a place that has yet to be recognized by UNESCO for its cultural and/or natural 
significance to humanity. Then do the following:

### 5.1 Add a new Heritage Site
Go to `site.html` and click the 'new' button to add a new Heritage Site. Fill out the "new" form.
 Provide values for the required fields (designated with an asterisk ('*')) only.

##### required fields
* Heritage site category
* Country area
* Site name
* Description
* Transboundary

After submitting the form take a screenshot of the new entry. Rename the screenshot

`<uniqname>-site_new.png`

If the entry has not been added, debug your app code.

### 5.2 Update the new Heritage Site
On the new entry's detail page, click the `edit` button.  Use Google [search](https://www.google.com/) to provide the 
longitude and latitude values for the new entry. Also provide the area in hectares. Depending on 
the site chosen, [Wikipedia](https://www.wikipedia.org/) entries sometimes provide area 
information. You can use Google to convert the value to hectares (search: "area to hectares"). If you can't find an area measure, provide a best-guess estimate.

After submitting the form take a screenshot of the updated entry. Rename the screenshot

`<uniqname>-site_updated.png`

:warning: leave the `date_inscribed` field empty. It's up to UNESCO to decide if the site
 you add should be designated a Heritage Site.
 
If the entry has not been updated, debug your app code.
 
### 5.3 Delete the entry
Open the MySQL Workbench.  Write a query that joins the `HeritageSiteJurisdiction` (hsj)table to 
the `HeritageSite` (hs) table (use an INNER JOIN) and return a record set of all Heritage Sites 
with a `heritage_site_id` value >= 1092. Your new entry should be listed in the result set along 
with two entries for the "Mosi-oa-Tunya / Victoria Falls" site. If 
not 
adjust the query until you retrieve your new 
Heritage Site entry. 
 
##### required fields
* hsj.heritage_site_jurisdiction_id, 
* hsj.heritage_site_id, 
* hsj.country_area_id, 
* hs.site_name

After executing the query take a screenshot of the query pane and the Result Grid. Rename the 
screenshot

`<uniqname>-mwb_site_new.png`

Next, on the new entry's detail page, click the `delete` button. On the delete page, click the `delete` button.
 
Return to the MySQL Workbench. Rerun your query. Your entry should be gone. If yes, take a 
screenshot of the query pane and the Result Grid.  Rename the screenshot

`<uniqname>-mwb_site_deleted.png`

If the entry has not been deleted, debug your app code.

### 5.4 Stage, commit and push changes to Github
Follow the Github guide for [macOS](../github/github-mac.md) or [Windows](../github/github-win.md) and after installing and configuring Git, stage, commit and push all changes to your Github `heritagesites` repo.

:warning: Before pushing code to your Github repo, pull the three secret key values out of 
`settings.py` and store them either as environment variables or in a `mysite/secrets.py` file. 
See the [Github guide](../github) for directions.

After pushing your code changes to Github take a screenshot of your repo's home page.  Rename the
 screenshot 
 
 `<uniqname>-github_repo.png`

### 5.5 Submit your screenshots
Create a zip archive of

* `<uniqname>-site_new.png` 
* `<uniqname>-site_updated.png`
* `<uniqname>-mwb_site_new.png`
* `<uniqname>-mwb_site_deleted.png`
* `<uniqname>-github_repo.png`

Name the archive `<uniqname>-si664-mtg8.zip`. Go to the Canvas assignment page and submit the zip
 archive.