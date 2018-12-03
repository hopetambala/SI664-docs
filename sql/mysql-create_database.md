# MySQL creating databases using flat files

use the Python [Pandas](https://pandas.pydata.org/) 
library to generate a *.csv or *.tsv file of the required values

### 
Since 

```mysql
SET FOREIGN_KEY_CHECKS=0;
DROP TABLE IF EXISTS developer, game, game_developer, genre, platform,
                     publisher, rating, sale, region;
SET FOREIGN_KEY_CHECKS=1;
```

### Creating and populating tables with no foreign keys
Certain tables exist to provide "lookup" values for other tables. Creating and populating such 
tables is a straightforward process. If you only have a few rows to insert into a table (i.e., <= 20 rows) you can hard code the values in your SQL INSERT statement as in the example below:

```mysql
CREATE TABLE IF NOT EXISTS region (
  region_id INTEGER NOT NULL AUTO_INCREMENT UNIQUE,
  region_name VARCHAR(25) NOT NULL UNIQUE,
  PRIMARY KEY (region_id)
)
ENGINE=InnoDB
CHARACTER SET utf8mb4
COLLATE utf8mb4_0900_ai_ci;

INSERT IGNORE INTO region (region_name) VALUES
  ('Global'), ('North America'), ('Europe'), ('Japan'), ('Other');
```

If, on the other hand, you need to populate a lookup table with no foreign keys with a large 
number of rows consider using the Python [Pandas](https://pandas.pydata.org/) library to extract the column data out of your source file and then write it to a `csv` or `tsv` file. You can then call the file with your *.sql script and write the row data into the table using the [LOAD DATA LOCAL INFILE](https://dev.mysql.com/doc/refman/8.0/en/load-data.html) syntax.

```mysql
CREATE TABLE IF NOT EXISTS publisher (
  publisher_id INTEGER NOT NULL AUTO_INCREMENT UNIQUE,
  publisher_name VARCHAR(100) NOT NULL UNIQUE,
  PRIMARY KEY (publisher_id)
)
ENGINE=InnoDB
CHARACTER SET utf8mb4
COLLATE utf8mb4_0900_ai_ci;

LOAD DATA LOCAL INFILE './output/video_games/video_game_publishers.csv'
INTO TABLE publisher
  CHARACTER SET utf8mb4
  FIELDS TERMINATED BY ',' ENCLOSED BY '"'
  LINES TERMINATED BY '\n'
  (publisher_name);
```

In the example above the file `video_game_publishers.csv` provides a list of publisher names.  
The `LOAD DATA LOCAL INFILE` portion of the statement reads rows from the csv file into a target table column (e.g., `publisher_name`). 

The following row handling options are specified:

* `CHARACTER SET`
* `FIELDS TERMINATED BY`
* `ENCLOSED BY`
* `LINES TERMINATED BY`

Beyond specifying the incoming character set, both field and line terminators are specified with 
one or more escape sequence characters:

* `\n`: line feed (i.e., new line); macOS, Unix
* `\r`: carriage return
* `\r\n`: carriage return, line feed; MS Windows
* `\t`: tabulator (i.e., tab separator)

Specifying the `ENCLOSED BY` option is required when reading comma-delimited files that may 
include string values that contain one or more commas (',') as in the following example:

```
...
Deep Silver,
"Destination Software, Inc.",
Destineer,
...
```

:bulb: For macOS and Unix users you can identify the type of line breaks a text file contains with 
the 
`file` command (Windows requires installation of a Bash shell to gain this capability).

 ```commandline
$ file video_game_sales-20161222.csv
video_game_sales-20161222.csv: UTF-8 Unicode text, with CRLF line terminators
```

### Creating and populating tables with foreign keys
If a table includes one or more foreign keys the task of populating the table with one or more 
foreign keys that indicate a many-to-one relationship with another table along with other row 
data is usually a straightforward process.

The following steps should usually suffice:

Ensure that the tables from which the foreign key values will be drawn are created and 
populated with row data. In the example below `genre`, `platform`, `publisher` and `rating` 
tables are required.

Create a temporary table (e.g., `temp_game`) and load it with the flat file source data that 
specifies from one column to the next the entity relationships that you have chosen to model.

```mysql
CREATE TEMPORARY TABLE temp_game (
  game_id INTEGER NOT NULL AUTO_INCREMENT UNIQUE,
  game_name VARCHAR(255) NOT NULL,
  platform_name VARCHAR(50),
  year_released CHAR(4) NULL,
  genre_name VARCHAR(25) NULL,
  publisher_name VARCHAR(100) NULL,
  north_america_sales DECIMAL(5,2) NULL,
  europe_sales DECIMAL(5,2) NULL,
  japan_sales DECIMAL(5,2) NULL,
  other_sales DECIMAL(5,2) NULL,
  global_sales DECIMAL(5,2) NULL,
  critic_score CHAR(3) NULL,
  critic_count VARCHAR(10) NULL,
  user_score CHAR(3) NULL,
  user_count VARCHAR(10) NULL,
  developer_name VARCHAR(100) NULL,
  rating_name CHAR(4) NULL,
  PRIMARY KEY (game_id)
)
ENGINE=InnoDB
CHARACTER SET utf8mb4
COLLATE utf8mb4_0900_ai_ci;
```

Populate the table with row data read from the original source file.

:bulb: I recommend changing the original source file column names so that they match the database
 table column names.
 
Note in the example below an `IGNORE 1 LINES` option is added to the `LOAD DATA LOCAL INFILE` 
statement in order to skip the column headings located in the first row in the data set. The 
columns to be read are also specified. You can skip a column by placing a `@dummy` variable in 
the list in the position where the column would otherwise be referenced.

:warning: If columns in the source file include blank values you should use the SET statement and
 the IF() conditional check to replace each blank value encountered with NULL.

```mysql
LOAD DATA LOCAL INFILE './output/video_games/video_game_sales_trimmed.csv'
INTO TABLE temp_game
  CHARACTER SET utf8mb4
  -- FIELDS TERMINATED BY '\t'
  FIELDS TERMINATED BY ','
  ENCLOSED BY '"'
  LINES TERMINATED BY '\n'
  IGNORE 1 LINES
  (game_name, platform_name, year_released, genre_name,
  publisher_name, north_america_sales, europe_sales, japan_sales, other_sales,
  global_sales, critic_score, critic_count, user_score, user_count, developer_name,
  rating_name)

  SET game_name = IF(game_name = '', NULL, TRIM(game_name)),
  platform_name = IF(platform_name = '', NULL, TRIM(platform_name)),
  year_released = IF(year_released = '', NULL, year_released),
  genre_name = IF(genre_name = '', NULL, genre_name),
  publisher_name = IF(publisher_name = '', NULL, TRIM(publisher_name)),
  critic_score = IF(critic_score = '', NULL, critic_score),
  critic_count = IF(critic_count = '', NULL, critic_count),
  user_score = IF(user_score = '', NULL, user_score),
  user_count = IF(user_count = '', NULL, user_count),
  developer_name = IF(developer_name = '', NULL, TRIM(developer_name)),
  rating_name = IF(rating_name = '', NULL, TRIM(rating_name));
```

Create the actual table (e.g., `game`) that will be used in production to store game-related 
attributes. Add foreign key and other constraints consistent with your logical data model.

```mysql
CREATE TABLE IF NOT EXISTS game (
    game_id INTEGER NOT NULL AUTO_INCREMENT UNIQUE,
    game_name VARCHAR(255) NULL,
    platform_id INTEGER NULL,
    year_released INTEGER NULL,
    genre_id INTEGER NULL,
    publisher_id INTEGER NULL,
    north_america_sales DECIMAL(5, 2) NULL,
    europe_sales DECIMAL(5,2) NULL,
    japan_sales DECIMAL(5,2) NULL,
    other_sales DECIMAL(5,2) NULL,
    global_sales DECIMAL(5,2) NULL,
    critic_score INTEGER NULL,
    critic_count INTEGER NULL,
    user_score INTEGER NULL,
    user_count INTEGER NULL,
    developer_name VARCHAR(100) NULL,
    rating_id INTEGER NULL,
    PRIMARY KEY (game_id),
    FOREIGN KEY (platform_id) REFERENCES platform(platform_id)
    ON DELETE CASCADE ON UPDATE CASCADE,
    FOREIGN KEY (genre_id) REFERENCES genre(genre_id)
    ON DELETE CASCADE ON UPDATE CASCADE,
    FOREIGN KEY (publisher_id) REFERENCES publisher(publisher_id)
    ON DELETE CASCADE ON UPDATE CASCADE,
    FOREIGN KEY (rating_id) REFERENCES rating(rating_id)
    ON DELETE CASCADE ON UPDATE CASCADE
  )
  ENGINE=InnoDB
  CHARACTER SET utf8mb4
  COLLATE utf8mb4_0900_ai_ci;
```

Populate the table using an `INSERT INTO SELECT` statement.  In the example below, the 
`temp_game` table is joined to four production tables in order to return a result set that 
includes the foreign key integer values required for each row in the table.  Each join is based 
on an equality check of string values comprising the name of each entity.  

:bulb: Code defensively. Use the TRIM() function to guard against the inadvertent inclusion of whitespace in the string values.

```mysql
INSERT IGNORE INTO game
(
  game_name,
  platform_id,
  year_released,
  genre_id,
  publisher_id,
  north_america_sales,
  europe_sales,
  japan_sales,
  other_sales,
  global_sales,
  critic_score,
  critic_count,
  user_score,
  user_count,
  developer_name,
  rating_id
)
SELECT tg.game_name, plat.platform_id, CAST(tg.year_released AS UNSIGNED) AS year_released,
       g.genre_id, pub.publisher_id,
       tg.north_america_sales, tg.europe_sales, tg.japan_sales, tg.other_sales, tg.global_sales,
       CAST(tg.critic_score AS UNSIGNED) AS critic_score,
       CAST(tg.critic_count AS UNSIGNED) AS critic_count,
       CAST(tg.user_score AS UNSIGNED) AS user_score,
       CAST(tg.user_count AS UNSIGNED) AS user_count,
       tg.developer_name, r.rating_id
 FROM temp_game tg
      LEFT JOIN genre g
             ON TRIM(tg.genre_name) = TRIM(g.genre_name)
      LEFT JOIN platform plat
             ON TRIM(tg.platform_name) = TRIM(plat.platform_name)
      LEFT JOIN publisher pub
             ON TRIM(tg.publisher_name) = TRIM(pub.publisher_name)
      LEFT JOIN rating r
             ON TRIM(tg.rating_name) = TRIM(r.rating_name)
WHERE tg.game_name IS NOT NULL AND tg.game_name != ''
ORDER BY tg.global_sales DESC, tg.game_name, tg.year_released;
```

:warning: Remember to drop the temporary table when finished with it.

```mysql
DROP TEMPORARY TABLE temp_game;
```









See https://en.wikipedia.org/wiki/Newline