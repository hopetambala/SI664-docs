# SI664: Django ORM QuerySet Examples

* 1.0 [Django Python shell](#shell)
* 2.0 [Django QuerySet examples](#examples)
  - 2.1 [Retrieving all objects](#examples_all_objects)
  - 2.2 [Working with many-to-many relationships](#examples_many_to_many)
  - 2.3 [Selecting related objects](#examples_related_objects)
  - 2.4 [Pruning duplicate data with distinct() and Count(value, distinct=True)](#examples_distinct)
  - 2.5 [Filtering with keyword args and Q objects](#examples_filtering)
  - 2.6 [Returning dictionaries or tuples](#examples_dicts_tuples)
  - 2.7 [Using F() expressions](#examples_f)
  - 2.8 [Sorting and imposing limits](#examples_sort)
  - 2.9 [Summary data](#examples_summary)
* 3.0 [Further reading: Django documentation of particular interest](#further_reading)

## <a name="app"></a>1.0 Django Python shell
Make sure you activate the `heritagesites` project virtual environment before running the Django 
Python shell.

#### macOS
```commandline
(venv) $ python3 manage.py shell
```

#### Windows
```commandline
(venv) > python manage.py shell
```

## <a name="examples"></a>2.0 Django QuerySet examples
The following examples represent the types of Django ORM queries you may be asked to 
write as part of a weekly assignment or as part of an exam.

### <a name="examples_all_objects"></a>2.1 Retrieving all objects
You can retrieve a `QuerySet` that contains a copy of all objects associated with a model 
instance by appending the `all()` clause to it.

Consider the following example. Invoking `HeritageSite.objects.all()` will return a `QuerySet` 
containing 1092 `HeritageSite` objects.

##### Retrieve all HeritageSite instances
```commandline
>>> from heritagesites.models import HeritageSite
>>> len(HeritageSite.objects.all())
1092
>>> HeritageSite.objects.all()
<QuerySet [<HeritageSite: <i>Aflaj</i> Irrigation Systems of Oman>, <HeritageSite: <I>Sacri Monti</I> of Piedmont and Lombardy>, '...(remaining elements truncated)...']>
```

The `QuerySet` is *iterable*, in other words, it is an object that has an `__iter__` method which
 returns an iterator. You can employ a `for` loop to iterate over the `QuerySet` list of 
 `HeritageSite objects` in order to extract object data.  
 
 In the following example a number of `HeritageSite` variable assignments are made, including the related heritage site category ("Cultural", "Mixed", "Natural") derived from the `HeritageSiteCategory` model by way of a foreign key relationship.  The values are then joined together and printed out (only the last 
 item is displayed).

##### Iterate over HeritageSite.objects.all()
```commandline
>>> from heritagesites.models import HeritageSite
>>> hs = HeritageSite.objects.all()

>>> hs.count()
1092

>>> for s in hs:
...     name = s.site_name
...     description = s.description
...     if s.justification:
...         justify = s.justification
...     else:
...         justify = 'Justification not provided.'
...     date_inscribed = s.date_inscribed
...     long = s.longitude
...     lat = s.latitude
...     area = s.area_hectares
...     category = s.heritage_site_category.category_name
...     transboundary = s.transboundary

...     site = ''.join([name, '\n', description, '\n', justify, '\n', str(date_inscribed), '\n', str(long), '\n', str(lat), '\n', str(area), '\n', category, '\n', str(transboundary), '\n'])

...     print(site)

[... last item]

ǂKhomani Cultural Landscape
<p>The ǂKhomani Cultural Landscape is located at the border with Botswana and Namibia . . . .</p>
Justification not provided.
2017
20.37458333
-25.68761111
959100.0
Cultural
0
```

### <a name="examples_many_to_many"></a>2.2 Working with many-to-many relationships
Recall that the `HeritageSite` model includes a `ManyToManyField` assignment: 

```commandline
country_area = models.ManyToManyField(CountryArea, through='HeritageSiteJurisdiction')
```

`HeritageSite.country_area` is an iterable object that contains other model instances linked to 
`HeritageSite` via the intermediary model `HeritageSiteJurisdiction`.  Rather like the dolls 
inside a [Matryoshka doll](https://en.wikipedia.org/wiki/Matryoshka_doll) we can access these 
related model instances using a `for` loop to iterate over `country_area.all()`.

##### Iterate over nested country_area.all()
```commandline
>>> from heritagesites.models import HeritageSite
>>> for site in HeritageSite.objects.all():
...     name = site.site_name
...     for ca in site.country_area.all():
...         country_area = ca.country_area_name
...         sub_region = ca.location.sub_region.sub_region_name
...         region = ca.location.region.region_name
...     category = site.heritage_site_category.category_name

...     site = ''.join([region, '\n', sub_region, '\n', country_area, '\n', name, '\n', category, '\n'])

...     print(site)

[... last three items]

Asia
Eastern Asia
China
Zuojiang Huashan Rock Art Cultural Landscape
Cultural

Europe
Northern Europe
Iceland
Þingvellir National Park
Cultural

Africa
Sub-Saharan Africa
South Africa
ǂKhomani Cultural Landscape
Cultural
```

### <a name="examples_related_objects"></a>2.3 Selecting related objects
You can minimize database calls and optimize the `QuerySet` creation process by using the `select_related(*fields)` clause to load additional object data available via direct foreign-key relationships.  

The following represents two calls to the database, one for the `ca` assignment and a second for 
the `dev_status` assignment:

##### Unoptimized database calls
```commandline
>>> from heritagesites.models import CountryArea
>>> ca = CountryArea.objects.get(country_area_id=111)                              <-- database call
>>> name = ca.country_area_name
>>> dev_status = ca.dev_status.dev_status_name                                     <-- database call

>>> print(''.join(['name: ', name, '\n', 'dev status: ', dev_status]))

name: Ireland
dev status: Developed
```

Using `select_related(*fields)` to pre-populate the QuerySet with the `Dev_Status` object, you can reduce the number of database hits to a *single* call:

##### Optimized database call using select_related()
```commandline
>>> from heritagesites.models import CountryArea
>>> ca = CountryArea.objects.select_related('dev_status').get(country_area_id=111) <-- database call
>>> name = ca.country_area_name
>>> dev_status = ca.dev_status.dev_status_name                                     <-- pre-populated 
```
 
`select_related(*fields)` can take multiple arguments:

##### Passing multiple arguments to select_related()
```commandline
>>> from heritagesites.models import CountryArea
>>> ca = CountryArea.objects.select_related('dev_status', 'location').get(country_area_id=111)
>>> name = ca.country_area_name
>>> region = ca.location.region.region_name                                        <-- pre-populated
>>> sub_region = ca.location.sub_region.sub_region_name                            <-- pre-populated
>>> dev_status = ca.dev_status.dev_status_name                                     <-- pre-populated

>>> country = ''.join(['region: ', region, '\n', 'subregion: ', sub_region, '\n', 'name: ', name, '\n', 'dev_status: ', dev_status])

>>> print(country)
region: Europe
subregion: Northern Europe
name: Ireland
dev_status: Developed
```

### <a name="examples_distinct"></a>2.4 Pruning duplicate data with distinct() and Count(value, distinct=True)
The `distinct(*fields)` clause eliminates duplicate field values in the same manner as a SQL `SELECT DISTINCT`. This `QuerySet` optimizer is particularly useful when joining across multiple models where duplicate data may lurk.

:warning: Django does *not* support referencing fields in `distinct(*fields)` when the database 
backend is MySQL.  Use `distinct()` only. Attempting to include field references will result in a
 runtime error:

```commandline
django.db.utils.NotSupportedError: DISTINCT ON fields is not supported by this database backend
``` 

:warning: Django will add all fields referenced in an `order_by()` to its SQL SELECT clause. This
 can cause unexpected results when using the `distinct()` clause.  See the Django `QuerySet` API 
 [distinct()](https://docs.djangoproject.com/en/2.1/ref/models/querysets/#distinct) documentation for more details.

The following two examples illustrate how use of `distinct()` can impact the number of results 
returned:

##### Retrieve countries/areas located in an intermediate region
```commandline
ca = CountryArea.objects\
... .select_related('location')\
... .values(name=F('location__intermediate_region__intermediate_region_name'))\
... .filter(name__isnull=False)\
... .order_by('name')

>>> ca.count()
108
```

##### Retrieve intermediate regions that include one or more countries/areas  
```commandline
ca = CountryArea.objects\
... .select_related('location')\
... .values(name=F('location__intermediate_region__intermediate_region_name'))\
... .filter(name__isnull=False)
... .distinct()\
... .order_by('name')

>>> ca.count()
8
```

:bulb: You can view the underlying query that the Django ORM creates and issues by calling the `.query` method:

##### View underlying SQL statement
```commandline
>>> print(str(ca.query))

SELECT DISTINCT `intermediate_region`.`intermediate_region_name` AS `name` 
  FROM `country_area` 
       INNER JOIN `location` 
               ON (`country_area`.`location_id` = `location`.`location_id`) 
       LEFT OUTER JOIN `intermediate_region` 
               ON (`location`.`intermediate_region_id` = `intermediate_region`.`intermediate_region_id`) 
WHERE `intermediate_region`.`intermediate_region_name` IS NOT NULL ORDER BY `name` ASC;
```

:bulb: You can also set a "distinct" boolean flag when using `Count` which is the 
equivalent of SQL `SELECT COUNT(DISTINCT <field>)`. See [https://docs.djangoproject.com/en/2
.1/ref/models/querysets/#id8](https://docs.djangoproject.com/en/2.1/ref/models/querysets/#id8).

### <a name="examples_filtering"></a>2.5 Filtering with keyword args and Q objects

#### 2.5.1 Filtering with keyword arguments
Retrieve a single object with the `get(**kwargs)` clause:

##### Retrieve a single HeritageSite instance with get()
```commandline
>>> from heritagesites.models import HeritageSite
>>> hs = HeritageSite.objects.select_related('heritage_site_category').get(heritage_site_id=375)
>>> name = hs.site_name

>>> print(name)
Acropolis, Athens
```

Retrieve one or more objects using the `filter(**kwargs)` clause. Note too the use of the `
.order_by(*fields)` clause to sort each object returned:

##### Applying a filter()
```commandline
>>> from heritagesites.models import HeritageSite
>>> hs = HeritageSite.objects\
... .select_related('heritage_site_category')\
... .filter(site_name__contains = 'Castle')\
... .order_by('site_name')

>>> for s in hs:
...     print(s.site_name)
...
Budapest, including the Banks of the Danube, the Buda Castle Quarter and Andrássy Avenue
Castle of the Teutonic Order in Malbork
Castles and Town Walls of King Edward in Gwynedd
Castles of Augustusburg and Falkenlust at Brühl
Collegiate Church, Castle and Old Town of Quedlinburg
Durham Castle and Cathedral
Forts and Castles, Volta, Greater Accra, Central and Western Regions
Gardens and Castle at Kroměříž
Kronborg Castle
Litomyšl Castle
Mir Castle Complex
San Pedro de la Roca Castle, Santiago de Cuba
Three Castles, Defensive Wall and Ramparts of the Market-Town of Bellinzona
Wartburg Castle

>>> hs.count()
14
```

#### 2.5.2 Filtering with Q() objects
Use `Q()` objects to encapsulate a collection of keyword arguments. This simplifies defining 
filters using OR ('|') and AND ('&') operators:

##### Applying a filter() using Q() objects
```commandline
>>> from heritagesites.models import CountryArea
>>> from django.db.models import Q
>>> ca = CountryArea.objects\
... .select_related('dev_status', 'location')\
... .filter(Q(iso_alpha3_code = 'CHN')|Q(iso_alpha3_code = 'HKG')|Q(iso_alpha3_code = 'MAC'))\
... .order_by('country_area_name')

>>> for c in ca:
...     name = c.country_area_name
...     region = c.location.region.region_name
...     sub_region = c.location.sub_region.sub_region_name
...     dev_status = c.dev_status.dev_status_name
...     iso_code = c.iso_alpha3_code

...     place = ''.join(['region: ', region, '\n', 'subregion: ', sub_region, '\n', 'name: ', name, '\n', 'ISO code: ', iso_code, '\n', 'dev_status: ', dev_status, '\n'])

...     print(place)
...
region: Asia
subregion: Eastern Asia
name: China
ISO code: CHN
dev_status: Developing

region: Asia
subregion: Eastern Asia
name: China, Hong Kong Special Administrative Region
ISO code: HKG
dev_status: Developing

region: Asia
subregion: Eastern Asia
name: China, Macao Special Administrative Region
ISO code: MAC
dev_status: Developing
```

A `Q()` object can be negated with the `~` operator:

##### Negating a Q() object
```commandline
>>> from heritagesites.models import CountryArea
>>> from django.db.models import Q
>>> ca = CountryArea.objects\
... .select_related('dev_status', 'location')\
... .filter(Q(country_area_name__startswith = 'A') & ~Q(country_area_name = 'Antarctica'))\
... .order_by('country_area_name')

>>> for c in ca:
...     print(c)
...
Afghanistan
Albania
Algeria
American Samoa
Andorra
Angola
Anguilla
Antigua and Barbuda
Argentina
Armenia
Aruba
Australia
Austria
Azerbaijan
>>> ca.count()
14
```

You can mix `Q()` objects and keyword arguments in `filter(**kwargs)` so long as the `Q()` objects 
are listed first:

##### Mixing keyword arguments with Q() objects
```commandline
>>> from heritagesites.models import CountryArea
>>> from django.db.models import Q
>>> ca = CountryArea.objects\
... .select_related('dev_status', 'location')\
... .filter(Q(country_area_name__startswith = 'A') & ~Q(country_area_name = 'Antarctica'), dev_status__dev_status_name = 'Developed')\
... .order_by('country_area_name')

>>> for c in ca:
...     print(c)
...
Albania
Andorra
Australia
Austria
>>> ca.count()
4
```

### <a name="examples_dicts_tuples"></a>2.6 Returning dictionaries or tuples
You can forgo the overhead associated with returning a `QuerySet` of model instances in favor of 
returning dictionaries or tuples by using either the `values(*fields, **expressions)` or 
`values_list(*fields, flat=False, named=False)` clauses. However, results are limited to one 
object per row.  This limitation prevents inclusion of object data otherwise available via 
many-to-many or other multi-valued relationships (e.g., reverse foreign key relationships).

#### 2.6.1 Calling values()
You can declare which fields to be included in a `QuerySet` by using the `values(*fields, 
**expressions)` clause.  This is akin to referencing a specific set of columns in a SQL `SELECT` 
statement.  

:warning: If you choose to use `values()` note that the `QuerySet` returned
 is composed of [dictionaries](https://docs.python.org/3/tutorial/datastructures.html#dictionaries) rather than Django model instances.

##### Returning dictionaries with values()
```commandline
>>> from heritagesites.models import CountryArea
>>> ca = CountryArea.objects\
... .select_related('dev_status')\
... .values('country_area_name', 'iso_alpha3_code')\
... .filter(location__intermediate_region__pk = 7)\
... .order_by('country_area_name')

>>> ca.count()
5

>>> for c in ca:
...     print(c)
...
{'country_area_name': 'Botswana', 'iso_alpha3_code': 'BWA'}
{'country_area_name': 'Eswatini', 'iso_alpha3_code': 'SWZ'}
{'country_area_name': 'Lesotho', 'iso_alpha3_code': 'LSO'}
{'country_area_name': 'Namibia', 'iso_alpha3_code': 'NAM'}
{'country_area_name': 'South Africa', 'iso_alpha3_code': 'ZAF'}
```

#### 2.6.2 Calling values_list()
Similarly, the `values_list(*fields, flat=False, named=False)` clause also allows one to specify 
which fields to include in a `QuerySet`. 

:warning: If you choose to use `values_list()` note that the `QuerySet` returned is composed of [tuples](https://docs.python.org/3/tutorial/datastructures.html#tuples-and-sequences).

##### Returning tuples with values_list()
```commandline
>>> from heritagesites.models import CountryArea
>>> ca = CountryArea.objects\
... .select_related('dev_status')\
... .values_list('country_area_name', 'iso_alpha3_code')\
... .filter(location__intermediate_region__pk = 7)\
... .order_by('country_area_name')

>>> ca
<QuerySet [('Botswana', 'BWA'), ('Eswatini', 'SWZ'), ('Lesotho', 'LSO'), ('Namibia', 'NAM'), ('South Africa', 'ZAF')]>

>>> ca.count()
5

>>> for c in ca:
...     print(c[0], c[1])
...
Botswana BWA
Eswatini SWZ
Lesotho LSO
Namibia NAM
South Africa ZAF
```

:bulb: If your `values_list()` is limited to a single field you can set the `flat` parameter 
to `True` in order to return the results as single values rather than one-tuples:

##### Returning single value tuples with values_list()
```commandline
>>> CountryArea.objects\
... .values_list('iso_alpha3_code')\
... .filter(location__intermediate_region__pk = 7)\
... .order_by('iso_alpha3_code')

<QuerySet [('BWA',), ('LSO',), ('NAM',), ('SWZ',), ('ZAF',)]>
```

##### Returning Flattened tuples with values_list()
```commandline
>>> CountryArea.objects\
... .values_list('iso_alpha3_code', flat = True)\
... .filter(location__intermediate_region__pk = 7)\
... .order_by('iso_alpha3_code')

<QuerySet ['BWA', 'LSO', 'NAM', 'SWZ', 'ZAF']>
```

:bulb: You can set the `named` parameter to `True` to have the results returned as a [namedtuple()](https://docs.python.org/3/library/collections.html#collections.namedtuple).

##### Returning named tuples with values_list()
```commandline
>>> CountryArea.objects\
... .select_related('dev_status')\
... .values_list('country_area_name', 'iso_alpha3_code', named = True)\
... .filter(location__intermediate_region__pk = 7)\
... .order_by('country_area_name')

<QuerySet [Row(country_area_name='Botswana', iso_alpha3_code='BWA'), Row(country_area_name='Eswatini', iso_alpha3_code='SWZ'), Row(country_area_name='Lesotho', iso_alpha3_code='LSO'), Row(country_area_name='Namibia', iso_alpha3_code='NAM'), Row(country_area_name='South Africa', iso_alpha3_code='ZAF')]>
```

### <a name="examples_f"></a>2.7 Using F() expressions
You can use an `F()` object to represent a model field or annotated column value. `F` 
expressions allows one to "alias" model field values and utilize them when querying the database.

:bulb: Once you declare a `F()` object you can use it in subsequent clauses chained to the 
`QuerySet` as is illustrated in the `.filter()` clause below:

##### Using F() expressions
```commandline
>>> from heritagesites.models import HeritageSite
>>> from django.db.models import F
>>> hs = HeritageSite.objects\
... .values('site_name', 'longitude', 'latitude', intermediate_region_name=F('country_area__location__intermediate_region__intermediate_region_name'), country_area_name=F('country_area__country_area_name'))\
... .filter(country_area_name = 'Botswana')
>>> hs.count()
2
>>> for s in hs:
...     print(s)
...
{'site_name': 'Okavango Delta', 'longitude': Decimal('22.90000000'), 'latitude': Decimal('-19.28333333'), 'intermediate_region_name': 'Southern Africa', 'country_area_name': 'Botswana'}
{'site_name': 'Tsodilo', 'longitude': Decimal('21.73333333'), 'latitude': Decimal('-18.75000000'), 'intermediate_region_name': 'Southern Africa', 'country_area_name': 'Botswana'}
```

:warning: List `F()` objects *after* keyword arguments in `values(*fields, **expressions)` in order to avoid triggering a syntax error:

```commandline
SyntaxError: positional argument follows keyword argument
``` 

### <a name="examples_sort"></a>2.8 Sorting and imposing limits
Previous examples sorted the `QuerySet` returned using the `order_by(*fields)` clause. The 
default sort order is _ascending_; to sort in _descending_ order add a hyphen ('-') in front of the 
keyword as is demonstrated below.

You can also impose a limit on the number of results returned using a subset of Python's [array slicing syntax](https://stackoverflow.com/questions/509211/understanding-pythons-slice-notation).

In the example below, the `QuerySet` is sorted by `area_hectares` (descending), followed by 
`site_name` (ascending) before limiting the `QuerySet` returned to the top five (5) largest protected areas.

#### Sorting in descending order and imposing limits
```commandline
>>> from heritagesites.models import HeritageSite
>>> hs = HeritageSite.objects.values('site_name', 'area_hectares').order_by('-area_hectares', 'site_name')[:5]
>>> hs.count()
5
>>> for s in hs:
...     site = ''.join(['site: ', s['site_name'], '\n', 'area (hectares): ', str(s['area_hectares']), '\n'])
...     print(site)
...
site: Phoenix Islands Protected Area
area (hectares): 40825000.0

site: Papahānaumokuākea
area (hectares): 36207499.0

site: Great Barrier Reef
area (hectares): 34870000.0

site: Galápagos Islands
area (hectares): 14066514.0

site: Kluane / Wrangell-St. Elias / Glacier Bay / Tatshenshini-Alsek
area (hectares): 9839121.0
```

### <a name="examples_summary"></a>2.9 Summary data

#### 2.9.1 Calling aggregate()
If you need to generate a summary value based on the *entire* `QuerySet` use the `aggregate()` clause.

##### Returning summary values with aggregate()
```commandline
>>> from heritagesites.models import HeritageSite
>>> from django.db.models import Avg, Max, Min, Sum

>>> HeritageSite.objects.aggregate(Max('area_hectares'))
{'area_hectares__max': 40825000.0}

>>> HeritageSite.objects.aggregate(Min('area_hectares'))
{'area_hectares__min': 0.0}

>>> HeritageSite.objects.aggregate(Avg('area_hectares'))
{'area_hectares__avg': 275951.4937591573}

>>> HeritageSite.objects.aggregate(Sum('area_hectares'))
{'area_hectares__sum': 301339031.18499976}

>>> HeritageSite.objects.all().count()
1092
```

The `aggregate()` clause can also accept multiple arguments:

##### Passing multiple arguments to aggregate()
```commandline
>>> HeritageSite.objects.aggregate(Max('area_hectares'), Min('area_hectares'), Avg('area_hectares'), Sum('area_hectares'))
{'area_hectares__max': 40825000.0, 'area_hectares__min': 0.0, 'area_hectares__avg': 275951.4937591573, 'area_hectares__sum': 301339031.18499976}
```

#### 2.9.2 Calling annotation()
If you need to generate summary values for *each* `QuerySet` item use the `annotate()` clause. The 
`annotate()` clause is the equivalent of a SQL `GROUP BY` clause.

Recall that a previous assignment involved using the `values(*fields, **expressions)` and 
`annotate()` clauses to return counts of Developed vs Developing countries / areas in Asia:

##### Grouping regions by development status using annotation()
```commandline
>>> loc = Location.objects\
... .select_related('region')\
... .values(region_name=F('region__region_name'), dev_status_name=F('countryarea__dev_status__dev_status_name'))\
... .filter(region__region_name='Asia')\
... .annotate(count=Count('dev_status_name'))\
... .order_by('dev_status_name')

>>> loc.count()
2

>>> for l in loc:
...     print(l)
...
{'region_name': 'Asia', 'dev_status_name': 'Developed', 'count': 3}
{'region_name': 'Asia', 'dev_status_name': 'Developing', 'count': 47}
```

Likewise, providing counts of countries / areas grouped by USND region and subregion also requires 
use of both the `values(*fields, **expressions)` and `annotate()` clauses:

##### Grouping countries/areas by region and subregion using annotation()
```commandline
>>> ca = CountryArea.objects\
... .select_related('location').values(region=F('location__region__region_name'), subregion=F('location__sub_region__sub_region_name'))\
... .annotate(country_area_count=Count('country_area_id'))\
... .order_by('region', 'subregion')

>>> ca.count()
18

>>> for c in ca:
...     print(c)
...
{'region': None, 'subregion': None, 'country_area_count': 1}
{'region': 'Africa', 'subregion': 'Northern Africa', 'country_area_count': 7}
{'region': 'Africa', 'subregion': 'Sub-Saharan Africa', 'country_area_count': 53}
{'region': 'Americas', 'subregion': 'Latin America and the Caribbean', 'country_area_count': 52}
{'region': 'Americas', 'subregion': 'Northern America', 'country_area_count': 5}
{'region': 'Asia', 'subregion': 'Central Asia', 'country_area_count': 5}
{'region': 'Asia', 'subregion': 'Eastern Asia', 'country_area_count': 7}
{'region': 'Asia', 'subregion': 'South-eastern Asia', 'country_area_count': 11}
{'region': 'Asia', 'subregion': 'Southern Asia', 'country_area_count': 9}
{'region': 'Asia', 'subregion': 'Western Asia', 'country_area_count': 18}
{'region': 'Europe', 'subregion': 'Eastern Europe', 'country_area_count': 10}
{'region': 'Europe', 'subregion': 'Northern Europe', 'country_area_count': 17}
{'region': 'Europe', 'subregion': 'Southern Europe', 'country_area_count': 16}
{'region': 'Europe', 'subregion': 'Western Europe', 'country_area_count': 9}
{'region': 'Oceania', 'subregion': 'Australia and New Zealand', 'country_area_count': 6}
{'region': 'Oceania', 'subregion': 'Melanesia', 'country_area_count': 5}
{'region': 'Oceania', 'subregion': 'Micronesia', 'country_area_count': 8}
{'region': 'Oceania', 'subregion': 'Polynesia', 'country_area_count': 10}
```

## <a name="further_reading"></a>3.0 Further reading: Django documentation of particular interest
* [QuerySet API](https://docs.djangoproject.com/en/2.1/ref/models/querysets/)
  * Methods that return new QuerySets
    - [all()](https://docs.djangoproject.com/en/2.1/ref/models/querysets/#all)
    - [annotate()](https://docs.djangoproject.com/en/2.1/ref/models/querysets/#annotate)
    - [distinct()](https://docs.djangoproject.com/en/2.1/ref/models/querysets/#distinct)
    - [filter()](https://docs.djangoproject.com/en/2.1/ref/models/querysets/#filter)
    - [order_by()](https://docs.djangoproject.com/en/2.1/ref/models/querysets/#order-by)
    - [select_related()](https://docs.djangoproject.com/en/2.1/ref/models/querysets/#select-related)
    - [values()](https://docs.djangoproject.com/en/2.1/ref/models/querysets/#values)
    - [values_list()](https://docs.djangoproject.com/en/2.1/ref/models/querysets/#values-list)
  * Operators that return new QuerySets
    - [AND (&)](https://docs.djangoproject.com/en/2.1/ref/models/querysets/#and)
    - [OR (|)](https://docs.djangoproject.com/en/2.1/ref/models/querysets/#or)
  * Methods that do not return QuerySets
    - [aggregate()](https://docs.djangoproject.com/en/2.1/ref/models/querysets/#aggregate)
    - [get()](https://docs.djangoproject.com/en/2.1/ref/models/querysets/#get)
    - [count()](https://docs.djangoproject.com/en/2.1/ref/models/querysets/#count)
  * Field lookups
    - [in](https://docs.djangoproject.com/en/2.1/ref/models/querysets/#in)
    - [isnull](https://docs.djangoproject.com/en/2.1/ref/models/querysets/#isnull)
    - [regex](https://docs.djangoproject.com/en/2.1/ref/models/querysets/#regex)
    - [startswith](https://docs.djangoproject.com/en/2.1/ref/models/querysets/#startswith)
  * Aggregate Functions
    - [Avg](https://docs.djangoproject.com/en/2.1/ref/models/querysets/#avg)
    - [Count](https://docs.djangoproject.com/en/2.1/ref/models/querysets/#id8)
    - [Max](https://docs.djangoproject.com/en/2.1/ref/models/querysets/#max)
    - [Min](https://docs.djangoproject.com/en/2.1/ref/models/querysets/#min)
    - [Sum](https://docs.djangoproject.com/en/2.1/ref/models/querysets/#sum)
* [Making Queries](https://docs.djangoproject.com/en/2.1/topics/db/queries/)
  * [Retrieving objects](https://docs.djangoproject.com/en/2.1/topics/db/queries/#retrieving-objects)
  * [Complex Lookups with Q objects](https://docs.djangoproject.com/en/2.1/topics/db/queries/#complex-lookups-with-q-objects)
* [Query Expressions](https://docs.djangoproject.com/en/2.1/ref/models/expressions/)
  * [F expressions](https://docs.djangoproject.com/en/2.1/ref/models/expressions/#f-expressions)
* [Aggregation](https://docs.djangoproject.com/en/2.1/topics/db/aggregation/)    