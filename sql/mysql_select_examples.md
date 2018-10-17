# MySQL SELECT Statement Examples

The following examples represent the type of SQL `SELECT` statements you may be asked to write as
 part of a weekly assignment or during the midterm examination.   




## TODO
IFNULL()
Correlated subquery






## MySQL Documentation
* [SELECT syntax](https://dev.mysql.com/doc/refman/8.0/en/select.html)
* [Subquery syntax](https://dev.mysql.com/doc/refman/8.0/en/subqueries.html)
* [Operators](https://dev.mysql.com/doc/refman/8.0/en/non-typed-operators.html)
* [String functions](https://dev.mysql.com/doc/refman/8.0/en/string-functions.html)
* [Numeric Functions and Operators](https://dev.mysql.com/doc/refman/8.0/en/numeric-functions.html)
* [Cast Functions and Operators](https://dev.mysql.com/doc/refman/8.0/en/cast-functions.html)


## MySQL shell
When using the MYSQL command line shell be sure to select the `unesco_heritage_sites` database 
before attempting to execute the example queries.

```mysql
USE unesco_heritage_sites;
``` 

## SQL: Return a list of heritage sites that includes the word 'Lake' in the name

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

## SQL: Return British, French, German and Dutch UNESCO heritage sites inscribed between 2010-2018

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

## SQL: Return a list of heritage site counts by region and subregion

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
This query requires use of an aggregate function together with a `GROUP BY` clause in order to group the result set counts by region and subregion. The SQL statement highlights use of the following syntax:

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

## SQL: Return a list of India's UNESCO heritage sites

#### Required column names (aliased)
* region
* subregion
* country / area
* heritage site
* category

#### Filter
* country / area name = 'India'

#### Sort order 
* heritage site name (ASC)

#### Query
This query requires a number of table joins in order to return country / area, subregion 
and region names. The SQL statement highlights use of the following syntax:

* `AS` -- permits the aliasing of column names.
* `LEFT JOIN` -- returns all records in table X that have a matching record in table Y as well as
 all records in table X that have no matching record in table Y.
* `TRIM(value)` -- removes leading as well as trailing whitespace in a string. Ensures that string 
comparisons are not disrupted by the presence of whitespace.

```mysql
SELECT r.region_name AS `region`, sr.sub_region_name AS `subregion`, 
       ca.country_area_name AS `country / area`, hs.site_name AS `heritage site`, 
       hsc.category_name AS `category`
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
       LEFT JOIN heritage_site_category hsc
              ON hs.heritage_site_category_id = hsc.category_id     
 WHERE TRIM(ca.country_area_name) = 'India'
 ORDER BY hs.site_name;
```

#### Result set
```commandline
+--------+---------------+--------------+--------------------------------------------------------------------------------------------+----------+
| region | subregion     | country/area | heritage site                                                                              | category |
+--------+---------------+--------------+--------------------------------------------------------------------------------------------+----------+
| Asia   | Southern Asia | India        | Agra Fort                                                                                  | Cultural |
| Asia   | Southern Asia | India        | Ajanta Caves                                                                               | Cultural |
| Asia   | Southern Asia | India        | Archaeological Site of Nalanda <i>Mahavihara</i> at Nalanda, Bihar                         | Cultural |
| Asia   | Southern Asia | India        | Buddhist Monuments at Sanchi                                                               | Cultural |
| Asia   | Southern Asia | India        | Champaner-Pavagadh Archaeological Park                                                     | Cultural |
| Asia   | Southern Asia | India        | Chhatrapati Shivaji Terminus (formerly Victoria Terminus)                                  | Cultural |
| Asia   | Southern Asia | India        | Churches and Convents of Goa                                                               | Cultural |
| Asia   | Southern Asia | India        | Elephanta Caves                                                                            | Cultural |
| Asia   | Southern Asia | India        | Ellora Caves                                                                               | Cultural |
| Asia   | Southern Asia | India        | Fatehpur Sikri                                                                             | Cultural |
| Asia   | Southern Asia | India        | Great Himalayan National Park Conservation Area                                            | Natural  |
| Asia   | Southern Asia | India        | Great Living Chola Temples                                                                 | Cultural |
| Asia   | Southern Asia | India        | Group of Monuments at Hampi                                                                | Cultural |
| Asia   | Southern Asia | India        | Group of Monuments at Mahabalipuram                                                        | Cultural |
| Asia   | Southern Asia | India        | Group of Monuments at Pattadakal                                                           | Cultural |
| Asia   | Southern Asia | India        | Hill Forts of Rajasthan                                                                    | Cultural |
| Asia   | Southern Asia | India        | Historic City of Ahmadabad                                                                 | Cultural |
| Asia   | Southern Asia | India        | Humayun's Tomb, Delhi                                                                      | Cultural |
| Asia   | Southern Asia | India        | Kaziranga National Park                                                                    | Natural  |
| Asia   | Southern Asia | India        | Keoladeo National Park                                                                     | Natural  |
| Asia   | Southern Asia | India        | Khajuraho Group of Monuments                                                               | Cultural |
| Asia   | Southern Asia | India        | Khangchendzonga National Park                                                              | Mixed    |
| Asia   | Southern Asia | India        | Mahabodhi Temple Complex at Bodh Gaya                                                      | Cultural |
| Asia   | Southern Asia | India        | Manas Wildlife Sanctuary                                                                   | Natural  |
| Asia   | Southern Asia | India        | Mountain Railways of India                                                                 | Cultural |
| Asia   | Southern Asia | India        | Nanda Devi and Valley of Flowers National Parks                                            | Natural  |
| Asia   | Southern Asia | India        | Qutb Minar and its Monuments, Delhi                                                        | Cultural |
| Asia   | Southern Asia | India        | Rani-ki-Vav (the Queen’s Stepwell) at Patan, Gujarat                                       | Cultural |
| Asia   | Southern Asia | India        | Red Fort Complex                                                                           | Cultural |
| Asia   | Southern Asia | India        | Rock Shelters of Bhimbetka                                                                 | Cultural |
| Asia   | Southern Asia | India        | Sun Temple, Konârak                                                                        | Cultural |
| Asia   | Southern Asia | India        | Sundarbans National Park                                                                   | Natural  |
| Asia   | Southern Asia | India        | Taj Mahal                                                                                  | Cultural |
| Asia   | Southern Asia | India        | The Architectural Work of Le Corbusier, an Outstanding Contribution to the Modern Movement | Cultural |
| Asia   | Southern Asia | India        | The Jantar Mantar, Jaipur                                                                  | Cultural |
| Asia   | Southern Asia | India        | Victorian Gothic and Art Deco Ensembles of Mumbai                                          | Cultural |
| Asia   | Southern Asia | India        | Western Ghats                                                                              | Natural  |
+--------+---------------+--------------+--------------------------------------------------------------------------------------------+----------+
37 rows in set (0.00 sec)
```

## SQL: Return a count of Indian UNESCO Heritage Sites by heritage site category, 

#### Required column names (aliased)
* region
* subregion
* country / area
* category

#### Filter
country / area name = 'India'

#### Sort order 
* category name (ASC)

#### Query
This query requires use of an aggregate function together with a `GROUP BY` clause in order to 
group the result set counts by region, subregion, country / area, and category. The SQL statement 
highlights use of the following syntax:

`COUNT(hsc.category_id)` -- count of the number of records returned by category. `COUNT(*)` is 
also acceptable.
`GROUP BY` -- follows the `WHERE` clause and groups the result set by region, subregion, country 
/ area, and category.

```mysql
SELECT r.region_name AS `region`, sr.sub_region_name AS `subregion`, 
       ca.country_area_name AS `country / area`, hsc.category_name AS `category`, 
       COUNT(hsc.category_id) AS `count`
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
       LEFT JOIN heritage_site_category hsc
              ON hs.heritage_site_category_id = hsc.category_id     
 WHERE TRIM(ca.country_area_name) = 'India'
 GROUP BY r.region_name, sr.sub_region_name, ca.country_area_name, hsc.category_name
 ORDER BY hsc.category_name;
```

#### Result set
```commandline
+--------+---------------+--------------+----------+-------+
| region | subregion     | country/area | category | count |
+--------+---------------+--------------+----------+-------+
| Asia   | Southern Asia | India        | Cultural |    29 |
| Asia   | Southern Asia | India        | Mixed    |     1 |
| Asia   | Southern Asia | India        | Natural  |     7 |
+--------+---------------+--------------+----------+-------+
3 rows in set (0.00 sec)
```

## SQL: Return the largest UNESCO heritage site by area (hectares) in the Caribbean.

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

## SQL: Return the total area (in hectares) per region that have been protected by the UNESCO 
heritage
 site designation

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

## SQL: Return a list of heritage sites, if any, that span regional boundaries?

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