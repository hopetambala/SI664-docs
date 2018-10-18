# SI664: Django ORM and QuerySet Examples

The following examples represent the types of Django ORM queries you may be asked to 
write as part of a weekly assignment or as part of an exam.

## 1.0 Django documentation
[QuerySet API](https://docs.djangoproject.com/en/2.1/ref/models/querysets/)
[Making Queries](https://docs.djangoproject.com/en/2.1/topics/db/queries/)
[Complex Lookups with Q objects](https://docs.djangoproject.com/en/2.1/topics/db/queries/#complex-lookups-with-q-objects)

## 2.0 Django Python shell
Make sure you activate the heritagesites project virtual environment before running the Django 
Python shell.

#### macOS
```commandline
(venv) $ python3 manage.py shell
```

#### Windows
```commandline
(venv) > python manage.py shell
```

## 3.0 Example Django ORM QuerySets

### 3.1 All objects
You can retrieve a `QuerySet` that contains all objects associated with a model by calling the 
`.all()` method on it.

Consider the following example. Invoking `HeritageSite.objects.all()` will return a `QuerySet` 
containing 1092 `HeritageSite` objects.

```commandline
>>> from heritagesites.models import HeritageSite
>>> len(HeritageSite.objects.all())
1092
>>> HeritageSite.objects.all()
<QuerySet [<HeritageSite: <i>Aflaj</i> Irrigation Systems of Oman>, <HeritageSite: <I>Sacri Monti</I> of Piedmont and Lombardy>, '...(remaining elements truncated)...']>
```

The `QuerySet` is *iterable*, in other words, it is an object that has an `__iter__` method which
 returns an iterator. You can employ a `for` loop to iterate over the `QuerySet` list of 
 `HeritageSite objects` in order to extract object data.  In the following example a number of 
 `HeritageSite` variable assignments are made, including the related heritage site category 
 ("Cultural", "Mixed", "Natural") derived from the `HeritageSiteCategory` model by way of a 
 foreign key relationship.  The values are then joined together and printed out (only the last 
 item is displayed).

```commandline
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

...     site = ''.join([name, '\n', description, '\n', justify, '\n', str(date_inscribed), '\n', str(long), '\n', str(lat), '\n', str(area), '\n', category, '\n', str(transboundary)])

...     print(site)

[... last record]

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

Recall that the `HeritageSite` model includes a `ManyToManyField` assignment: 

```commandline
country_area = models.ManyToManyField(CountryArea, through='HeritageSiteJurisdiction')
```

`HeritageSite.country_area` is an iterable object that contains other model objects linked to `HeritageSite` via the intermediary model `HeritageSiteJurisdiction`.  Rather like the dolls inside a [Matryoshka doll](https://en.wikipedia.org/wiki/Matryoshka_doll) we can access these related model objects using a `for` loop to iterate over `country_area.all()`.

```commandline
>>> for site in HeritageSite.objects.all():
...     name = site.site_name
...     for ca in site.country_area.all():
...         country_area = ca.country_area_name
...         sub_region = ca.location.sub_region.sub_region_name
...         region = ca.location.region.region_name
...     category = site.heritage_site_category.category_name

...     site = ''.join([region, '\n', sub_region, '\n', country_area, '\n', name, '\n', category])

...     print(site)

[... last three records]

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

### 3.2 Select related objects
You can minimize database calls by optimizing `QuerySet` creation using the `.select_related(*fields)` method to load additional object data available via direct foreign-key relationships.  

The following represents two calls to the database, one for the `ca` assignment and a second for 
the `dev_status` assignment:

```commandline
>>> from heritagesites.models import CountryArea
>>> ca = CountryArea.objects.get(country_area_id=111)                              <-- database call
>>> name = ca.country_area_name
>>> dev_status = ca.dev_status.dev_status_name                                     <-- database call

>>> print(''.join(['name: ', name, '\n', 'dev status: ', dev_status]))

name: Ireland
dev status: Developed
```

Using `.select_related(*fields)` to pre-populate the QuerySet with the `Dev_Status` object, we reduce 
the number of database hits to a *single* call:

```commandline
>>> from heritagesites.models import CountryArea
>>> ca = CountryArea.objects.select_related('dev_status').get(country_area_id=111) <-- database call
>>> name = ca.country_area_name
>>> dev_status = ca.dev_status.dev_status_name                                     <-- pre-populated 
```
 
`.select_related(*fields)` can take multiple arguments:
 
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

### 3.2 Filter objects
Retrieve a single object with the `.get(**kwargs)` method:

```commandline
>>> from heritagesites.models import HeritageSite
>>> hs = HeritageSite.objects.select_related('heritage_site_category').get(heritage_site_id=375)
>>> name = hs.site_name
>>> print(name)
Acropolis, Athens
```

Retrieve one or more objects using the `filter(**kwargs)` method:

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
>>> hs1.count()
14
```

Use `Q` objects to encapsulate a collection of keyword arguments. This permits the construction of
 complex database queries using | (OR) and & (AND) operators:

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

...     place = ''.join(['region: ', region, '\n', 'subregion: ', sub_region, '\n', 'name: ', name, '\n', 'ISO code: ', iso_code, '\n', 'dev_status: ', dev_status])

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

A `Q` object can be negated with the `~` operator:

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

You can mix `Q` objects and keyword arguements in `.filter(**kwargs)` so long as the `Q` objects 
are listed first:

```commandline
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

_______________________________________

### 3.x Aggregation and annotation


### 3.x F object










### Limit number of fields returned with .values()

```commandline
>>> from heritagesites.models import CountryArea
>>> ca = CountryArea.objects.filter(country_area_name__startswith = 'China').values('country_area_name', 'iso_alpha3_code')
>>> for c in ca:
...     print(c)
...
...
{'country_area_name': 'China', 'iso_alpha3_code': 'CHN'}
{'country_area_name': 'China, Hong Kong Special Administrative Region', 'iso_alpha3_code': 'HKG'}
{'country_area_name': 'China, Macao Special Administrative Region', 'iso_alpha3_code': 'MAC'}
```

### Limit number of fields returned with .only()

```commandline
>>> from heritagesites.models import HeritageSite
>>> hs = HeritageSite.objects.filter(site_name__startswith ='A').only('site_name', 'area_hectares')
>>> for s in hs:
...     print(s.site_name, s.area_hectares)
...
...
Aachen Cathedral  0.2
Aapravasi Ghat 0.164
Aasivissuit – Nipisat. Inuit Hunting Ground between Ice and Sea 417800.0
Abbey and Altenmünster of Lorsch 3.34
Abbey Church of Saint-Savin sur Gartempe 1.61
Abbey of St Gall 0.0
Abu Mena 182.72
Acropolis, Athens 3.04
. . .
```

### Subquery
```commandline
>>> from heritagesites.models import CountryArea, SubRegion
>>> from django.db.models import Subquery
>>> sr = SubRegion.objects.filter(sub_region_name = 'Eastern Asia')
>>> ca1 = CountryArea.objects.filter(sub_region_id = Subquery(sr.values('sub_region_id')))
>>> ca1.count()
7
>>> for c in ca1:
...     print(c.country_area_name)
...
...
China
China, Hong Kong Special Administrative Region
China, Macao Special Administrative Region
Democratic People's Republic of Korea
Japan
Mongolia
Republic of Korea
```

### JOIN across multiple tables

```commandline
>>> from heritagesites.models import CountryArea, Region, SubRegion, IntermediateRegion, DevStatus
>>> ca8 = HeritageSite.objects.select_related('heritage_site_category').filter(country_area__intermediate_region__intermediate_region_name = 'Southern Africa')
>>> for c in ca8:
...     print(c.site_name)
...
...
Barberton Makhonjwa Mountains
Cape Floral Region Protected Areas
Fossil Hominid Sites of South Africa
iSimangaliso Wetland Park
Maloti-Drakensberg Park
Maloti-Drakensberg Park
Mapungubwe Cultural Landscape
Namib Sand Sea
Okavango Delta
Richtersveld Cultural and Botanical Landscape
Robben Island
Tsodilo
Twyfelfontein or /Ui-//aes
Vredefort Dome
ǂKhomani Cultural Landscape

>>> str(ca8.query)
'SELECT `heritage_site`.`heritage_site_id`, `heritage_site`.`site_name`, `heritage_site`.`description`, `heritage_site`.`justification`, `heritage_site`.`date_inscribed`, `heritage_site`.`longitude`, `heritage_site`.`latitude`, `heritage_site`.`area_hectares`, `heritage_site`.`heritage_site_category_id`, `heritage_site`.`transboundary`, `heritage_site_category`.`category_id`, `heritage_site_category`.`category_name` FROM `heritage_site` INNER JOIN `heritage_site_jurisdiction` ON (`heritage_site`.`heritage_site_id` = `heritage_site_jurisdiction`.`heritage_site_id`) INNER JOIN `country_area` ON (`heritage_site_jurisdiction`.`country_area_id` = `country_area`.`country_area_id`) INNER JOIN `intermediate_region` ON (`country_area`.`intermediate_region_id` = `intermediate_region`.`intermediate_region_id`) INNER JOIN `heritage_site_category` ON (`heritage_site`.`heritage_site_category_id` = `heritage_site_category`.`category_id`) WHERE `intermediate_region`.`intermediate_region_name` = Southern Africa ORDER BY `heritage_site`.`site_name` ASC'
```

### Add a values_list()

```commandline
>>> hs = HeritageSite.objects.select_related('heritage_site_category').filter(country_area__country_area_name__startswith = 'China').values_list('country_area__region__region_name', 'country_area__sub_region__sub_region_name', 'country_area__country_area_name', 'site_name', 'heritage_site_category__category_name')
>>> for s in hs:
...     print(s)
...
...
('Asia', 'Eastern Asia', 'China', 'Ancient Building Complex in the Wudang Mountains', 'Cultural')
('Asia', 'Eastern Asia', 'China', 'Ancient City of Ping Yao', 'Cultural')
('Asia', 'Eastern Asia', 'China', 'Ancient Villages in Southern Anhui – Xidi and Hongcun', 'Cultural')
('Asia', 'Eastern Asia', 'China', 'Capital Cities and Tombs of the Ancient Koguryo Kingdom', 'Cultural')
('Asia', 'Eastern Asia', 'China', 'Chengjiang Fossil Site', 'Natural')
('Asia', 'Eastern Asia', 'China', 'China Danxia', 'Natural')
('Asia', 'Eastern Asia', 'China', 'Classical Gardens of Suzhou', 'Cultural')
('Asia', 'Eastern Asia', 'China', 'Cultural Landscape of Honghe Hani Rice Terraces ', 'Cultural')
('Asia', 'Eastern Asia', 'China', 'Dazu Rock Carvings', 'Cultural')
('Asia', 'Eastern Asia', 'China', 'Fanjingshan', 'Natural')
('Asia', 'Eastern Asia', 'China', 'Fujian <em>Tulou</em>', 'Cultural')
('Asia', 'Eastern Asia', 'China', 'Historic Centre of Macao', 'Cultural')
('Asia', 'Eastern Asia', 'China', 'Historic Ensemble of the Potala Palace, Lhasa', 'Cultural')
('Asia', 'Eastern Asia', 'China', 'Historic Monuments of Dengfeng in “The Centre of Heaven and Earth”', 'Cultural')
('Asia', 'Eastern Asia', 'China', 'Huanglong Scenic and Historic Interest Area', 'Natural')
('Asia', 'Eastern Asia', 'China', 'Hubei Shennongjia', 'Natural')
('Asia', 'Eastern Asia', 'China', 'Imperial Palaces of the Ming and Qing Dynasties in Beijing and Shenyang', 'Cultural')
('Asia', 'Eastern Asia', 'China', 'Imperial Tombs of the Ming and Qing Dynasties', 'Cultural')
('Asia', 'Eastern Asia', 'China', 'Jiuzhaigou Valley Scenic and Historic Interest Area', 'Natural')
('Asia', 'Eastern Asia', 'China', 'Kaiping Diaolou and Villages', 'Cultural')
('Asia', 'Eastern Asia', 'China', 'Kulangsu, a Historic International Settlement', 'Cultural')
('Asia', 'Eastern Asia', 'China', 'Longmen Grottoes', 'Cultural')
('Asia', 'Eastern Asia', 'China', 'Lushan National Park', 'Cultural')
('Asia', 'Eastern Asia', 'China', 'Mausoleum of the First Qin Emperor', 'Cultural')
('Asia', 'Eastern Asia', 'China', 'Mogao Caves', 'Cultural')
('Asia', 'Eastern Asia', 'China', 'Mount Emei Scenic Area, including Leshan Giant Buddha Scenic Area', 'Mixed')
('Asia', 'Eastern Asia', 'China', 'Mount Huangshan', 'Mixed')
('Asia', 'Eastern Asia', 'China', 'Mount Qingcheng and the Dujiangyan Irrigation System', 'Cultural')
('Asia', 'Eastern Asia', 'China', 'Mount Sanqingshan National Park', 'Natural')
('Asia', 'Eastern Asia', 'China', 'Mount Taishan', 'Mixed')
('Asia', 'Eastern Asia', 'China', 'Mount Wutai', 'Cultural')
('Asia', 'Eastern Asia', 'China', 'Mount Wuyi', 'Mixed')
('Asia', 'Eastern Asia', 'China', 'Mountain Resort and its Outlying Temples, Chengde', 'Cultural')
('Asia', 'Eastern Asia', 'China', 'Old Town of Lijiang', 'Cultural')
('Asia', 'Eastern Asia', 'China', 'Peking Man Site at Zhoukoudian', 'Cultural')
('Asia', 'Eastern Asia', 'China', 'Qinghai Hoh Xil', 'Natural')
('Asia', 'Eastern Asia', 'China', 'Sichuan Giant Panda Sanctuaries - Wolong, Mt Siguniang and Jiajin Mountains ', 'Natural')
('Asia', 'Eastern Asia', 'China', "Silk Roads: the Routes Network of Chang'an-Tianshan Corridor", 'Cultural')
('Asia', 'Eastern Asia', 'China', 'Site of Xanadu', 'Cultural')
('Asia', 'Eastern Asia', 'China', 'South China Karst', 'Natural')
('Asia', 'Eastern Asia', 'China', 'Summer Palace, an Imperial Garden in Beijing', 'Cultural')
('Asia', 'Eastern Asia', 'China', 'Temple and Cemetery of Confucius and the Kong Family Mansion in Qufu', 'Cultural')
('Asia', 'Eastern Asia', 'China', 'Temple of Heaven: an Imperial Sacrificial Altar in Beijing', 'Cultural')
('Asia', 'Eastern Asia', 'China', 'The Grand Canal', 'Cultural')
('Asia', 'Eastern Asia', 'China', 'The Great Wall', 'Cultural')
('Asia', 'Eastern Asia', 'China', 'Three Parallel Rivers of Yunnan Protected Areas', 'Natural')
('Asia', 'Eastern Asia', 'China', 'Tusi Sites', 'Cultural')
('Asia', 'Eastern Asia', 'China', 'West Lake Cultural Landscape of Hangzhou', 'Cultural')
('Asia', 'Eastern Asia', 'China', 'Wulingyuan Scenic and Historic Interest Area', 'Natural')
('Asia', 'Eastern Asia', 'China', 'Xinjiang Tianshan', 'Natural')
('Asia', 'Eastern Asia', 'China', 'Yin Xu', 'Cultural')
('Asia', 'Eastern Asia', 'China', 'Yungang Grottoes', 'Cultural')
('Asia', 'Eastern Asia', 'China', 'Zuojiang Huashan Rock Art Cultural Landscape', 'Cultural')
>>> ca8.count()
53
>>> str(hs.query)
'SELECT `region`.`region_name`, `sub_region`.`sub_region_name`, `country_area`.`country_area_name`, `heritage_site`.`site_name`, `heritage_site_category`.`category_name` FROM `heritage_site` INNER JOIN `heritage_site_jurisdiction` ON (`heritage_site`.`heritage_site_id` = `heritage_site_jurisdiction`.`heritage_site_id`) INNER JOIN `country_area` ON (`heritage_site_jurisdiction`.`country_area_id` = `country_area`.`country_area_id`) LEFT OUTER JOIN `region` ON (`country_area`.`region_id` = `region`.`region_id`) LEFT OUTER JOIN `sub_region` ON (`country_area`.`sub_region_id` = `sub_region`.`sub_region_id`) INNER JOIN `heritage_site_category` ON (`heritage_site`.`heritage_site_category_id` = `heritage_site_category`.`category_id`) WHERE `country_area`.`country_area_name` LIKE BINARY China% ORDER BY `heritage_site`.`site_name` ASC'
```

### Add a values_list() and extract values from tuples
```commandline
>>> hs = HeritageSite.objects.select_related('heritage_site_category').filter(country_area__country_area_name__startswith = 'China').values_list('country_area__region__region_name', 'country_area__sub_region__sub_region_name', 'country_area__country_area_name', 'site_name', 'heritage_site_category__category_name')
>>> hs.count()
53
>>> for s in hs:
...     print(s[0], s[1], s[2], s[3], s[4])
...
...
Asia Eastern Asia China Ancient Building Complex in the Wudang Mountains Cultural
Asia Eastern Asia China Ancient City of Ping Yao Cultural
Asia Eastern Asia China Ancient Villages in Southern Anhui – Xidi and Hongcun Cultural
Asia Eastern Asia China Capital Cities and Tombs of the Ancient Koguryo Kingdom Cultural
Asia Eastern Asia China Chengjiang Fossil Site Natural
Asia Eastern Asia China China Danxia Natural
Asia Eastern Asia China Classical Gardens of Suzhou Cultural
Asia Eastern Asia China Cultural Landscape of Honghe Hani Rice Terraces  Cultural
Asia Eastern Asia China Dazu Rock Carvings Cultural
Asia Eastern Asia China Fanjingshan Natural
Asia Eastern Asia China Fujian <em>Tulou</em> Cultural
Asia Eastern Asia China Historic Centre of Macao Cultural
Asia Eastern Asia China Historic Ensemble of the Potala Palace, Lhasa Cultural
Asia Eastern Asia China Historic Monuments of Dengfeng in “The Centre of Heaven and Earth” Cultural
Asia Eastern Asia China Huanglong Scenic and Historic Interest Area Natural
Asia Eastern Asia China Hubei Shennongjia Natural
Asia Eastern Asia China Imperial Palaces of the Ming and Qing Dynasties in Beijing and Shenyang Cultural
Asia Eastern Asia China Imperial Tombs of the Ming and Qing Dynasties Cultural
Asia Eastern Asia China Jiuzhaigou Valley Scenic and Historic Interest Area Natural
Asia Eastern Asia China Kaiping Diaolou and Villages Cultural
Asia Eastern Asia China Kulangsu, a Historic International Settlement Cultural
Asia Eastern Asia China Longmen Grottoes Cultural
Asia Eastern Asia China Lushan National Park Cultural
Asia Eastern Asia China Mausoleum of the First Qin Emperor Cultural
Asia Eastern Asia China Mogao Caves Cultural
Asia Eastern Asia China Mount Emei Scenic Area, including Leshan Giant Buddha Scenic Area Mixed
Asia Eastern Asia China Mount Huangshan Mixed
Asia Eastern Asia China Mount Qingcheng and the Dujiangyan Irrigation System Cultural
Asia Eastern Asia China Mount Sanqingshan National Park Natural
Asia Eastern Asia China Mount Taishan Mixed
Asia Eastern Asia China Mount Wutai Cultural
Asia Eastern Asia China Mount Wuyi Mixed
Asia Eastern Asia China Mountain Resort and its Outlying Temples, Chengde Cultural
Asia Eastern Asia China Old Town of Lijiang Cultural
Asia Eastern Asia China Peking Man Site at Zhoukoudian Cultural
Asia Eastern Asia China Qinghai Hoh Xil Natural
Asia Eastern Asia China Sichuan Giant Panda Sanctuaries - Wolong, Mt Siguniang and Jiajin Mountains  Natural
Asia Eastern Asia China Silk Roads: the Routes Network of Chang'an-Tianshan Corridor Cultural
Asia Eastern Asia China Site of Xanadu Cultural
Asia Eastern Asia China South China Karst Natural
Asia Eastern Asia China Summer Palace, an Imperial Garden in Beijing Cultural
Asia Eastern Asia China Temple and Cemetery of Confucius and the Kong Family Mansion in Qufu Cultural
Asia Eastern Asia China Temple of Heaven: an Imperial Sacrificial Altar in Beijing Cultural
Asia Eastern Asia China The Grand Canal Cultural
Asia Eastern Asia China The Great Wall Cultural
Asia Eastern Asia China Three Parallel Rivers of Yunnan Protected Areas Natural
Asia Eastern Asia China Tusi Sites Cultural
Asia Eastern Asia China West Lake Cultural Landscape of Hangzhou Cultural
Asia Eastern Asia China Wulingyuan Scenic and Historic Interest Area Natural
Asia Eastern Asia China Xinjiang Tianshan Natural
Asia Eastern Asia China Yin Xu Cultural
Asia Eastern Asia China Yungang Grottoes Cultural
Asia Eastern Asia China Zuojiang Huashan Rock Art Cultural Landscape Cultural
```





