# Meeting 3 Exercise

## 1.0 Data sets
Using MySQL Workbench you will create a logical data model that combines two United Nations data sets.

### 1.1 United Nations Statistical Division (UNSD) M49 Standard
The UNSD maintains a standard set of region, area and country codes referenced as the [M49](https://unstats.un.org/unsd/methodology/m49/) Standard. A copy of the data set in *.csv and *.xlsx formats are available locally for your use.

[un_area_country_codes_m49.csv](https://umich.instructure.com/courses/245664/files/8281767/download?verifier=eflNfPn2ZRRPpMBSG7MjUGYBWgGu6TGqebbj2oAe&wrap=1)

[un_area_country_codes_M49.tsv](https://umich.instructure.com/courses/245664/files/8592904/download?verifier=DLNCaOWstOrpjix20g7MmgNTXWqoCrfd9osScFC7&wrap=1)

[un_area_country_codes_M49.xlsx](https://umich.instructure.com/courses/245664/files/8578163/download?verifier=5ZJDXrcSBokwsDGEGoqPZTSgZBcwAf98tKZ4ZqAh&wrap=1)

### 1.2 United Nations Educational, Scientific, and Cultural Organization (UNESCO) Heritage Sites
UNESCO has designated over 1000 [heritage sites](https://whc.unesco.org/en/list/) located 
throughout the world as "cultural and natural treasures." A copy of the data set in `*.csv` and `*.xlsx` formats are available locally for your use.

[unesco_heritage_sites.csv](https://umich.instructure.com/courses/245664/files/8283366/download?verifier=FKYIzBpfmkqGj5DEat4b4cZ6LtlhZJYvgZMVCxC4&wrap=1)

[unesco_heritage_sites.tsv](https://umich.instructure.com/courses/245664/files/8592909/download?verifier=ZRNrOXJRnuKuarx3E0RZ0tNe4H7G0PBzX6Yh9u9i&wrap=1)

[unesco_heritage_sites.xlsx](https://umich.instructure.com/courses/245664/files/8578165/download?verifier=QA5P2Z5w6PiFSlDAjsdSm6WyPzO5zbyDjKUjGUyJ&wrap=1)

## 2.0 Exercise
Download the files and peruse their contents.  Then use the MySQL Workbench EER diagraming tool to create a logical data model for a "unesco_heritage_sites" database that combines the information contained in both data sets.

The design of your logical data model must reflect a data normalization strategy that adheres to the requirements imposed by third normal form (3NF). Across the two data sets identify candidate entities and establish entity relationships in terms of their cardinality (e.g. one-to-many, many-to-many). Ensure that data you identify as an attribute of a particular entity (i.e., a property) express a fact about the entity and no other entity. The goal is a logical data model that eliminates data duplication and ensures referential integrity.

## 3.0 Columns of interest
For this exercise, the following columns are relevant. You can ignore all others.

### 3.1 UNSD M49
* global_name
* region_name
* sub_region_name
* intermediate_region_name
* country_area_name
* country_area_m49_code
* country_area_iso_alpha3_code
* country_area_development_status

### 3.2 UNESCO Heritage Sites
* site_name
* description
* justification
* date_inscribed
* longitude
* latitude
* area_hectares
* category
* country_area
* iso_code
* udnp_code
* transboundary

## 4.0 Gotchas
The UNSD M49 data set includes one potential gotcha that may impact your design: the Antarctica entry.  The UNESCO data set includes several entries that will definitely impact how you design your data model.  Note in particular the following entries: "Old City of Jerusalem and its Walls ", and heritage sites coded with a transboundary = 1 (i.e., the site crosses national boundaries).

### 4.1 Naming conventions
Adhere to Simon Holywell's [SQL Style Guide](https://www.sqlstyle.guide/) recommendations when choosing names for tables, columns, etc. 

## File Submission
Name your EER diagram `<uniqname>_unesco_heritage_sites.mwb` (e.g., 
`arwhyte_unesco_heritage_sites.mwb`).  When submit your file to Canvas.



























