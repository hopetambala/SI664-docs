# SI664: MySQL SELECT Statement Examples

The following examples represent the types of SQL `SELECT` statements you may be asked to write 
as part of a weekly assignment or as part of an exam.

## 1.0 MySQL documentation
* [SELECT syntax](https://dev.mysql.com/doc/refman/8.0/en/select.html)
* [Subquery syntax](https://dev.mysql.com/doc/refman/8.0/en/subqueries.html)
* [Operators](https://dev.mysql.com/doc/refman/8.0/en/non-typed-operators.html)
* [String functions](https://dev.mysql.com/doc/refman/8.0/en/string-functions.html)
* [Numeric Functions and Operators](https://dev.mysql.com/doc/refman/8.0/en/numeric-functions.html)
* [Cast Functions and Operators](https://dev.mysql.com/doc/refman/8.0/en/cast-functions.html)


## 2.0 MySQL shell
When using the MYSQL command line shell be sure to select the `unesco_heritage_sites` database 
before attempting to execute the example queries.

```mysql
USE unesco_heritage_sites;
``` 

## 3.0 Example SQL statements
:warning: Actual assignment or exam questions that ask you to write a SQL statement may vary with respect to the number of `SELECT` columns required along with supporting table joins, aliases needed, `WHERE` or `HAVING` clause filter conditions, aggregate functions needed together with supporting `GROUP BY` clauses, subqueries, and sort order.

### 3.1 Return a list of India's UNESCO heritage sites

#### Required column names (aliased)
* country / area
* heritage site
* category

#### Filter
* country / area name = 'India'

#### Sort order 
* heritage site name (ASC)

#### Query
This query requires a number of table joins in order to return the country / area name, and the 
heritage site category name. The SQL statement highlights use of the following syntax:

* `AS` -- permits the aliasing of column names.
* `LEFT JOIN` -- returns all records in table X that have a matching record in table Y as well as
 all records in table X that have no matching record in table Y.
* `TRIM(value)` -- removes leading as well as trailing whitespace in a string. Ensures that string 
comparisons are not disrupted by the presence of whitespace.

```mysql
SELECT ca.country_area_name AS `country / area`, hs.site_name AS `heritage site`, 
       hsc.category_name AS `category`
  FROM heritage_site hs
       LEFT JOIN heritage_site_jurisdiction hsj 
              ON hs.heritage_site_id = hsj.heritage_site_id
       LEFT JOIN country_area ca 
              ON hsj.country_area_id = ca.country_area_id
       LEFT JOIN heritage_site_category hsc
              ON hs.heritage_site_category_id = hsc.category_id
 WHERE TRIM(ca.country_area_name) = 'India'
 ORDER BY hs.site_name;
```

#### Result set
```commandline
+----------------+--------------------------------------------------------------------------------------------+----------+
| country / area | heritage site                                                                              | category |
+----------------+--------------------------------------------------------------------------------------------+----------+
| India          | Agra Fort                                                                                  | Cultural |
| India          | Ajanta Caves                                                                               | Cultural |
| India          | Archaeological Site of Nalanda <i>Mahavihara</i> at Nalanda, Bihar                         | Cultural |
| India          | Buddhist Monuments at Sanchi                                                               | Cultural |
| India          | Champaner-Pavagadh Archaeological Park                                                     | Cultural |
| India          | Chhatrapati Shivaji Terminus (formerly Victoria Terminus)                                  | Cultural |
| India          | Churches and Convents of Goa                                                               | Cultural |
| India          | Elephanta Caves                                                                            | Cultural |
| India          | Ellora Caves                                                                               | Cultural |
| India          | Fatehpur Sikri                                                                             | Cultural |
| India          | Great Himalayan National Park Conservation Area                                            | Natural  |
| India          | Great Living Chola Temples                                                                 | Cultural |
| India          | Group of Monuments at Hampi                                                                | Cultural |
| India          | Group of Monuments at Mahabalipuram                                                        | Cultural |
| India          | Group of Monuments at Pattadakal                                                           | Cultural |
| India          | Hill Forts of Rajasthan                                                                    | Cultural |
| India          | Historic City of Ahmadabad                                                                 | Cultural |
| India          | Humayun's Tomb, Delhi                                                                      | Cultural |
| India          | Kaziranga National Park                                                                    | Natural  |
| India          | Keoladeo National Park                                                                     | Natural  |
| India          | Khajuraho Group of Monuments                                                               | Cultural |
| India          | Khangchendzonga National Park                                                              | Mixed    |
| India          | Mahabodhi Temple Complex at Bodh Gaya                                                      | Cultural |
| India          | Manas Wildlife Sanctuary                                                                   | Natural  |
| India          | Mountain Railways of India                                                                 | Cultural |
| India          | Nanda Devi and Valley of Flowers National Parks                                            | Natural  |
| India          | Qutb Minar and its Monuments, Delhi                                                        | Cultural |
| India          | Rani-ki-Vav (the Queen’s Stepwell) at Patan, Gujarat                                       | Cultural |
| India          | Red Fort Complex                                                                           | Cultural |
| India          | Rock Shelters of Bhimbetka                                                                 | Cultural |
| India          | Sun Temple, Konârak                                                                        | Cultural |
| India          | Sundarbans National Park                                                                   | Natural  |
| India          | Taj Mahal                                                                                  | Cultural |
| India          | The Architectural Work of Le Corbusier, an Outstanding Contribution to the Modern Movement | Cultural |
| India          | The Jantar Mantar, Jaipur                                                                  | Cultural |
| India          | Victorian Gothic and Art Deco Ensembles of Mumbai                                          | Cultural |
| India          | Western Ghats                                                                              | Natural  |
+----------------+--------------------------------------------------------------------------------------------+----------+
37 rows in set (0.01 sec)
```

### 3.2 Return a count of Indian UNESCO Heritage Sites by heritage site category

#### Required column names (aliased)
* country / area
* category
* heritage site count (COUNT(*))

#### Filter
country / area name = 'India'

#### Sort order 
* category name (ASC)

#### Query
This query requires use of an aggregate function together with a `GROUP BY` clause in order to 
group the result set counts by country / area, and category. The SQL statement highlights use of the following syntax:

* `COUNT(*)` -- count of the number of heritage sites returned by category.
* `GROUP BY` -- follows the `WHERE` clause and groups the result set by region, subregion, country / area, and category.

```mysql
SELECT ca.country_area_name AS `country / area`, hsc.category_name AS `category`, 
       COUNT(*) AS `heritage site count`
  FROM heritage_site hs
       LEFT JOIN heritage_site_jurisdiction hsj 
              ON hs.heritage_site_id = hsj.heritage_site_id
       LEFT JOIN country_area ca 
              ON hsj.country_area_id = ca.country_area_id
       LEFT JOIN heritage_site_category hsc
              ON hs.heritage_site_category_id = hsc.category_id     
 WHERE TRIM(ca.country_area_name) = 'India'
 GROUP BY ca.country_area_name, hsc.category_name
 ORDER BY hsc.category_name;
```

#### Result set
```commandline
+----------------+----------+---------------------+
| country / area | category | heritage site count |
+----------------+----------+---------------------+
| India          | Cultural |                  29 |
| India          | Mixed    |                   1 |
| India          | Natural  |                   7 |
+----------------+----------+---------------------+
3 rows in set (0.00 sec)
```

Given the many-to-many relationship that exists between heritage sites and countries / areas you should check whether or not any of the counts are inflated by the presence of sites that span more than one country / area.  Note the following adjustment:

* `COUNT(DISTINCT hs.heritage_site_id)` -- filters out duplicate rows produced by a heritage 
site located in more than a single country / area.

```mysql
SELECT ca.country_area_name AS `country / area`, hsc.category_name AS `category`, 
       COUNT(DISTINCT hs.heritage_site_id) AS `heritage site count`
  FROM heritage_site hs
       LEFT JOIN heritage_site_jurisdiction hsj 
              ON hs.heritage_site_id = hsj.heritage_site_id
       LEFT JOIN country_area ca 
              ON hsj.country_area_id = ca.country_area_id
       LEFT JOIN heritage_site_category hsc
              ON hs.heritage_site_category_id = hsc.category_id     
 WHERE TRIM(ca.country_area_name) = 'India'
 GROUP BY ca.country_area_name, hsc.category_name
 ORDER BY hsc.category_name;
```

In this case, the counts remain the same.  This cannot be guaranteed in other cases, however.

#### Result set
```commandline
+----------------+----------+---------------------+
| country / area | category | heritage site count |
+----------------+----------+---------------------+
| India          | Cultural |                  29 |
| India          | Mixed    |                   1 |
| India          | Natural  |                   7 |
+----------------+----------+---------------------+
3 rows in set (0.00 sec)
```

### 3.3 Return a list of heritage sites that includes the word 'Lake' in the name

#### Required column names (aliased)
* country / area
* heritage site
* category

#### Filter
* category = 'Natural' (i.e., `category_id` = 2)
* and substring 'Lake' in the heritage site name

#### Sort order 
* country / area name (ASC)
* heritage site name (ASC)

#### Query
This query retrieves all heritage sites that are categorized as "natural" and include the word 
"Lake" in their name.  The SQL statement highlights use of the following syntax:

* `INNER JOIN` -- returns all records in table X that have a matching record in table Y. Replacing each `INNER JOIN` with a `LEFT JOIN` returns the same results.
* `AND` -- an operator that requires that all conditions separated by `AND` return true
* `INSTR(string, substring)` -- returns the first occurrence of the substring in the string value.
* `'%\[value\]%'` -- use of wildcards in the search string (2nd query example).

```mysql
SELECT ca.country_area_name AS `country / area`, hs.site_name AS `heritage site`, 
       hsc.category_name AS `category`
  FROM heritage_site hs 
       INNER JOIN heritage_site_jurisdiction hsj 
              ON hs.heritage_site_id = hsj.heritage_site_id 
       INNER JOIN heritage_site_category hsc
              ON hs.heritage_site_category_id = hsc.category_id 
       INNER JOIN country_area ca 
              ON hsj.country_area_id = ca.country_area_id
WHERE hsc.category_id = 2 AND INSTR(hs.site_name, 'Lake') > 0
ORDER BY ca.country_area_name, hs.site_name;
```

The same result set can also be obtained with a `WHERE` clause that employs the `LIKE` operator 
and string wildcards (%):

```mysql
SELECT ca.country_area_name AS `country / area`, hs.site_name AS `heritage site`, 
       hsc.category_name AS `category`
  FROM heritage_site hs 
       INNER JOIN heritage_site_jurisdiction hsj 
              ON hs.heritage_site_id = hsj.heritage_site_id 
       INNER JOIN heritage_site_category hsc
              ON hs.heritage_site_category_id = hsc.category_id 
       INNER JOIN country_area ca 
              ON hsj.country_area_id = ca.country_area_id
WHERE hsc.category_id = 2 AND hs.site_name LIKE '%Lake%'
ORDER BY ca.country_area_name, hs.site_name;
````

#### Result set
```commandline
+--------------------+------------------------------------------------------+----------+
| country / area     | heritage site                                        | category |
+--------------------+------------------------------------------------------+----------+
| Chad               | Lakes of Ounianga                                    | Natural  |
| Croatia            | Plitvice Lakes National Park                         | Natural  |
| Kazakhstan         | Saryarka – Steppe and Lakes of Northern Kazakhstan   | Natural  |
| Kenya              | Kenya Lake System in the Great Rift Valley           | Natural  |
| Kenya              | Lake Turkana National Parks                          | Natural  |
| Malawi             | Lake Malawi National Park                            | Natural  |
| Russian Federation | Lake Baikal                                          | Natural  |
+--------------------+------------------------------------------------------+----------+
7 rows in set (0.00 sec)
```

### 3.4 Return list of castles, fortifications and walled cities and towns inscribed as UNESCO heritage sites

#### Required column names (aliased)
* country / area
* heritage site
* category

#### Filter
* category = 'Cultural'
* and heritage site name includes 'Castle', 'Citadel', 'Fort', 'Fortification', 'Fortress' or 
'Wall' in its name

#### Sort order 
* country / area name (ASC)
* heritage site name (ASC)

#### Query
This query retrieves all heritage sites that includes the word 'Castle', 'Citadel', 'Fort', 
'Fortification', 'Fortress' or 'Wall' in its name. The SQL statement highlights use of the following syntax: 

`AND` -- an operator that requires that all conditions separated by `AND` return true.
`OR` -- an operator that requires that any condition separated by `OR` return true.
`REGEXP` -- leverage regular expressions as a filter (e.g., akin to the regexp '/([a-zA-Z0-9])*Castle([a-zA-Z0-9])*/g').

:bulb:Implementing Full-text search would likely increase confidence that the result set returned 
includes all such sites.  

```mysql
SELECT ca.country_area_name AS `country / area`, hs.site_name AS `heritage site`, 
       hsc.category_name AS `category`
  FROM heritage_site hs
       LEFT JOIN heritage_site_jurisdiction hsj 
              ON hs.heritage_site_id = hsj.heritage_site_id
       LEFT JOIN country_area ca 
              ON hsj.country_area_id = ca.country_area_id
       LEFT JOIN heritage_site_category hsc
              ON hs.heritage_site_category_id = hsc.category_id
 WHERE hsc.category_name = 'Cultural' AND (hs.site_name LIKE '%Castle%' 
       OR hs.site_name LIKE '%Citadel%'
       OR hs.site_name LIKE '%Fort%' 
       OR hs.site_name LIKE '%Wall%')
 ORDER BY ca.country_area_name, hs.site_name;
```

```mysql
SELECT ca.country_area_name AS `country / area`, hs.site_name AS `heritage site`, 
       hsc.category_name AS `category`
  FROM heritage_site hs
       LEFT JOIN heritage_site_jurisdiction hsj 
              ON hs.heritage_site_id = hsj.heritage_site_id
       LEFT JOIN country_area ca 
              ON hsj.country_area_id = ca.country_area_id
       LEFT JOIN heritage_site_category hsc
              ON hs.heritage_site_category_id = hsc.category_id
 WHERE hsc.category_name = 'Cultural' AND hs.site_name REGEXP 'Castle|Citadel|Fort|Wall'
 ORDER BY ca.country_area_name, hs.site_name;
```

#### Result set
```commandline
+------------------------------------------------------+-------------------------------------------------------------------------------------------+----------+
| country / area                                       | heritage site                                                                             | category |
+------------------------------------------------------+-------------------------------------------------------------------------------------------+----------+
| NULL                                                 | Old City of Jerusalem and its Walls                                                       | Cultural |
| Azerbaijan                                           | Walled City of Baku with the Shirvanshah's Palace and Maiden Tower                        | Cultural |
| Belarus                                              | Mir Castle Complex                                                                        | Cultural |
| Belgium                                              | Major Mining Sites of Wallonia                                                            | Cultural |
| China                                                | The Great Wall                                                                            | Cultural |
| Colombia                                             | Port, Fortresses and Group of Monuments, Cartagena                                        | Cultural |
| Cuba                                                 | Old Havana and its Fortification System                                                   | Cultural |
| Cuba                                                 | San Pedro de la Roca Castle, Santiago de Cuba                                             | Cultural |
| Czechia                                              | Gardens and Castle at Kroměříž                                                            | Cultural |
| Czechia                                              | Litomyšl Castle                                                                           | Cultural |
| Denmark                                              | Kronborg Castle                                                                           | Cultural |
| Ethiopia                                             | Harar Jugol, the Fortified Historic Town                                                  | Cultural |
| Finland                                              | Fortress of Suomenlinna                                                                   | Cultural |
| France                                               | Fortifications of Vauban                                                                  | Cultural |
| France                                               | Historic Fortified City of Carcassonne                                                    | Cultural |
| Germany                                              | Castles of Augustusburg and Falkenlust at Brühl                                           | Cultural |
| Germany                                              | Collegiate Church, Castle and Old Town of Quedlinburg                                     | Cultural |
| Germany                                              | Wartburg Castle                                                                           | Cultural |
| Ghana                                                | Forts and Castles, Volta, Greater Accra, Central and Western Regions                      | Cultural |
| Haiti                                                | National History Park – Citadel, Sans Souci, Ramiers                                      | Cultural |
| Hungary                                              | Budapest, including the Banks of the Danube, the Buda Castle Quarter and Andrássy Avenue  | Cultural |
| India                                                | Agra Fort                                                                                 | Cultural |
| India                                                | Hill Forts of Rajasthan                                                                   | Cultural |
| India                                                | Red Fort Complex                                                                          | Cultural |
| Iraq                                                 | Erbil Citadel                                                                             | Cultural |
| Kenya                                                | Fort Jesus, Mombasa                                                                       | Cultural |
| Luxembourg                                           | City of Luxembourg: its Old Quarters and Fortifications                                   | Cultural |
| Mexico                                               | Historic Fortified Town of Campeche                                                       | Cultural |
| Oman                                                 | Bahla Fort                                                                                | Cultural |
| Pakistan                                             | Fort and Shalamar Gardens in Lahore                                                       | Cultural |
| Pakistan                                             | Rohtas Fort                                                                               | Cultural |
| Panama                                               | Fortifications on the Caribbean Side of Panama: Portobelo-San Lorenzo                     | Cultural |
| Poland                                               | Castle of the Teutonic Order in Malbork                                                   | Cultural |
| Portugal                                             | Garrison Border Town of Elvas and its Fortifications                                      | Cultural |
| Republic of Korea                                    | Hwaseong Fortress                                                                         | Cultural |
| Romania                                              | Dacian Fortresses of the Orastie Mountains                                                | Cultural |
| Romania                                              | Villages with Fortified Churches in Transylvania                                          | Cultural |
| Russian Federation                                   | Citadel, Ancient City and Fortress Buildings of Derbent                                   | Cultural |
| Saint Kitts and Nevis                                | Brimstone Hill Fortress National Park                                                     | Cultural |
| Spain                                                | Historic Walled Town of Cuenca                                                            | Cultural |
| Spain                                                | Roman Walls of Lugo                                                                       | Cultural |
| Sri Lanka                                            | Old Town of Galle and its Fortifications                                                  | Cultural |
| Switzerland                                          | Three Castles, Defensive Wall and Ramparts of the Market-Town of Bellinzona               | Cultural |
| Turkey                                               | Diyarbakır Fortress and Hevsel Gardens Cultural Landscape                                 | Cultural |
| Turkmenistan                                         | Parthian Fortresses of Nisa                                                               | Cultural |
| United Kingdom of Great Britain and Northern Ireland | Castles and Town Walls of King Edward in Gwynedd                                          | Cultural |
| United Kingdom of Great Britain and Northern Ireland | Cornwall and West Devon Mining Landscape                                                  | Cultural |
| United Kingdom of Great Britain and Northern Ireland | Durham Castle and Cathedral                                                               | Cultural |
| United Kingdom of Great Britain and Northern Ireland | Historic Town of St George and Related Fortifications, Bermuda                            | Cultural |
| United Kingdom of Great Britain and Northern Ireland | The Forth Bridge                                                                          | Cultural |
| United States of America                             | La Fortaleza and San Juan National Historic Site in Puerto Rico                           | Cultural |
| Viet Nam                                             | Central Sector of the Imperial Citadel of Thang Long - Hanoi                              | Cultural |
| Viet Nam                                             | Citadel of the Ho Dynasty                                                                 | Cultural |
| Yemen                                                | Old Walled City of Shibam                                                                 | Cultural |
+------------------------------------------------------+-------------------------------------------------------------------------------------------+----------+
54 rows in set (0.01 sec)
```

### 3.5 Return British, French, German and Dutch UNESCO heritage sites inscribed between 2010-2018

#### Required column names (aliased)
* country / area
* heritage site
* category
* date inscribed

#### Filter
* British, French, German and Dutch ISO Alpha 3 codes
* and year heritage site inscribed between 2010-2018.

#### Sort order 
* date inscribed (DESC)
* country / area name (ASC)
* heritage site name (ASC)

#### Query
This query retrieves all British, French, German, and Dutch heritage sites inscribed between the 
years 2010 and 2018.  The SQL statement highlights use of the following syntax:

* `REPLACE(value, replace, replace with)` -- cosmetic; substitutes the "UK" acronym for the 
official country name.
* `IN(value1, value2, ...)` -- an operator that permits multiple values to be referenced in the 
`WHERE` clause.
* `BETWEEN` -- an operator that selects values within a given range, including the begin and end 
values.

```mysql
SELECT REPLACE(ca.country_area_name, 'United Kingdom of Great Britain and Northern Ireland', 'UK') AS `country / area`, hs.site_name AS `heritage site`, hsc.category_name AS `category`, hs.date_inscribed AS `date inscribed`
  FROM heritage_site hs 
       INNER JOIN heritage_site_jurisdiction hsj 
              ON hs.heritage_site_id = hsj.heritage_site_id 
       INNER JOIN heritage_site_category hsc
              ON hs.heritage_site_category_id = hsc.category_id 
       INNER JOIN country_area ca 
              ON hsj.country_area_id = ca.country_area_id
WHERE ca.iso_alpha3_code IN ('DEU', 'FRA', 'GBR', 'NLD') AND hs.date_inscribed BETWEEN 2010 AND 2018
ORDER BY hs.date_inscribed DESC, ca.country_area_name, hs.site_name;
```

#### Result set
```commandline
+----------------+--------------------------------------------------------------------------------------------+----------+----------------+
| country / area | heritage site                                                                              | category | date inscribed |
+----------------+--------------------------------------------------------------------------------------------+----------+----------------+
| France         | Chaîne des Puys - Limagne fault tectonic arena                                             | Natural  |           2018 |
| Germany        | Archaeological Border complex of Hedeby and the Danevirke                                  | Cultural |           2018 |
| Germany        | Naumburg Cathedral                                                                         | Cultural |           2018 |
| France         | Taputapuātea                                                                               | Cultural |           2017 |
| Germany        | Caves and Ice Age Art in the Swabian Jura                                                  | Cultural |           2017 |
| UK             | The English Lake District                                                                  | Cultural |           2017 |
| France         | The Architectural Work of Le Corbusier, an Outstanding Contribution to the Modern Movement | Cultural |           2016 |
| Germany        | The Architectural Work of Le Corbusier, an Outstanding Contribution to the Modern Movement | Cultural |           2016 |
| UK             | Gorham's Cave Complex                                                                      | Cultural |           2016 |
| France         | Champagne Hillsides, Houses and Cellars                                                    | Cultural |           2015 |
| France         | The Climats, terroirs of Burgundy                                                          | Cultural |           2015 |
| Germany        | Speicherstadt and Kontorhaus District with Chilehaus                                       | Cultural |           2015 |
| UK             | The Forth Bridge                                                                           | Cultural |           2015 |
| France         | Decorated Cave of Pont d’Arc, known as Grotte Chauvet-Pont d’Arc, Ardèche                  | Cultural |           2014 |
| Germany        | Carolingian Westwork and Civitas Corvey                                                    | Cultural |           2014 |
| Netherlands    | Van Nellefabriek                                                                           | Cultural |           2014 |
| Germany        | Bergpark Wilhelmshöhe                                                                      | Cultural |           2013 |
| France         | Nord-Pas de Calais Mining Basin                                                            | Cultural |           2012 |
| Germany        | Margravial Opera House Bayreuth                                                            | Cultural |           2012 |
| France         | Prehistoric Pile Dwellings around the Alps                                                 | Cultural |           2011 |
| France         | The Causses and the Cévennes, Mediterranean agro-pastoral Cultural Landscape               | Cultural |           2011 |
| Germany        | Fagus Factory in Alfeld                                                                    | Cultural |           2011 |
| Germany        | Prehistoric Pile Dwellings around the Alps                                                 | Cultural |           2011 |
| France         | Episcopal City of Albi                                                                     | Cultural |           2010 |
| France         | Pitons, cirques and remparts of Reunion Island                                             | Natural  |           2010 |
| Netherlands    | Seventeenth-Century Canal Ring Area of Amsterdam inside the Singelgracht                   | Cultural |           2010 |
+----------------+--------------------------------------------------------------------------------------------+----------+----------------+
26 rows in set (0.00 sec)
```

### 3.6 Return a list of heritage site counts by region and subregion

#### Required column names (aliased)
* region
* subregion
* heritage sites (COUNT(*))

#### Filter
* exclude Antarctica

#### Sort order 
* region name (ASC)
* subregion name (ASC)

#### Query
This query requires use of an aggregate function together with a `GROUP BY` clause in order to 
group the result set counts by region and subregion. The following SQL statement highlights use
 of the following syntax:

* `COUNT(*)` -- returns heritage site counts per region and sub region.  Given the many-to-many 
relationship that exists between heritage sites and countries / areas duplicate results cannot be
 ruled out.    
* `!=` -- `WHERE` clause condition that excludes Antarctica from the search results

```mysql
SELECT r.region_name AS `region`, sr.sub_region_name AS `subregion`, COUNT(*) AS `heritage sites`
  FROM heritage_site hs
       INNER JOIN heritage_site_jurisdiction hsj 
              ON hs.heritage_site_id = hsj.heritage_site_id
       INNER JOIN country_area ca 
              ON hsj.country_area_id = ca.country_area_id
       INNER JOIN location l
              ON ca.location_id = l.location_id
       INNER JOIN region r
              ON l.region_id = r.region_id
       INNER JOIN sub_region sr 
              ON l.sub_region_id = sr.sub_region_id
WHERE TRIM(ca.country_area_name) != 'Antarctica'
GROUP BY r.region_name, sr.sub_region_name
ORDER BY r.region_name, sr.sub_region_name;
```

#### Result set
```commandline
+----------+---------------------------------+----------------+
| region   | subregion                       | heritage sites |
+----------+---------------------------------+----------------+
| Africa   | Northern Africa                 |             39 |
| Africa   | Sub-Saharan Africa              |            105 |
| Americas | Latin America and the Caribbean |            149 |
| Americas | Northern America                |             42 |
| Asia     | Central Asia                    |             18 |
| Asia     | Eastern Asia                    |             95 |
| Asia     | South-eastern Asia              |             38 |
| Asia     | Southern Asia                   |             83 |
| Asia     | Western Asia                    |             80 |
| Europe   | Eastern Europe                  |            100 |
| Europe   | Northern Europe                 |             83 |
| Europe   | Southern Europe                 |            171 |
| Europe   | Western Europe                  |            134 |
| Oceania  | Australia and New Zealand       |             22 |
| Oceania  | Melanesia                       |              4 |
| Oceania  | Micronesia                      |              4 |
+----------+---------------------------------+----------------+
16 rows in set (0.01 sec)
```

As was noted earlier, given the many-to-many relationship that exists between heritage sites and countries / areas you should check whether or not any of the counts are inflated by the presence of sites that span more than one country / area.  Note the following adjustment:

* `COUNT(DISTINCT hs.heritage_site_id)` -- filters out duplicate rows produced by a heritage 
site located in more than a single country / area.

```mysql
SELECT r.region_name AS `region`, sr.sub_region_name AS `subregion`, COUNT(DISTINCT hs.heritage_site_id) AS `heritage sites`
  FROM heritage_site hs
       INNER JOIN heritage_site_jurisdiction hsj 
              ON hs.heritage_site_id = hsj.heritage_site_id
       INNER JOIN country_area ca 
              ON hsj.country_area_id = ca.country_area_id
       INNER JOIN location l
              ON ca.location_id = l.location_id
       INNER JOIN region r
              ON l.region_id = r.region_id
       INNER JOIN sub_region sr 
              ON l.sub_region_id = sr.sub_region_id
WHERE TRIM(ca.country_area_name) != 'Antarctica'
GROUP BY r.region_name, sr.sub_region_name
ORDER BY r.region_name, sr.sub_region_name;
```

#### Result set
```commandline
+----------+---------------------------------+----------------+
| region   | subregion                       | heritage sites |
+----------+---------------------------------+----------------+
| Africa   | Northern Africa                 |             39 |
| Africa   | Sub-Saharan Africa              |             97 |
| Americas | Latin America and the Caribbean |            142 |
| Americas | Northern America                |             40 |
| Asia     | Central Asia                    |             15 |
| Asia     | Eastern Asia                    |             95 |
| Asia     | South-eastern Asia              |             38 |
| Asia     | Southern Asia                   |             83 |
| Asia     | Western Asia                    |             80 |
| Europe   | Eastern Europe                  |             91 |
| Europe   | Northern Europe                 |             77 |
| Europe   | Southern Europe                 |            158 |
| Europe   | Western Europe                  |            124 |
| Oceania  | Australia and New Zealand       |             22 |
| Oceania  | Melanesia                       |              4 |
| Oceania  | Micronesia                      |              4 |
+----------+---------------------------------+----------------+
16 rows in set (0.01 sec)
```

As the result set above illustrates, in certain cases, such as Southern Europe, the counts fall 
significantly.

### 3.7 Return the largest UNESCO heritage site by area (hectares) in the Caribbean

#### Required column names (aliased)
* region
* subregion
* intermediate region
* country / area
* heritage site
* area (hectares)

#### Filter
* intermediate region name =  'Caribbean'
* and area (hecatares) = largest heritage site in the Caribbean

#### Query
This query uses a subquery that employs a `Max()` aggregate function to help return a result set 
containing the largest heritage site in the Caribbean. The SQL statement highlights use of the following syntax:

* `WHERE` -- includes a subquery for filtering out all records except the largest heritage.
 site in the Caribbean
* `MAX(value)` -- returns the maximum value, in this case, area (hectares).
* `\G` -- terminator that informs MySQL to print the result set in vertical output form.

```mysql
SELECT r.region_name AS `region`, sr.sub_region_name AS `subregion`,  
       ir.intermediate_region_name AS `intermediate region`, 
       ca.country_area_name AS `country / area`, hs.site_name AS `heritage site`, 
       hs.area_hectares AS `area (hectares)` 
  FROM heritage_site hs 
       LEFT JOIN heritage_site_jurisdiction hsj 
              ON hs.heritage_site_id = hsj.heritage_site_id 
       LEFT JOIN country_area ca 
              ON hsj.country_area_id = ca.country_area_id 
       LEFT JOIN location l 
              ON ca.location_id = l.location_id 
       LEFT JOIN region r 
              ON l.region_id = r.region_id 
       LEFT JOIN sub_region sr 
              ON l.sub_region_id = sr.sub_region_id 
       LEFT JOIN intermediate_region ir
              ON l.intermediate_region_id = ir.intermediate_region_id
 WHERE TRIM(ir.intermediate_region_name) = 'Caribbean' 
       AND hs.area_hectares = (SELECT MAX(hs1.area_hectares) 
                                 FROM heritage_site hs1 
                                      LEFT JOIN heritage_site_jurisdiction hsj1 
                                             ON hs1.heritage_site_id = hsj1.heritage_site_id 
                                      LEFT JOIN country_area ca1 
                                             ON hsj1.country_area_id = ca1.country_area_id 
                                      LEFT JOIN location l1 
                                             ON ca1.location_id = l1.location_id
                                      LEFT JOIN intermediate_region ir1
                                             ON l1.intermediate_region_id = ir1.intermediate_region_id
                                WHERE TRIM(ir1.intermediate_region_name) = 'Caribbean')\G
```

The following query also returns the same Cuban record.  It chooses a sort strategy that imposes 
a row count limit of 1 after sorting the area values in descending order.  This query is slower 
than using the `MAX()` subquery to filter on area. 

```mysql
SELECT r.region_name AS `region`, sr.sub_region_name AS `subregion`,  
       ir.intermediate_region_name AS `intermediate region`, 
       ca.country_area_name AS `country / area`, hs.site_name AS `heritage site`, 
       hs.area_hectares AS `area (hectares)` 
  FROM heritage_site hs 
       LEFT JOIN heritage_site_jurisdiction hsj 
              ON hs.heritage_site_id = hsj.heritage_site_id 
       LEFT JOIN country_area ca 
              ON hsj.country_area_id = ca.country_area_id 
       LEFT JOIN location l 
              ON ca.location_id = l.location_id 
       LEFT JOIN region r 
              ON l.region_id = r.region_id 
       LEFT JOIN sub_region sr 
              ON l.sub_region_id = sr.sub_region_id 
       LEFT JOIN intermediate_region ir
              ON l.intermediate_region_id = ir.intermediate_region_id
 WHERE TRIM(ir.intermediate_region_name) = 'Caribbean' 
 ORDER BY hs.area_hectares DESC LIMIT 1;
```

#### Result set
```commandline
*************************** 1. row ***************************
            region: Americas
         subregion: Latin America and the Caribbean
intermediateregion: Caribbean
    country / area: Cuba
     heritage site: Archaeological Landscape of the First Coffee Plantations in the South-East of Cuba
   area (hectares): 81475
1 row in set (0.02 sec)
```

### 3.8 Return the total area (in hectares) per region that have been protected by the UNESCO heritage site designation

#### Required column names (aliased)
* region
* area (hectares)

#### Filter
* exclude Antarctica

#### Sort order 
* area (hectares) (DESC)

#### Query
This query requires use of a number of functions to ensure that the result set is returned in 
the proper order.  The SQL statement highlights use of the following syntax:

* `SUM()` -- aggregate function that, in combination with the `GROUP BY`, provides the total area of 
heritage sites located in a given region.
* `CAST(value AS datatype)` -- casts area_hectares (a `DOUBLE`) as a `DECIMAL` datatype in order to
 ensure that the `ORDER BY` returns the result set in descending order (`FLOAT` and `DOUBLE` lack of precision can confound proper ordering).
* `ROUND(number, decimals)` -- rounds a number to a specified number of decimal places (in this case 
zero places).
* `WHERE` -- filters out `Antarctica` (`country_area_id` = 9) which is not assigned a regional 
affiliation by the UNSD.
* `GROUP BY` -- required in order to group `r.region_name` by the results of the aggregate function employed in the `SELECT` clause.
* `ORDER BY` -- note that you can reference aliases.


```mysql
SELECT r.region_name AS 'region', 
       ROUND(CAST(SUM(hs.area_hectares) AS DECIMAL(10,1))) AS `area (hectares)`
  FROM heritage_site hs
       LEFT JOIN heritage_site_jurisdiction hsj
              ON hs.heritage_site_id = hsj.heritage_site_id
       LEFT JOIN country_area ca
              ON hsj.country_area_id = ca.country_area_id
       LEFT JOIN location l
              ON ca.location_id = l.location_id
       LEFT JOIN region r
              ON l.region_id = r.region_id
 WHERE TRIM(ca.country_area_name) != 'Antarctica' 
 GROUP BY r.region_name
 ORDER BY `area (hectares)` DESC;
```

#### Result set
```commandline
+----------+---------------+
| region   | area_hectares |
+----------+---------------+
| Americas |     112117134 |
| Oceania  |      89583502 |
| Africa   |      60419026 |
| Europe   |      34578501 |
| Asia     |      23761530 |
+----------+---------------+
5 rows in set (0.01 sec)
```

## 3.9 Return a list of heritage sites, if any, that span regional boundaries

#### Required column names (aliased)
* `heritage site`
* `regions` (`GROUP_CONCAT()`)
* `region count`

#### Filter
* region count > 1

#### Sort order 
* region count (DESC)

#### Query
The `heritage_site` table includes a `transboundary` property that provides a shortcut to 
determining the number of UNESCO heritage sites that cross national boundaries.

```mysql
SELECT COUNT(*) 
  FROM heritage_site 
 WHERE transboundary = 1;
```

```commandline
+----------+
| COUNT(*) |
+----------+
|       37 |
+----------+
1 row in set (0.00 sec)
```

If heritage sites can span national boundaries do any heritage sites also span regional boundaries? 
 In other words, will the `heritagesites` app need to display multiple regional affiliations for 
 individual heritage sites? The query below supplies the answer. Three heritage sites span not only national boundaries but also UNSD regional boundaries. The SQL statement highlights use of the following syntax:

* `COUNT(DISTINCT r.region_id)` -- filters out duplicate rows produced by a heritage site located
in more than a single country / area. Le Corbusier's architectural works (`heritage_site_id` = 
1064), which are situated in many countries across Asia, Europe and the Americas is one such 
example.
* `INNER JOIN` -- returns all records in table X that have a matching record in table Y. Replacing each `INNER JOIN` with a `LEFT JOIN` returns the same results.  Replacement with a 
`RIGHT JOIN` does not.
* `GROUP BY` -- required in order to group `hs.site_name` by the results of the aggregate 
function employed in the `SELECT` clause.
* `HAVING` -- acts as a filter on the `GROUP BY` clause, eliminating rows that do not meet the 
condition specified
* `GROUP_CONCAT()` -- entirely decorative but handy nonetheless since it augments the raw counts 
with a list of the regions associated with each site. 

```mysql
   SELECT hs.site_name AS `heritage site`, 
          GROUP_CONCAT(DISTINCT r.region_name ORDER BY r.region_name SEPARATOR ', ') AS `regions`, 
          COUNT(DISTINCT r.region_id) as `region count`
     FROM heritage_site hs
          INNER JOIN heritage_site_jurisdiction hsj
                  ON hs.heritage_site_id = hsj.heritage_site_id
          INNER JOIN country_area ca
                  ON hsj.country_area_id = ca.country_area_id
          INNER JOIN location l
                  ON ca.location_id = l.location_id
          INNER JOIN region r
                  ON l.region_id = r.region_id
    GROUP BY hs.site_name
   HAVING COUNT(DISTINCT r.region_id) > 1
    ORDER BY `region count` DESC;
```

#### Result set
```commandline
+--------------------------------------------------------------------------------------------+------------------------+--------------+
| heritage site                                                                              | regions                | region count |
+--------------------------------------------------------------------------------------------+------------------------+--------------+
| The Architectural Work of Le Corbusier, an Outstanding Contribution to the Modern Movement | Americas, Asia, Europe |            3 |
| Landscapes of Dauria                                                                       | Asia, Europe           |            2 |
| Uvs Nuur Basin                                                                             | Asia, Europe           |            2 |
+--------------------------------------------------------------------------------------------+------------------------+--------------+
3 rows in set (0.01 sec)
```

## Appendix A. Retrieving database and table information
It can prove helpful at times to retrieve information about the database, tables, and columns 
that are the target of your queries.  MySQL provides several useful SQL statements for 
retrieving such information.

### A.1 Database currently selected

```mysql
SELECT DATABASE();
```

```commandline
+-----------------------+
| DATABASE()            |
+-----------------------+
| unesco_heritage_sites |
+-----------------------+
1 row in set (0.01 sec)
```

### A.2. List of database tables

```mysql
SHOW TABLES;
```

```commandline
+---------------------------------+
| Tables_in_unesco_heritage_sites |
+---------------------------------+
| auth_group                      |
| auth_group_permissions          |
| auth_permission                 |
| auth_user                       |
| auth_user_groups                |
| auth_user_user_permissions      |
| country_area                    |
| dev_status                      |
| django_admin_log                |
| django_content_type             |
| django_migrations               |
| django_session                  |
| heritage_site                   |
| heritage_site_category          |
| heritage_site_jurisdiction      |
| intermediate_region             |
| location                        |
| planet                          |
| region                          |
| sub_region                      |
+---------------------------------+
20 rows in set (0.02 sec)
```

### A.3. Table information

```mysql
DESCRIBE heritage_site;
```

```commandline
+---------------------------+---------------+------+-----+---------+----------------+
| Field                     | Type          | Null | Key | Default | Extra          |
+---------------------------+---------------+------+-----+---------+----------------+
| heritage_site_id          | int(11)       | NO   | PRI | NULL    | auto_increment |
| site_name                 | varchar(255)  | NO   | UNI | NULL    |                |
| description               | text          | NO   |     | NULL    |                |
| justification             | text          | YES  |     | NULL    |                |
| date_inscribed            | year(4)       | YES  |     | NULL    |                |
| longitude                 | decimal(11,8) | YES  |     | NULL    |                |
| latitude                  | decimal(10,8) | YES  |     | NULL    |                |
| area_hectares             | double        | YES  |     | NULL    |                |
| heritage_site_category_id | int(11)       | NO   | MUL | NULL    |                |
| transboundary             | tinyint(4)    | NO   |     | NULL    |                |
+---------------------------+---------------+------+-----+---------+----------------+
10 rows in set (0.02 sec)
```