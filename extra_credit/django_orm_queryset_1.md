# Extra credit: Django ORM queryset (75 points)

## 1.0 Queryset requirements
Construct a `QuerySet` of Northern European UNESCO heritage sites that are described as a 
"settlement" of ancient origin. *Exclude* all Roman sites. The `QuerySet` *must* be in the form of 
*distinct* site counts grouped by region, sub_region, country, and heritage site category. 
The `QuerySet` *must* also be optimized by use of the `.select_related()` method.  Use the `
.values()` method and `F` expressions to define the key/value pairs to be included in the `QuerySet`
 of dictionaries. Use `Q` objects in your `.filter()` method. The `.annotate()` method *must* 
 define a *distinct* site count. Order the `QuerySet` by site count *descending*, and then country.
 
## 2.0 Document your work
Construct the `QuerySet`and execute your Python code in the Django Python shell. Then do the 
following in the shell: 

* Print a `count()` of the `QuerySet` object. 
* Loop over the `QuerySet` iterable and print each dictionary. 

Copy and paste your `QuerySet` Python code, the count, and the list of dictionary items into a 
file named `<uniqname>-heritage_site_north_euro_non_roman_settlements.txt`. Submit the file for 
grading via Canvas.

### Points available
Up to 75 points will be awarded. Partial credit will be awarded at the discretion of the grader. 
Late entries are subject to the standard late penalty as described in the syllabus.

### Due date
Thursday, 13 December 2018 by 11:59 pm.