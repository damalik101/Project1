# Project1
Motivation
I'll use this script to provide introduction to data analysis using SQL language, which should be a must tool for every data scientist - both for getting access to data, but more interesting, as a simple tool for advance data analysis. The logic behind SQL is very similar to any other tool or language that used for data analysis (excel, Pandas), and for those that used to work with data, should be very intuitive.

Important Definitions
SQL is a conceptual language for working with data stored in databases. In our case, SQLite is the specific implementation. Most SQL languges share all of the capabilities in this doc. The differences are usually in performance and advances analytical funcionalities (and pricing of course). Eventually, we will use SQL lunguage to write queries that would pull data from the DB, manipulate it, sort it, and extract it.

The most important component of the DB is its tables - that's where all the data stored. Usually the data would be devided to many tables, and not stored all in one place (so designing the data stracture properly is very important). Most of this script would handle how to work with tables. Other than tables, there are some other very useful concepts/features that we won't cover here:

# Input data files are available in the "../input/" directory.
# For example, running this (by clicking run or pressing Shift+Enter) will list the files in the input directory

First we will create the connection to the DB, and see what tables we have
The basic structure of the query is very simple: You define what you want to see after the SELECT, * means all possible columns You choose the table after the FROM You add the conditions for the data you want to use from the table(s) after the WHERE

The stracture, and the order of the sections matter, while spaces, new lines, capital words and indentation are there to make the code easier to read.


List of leagues and their country
JOIN is used when you want to connect two tables to each other. It works when you have a common key in each of them. Understanding the concept of Keys is crucial for connecting (joining) between data set (tables). A key is uniquely identifies each record (row) in a table. It can consinst of one value (cell) - usually ID, or from a combination of values that are unique in the table.

When joinin between different tables, you must:

Decide what type of join to use. The most common are:
(INNER) JOIN - keep only records that match the condition (after the ON) in both the tables, and records in both tables that do not match wouldn't appear in the output
LEFT JOIN - keep all the values from the first (left) table - in conjunction with the matching rows from the right table. The columns from the right table, that don't have matching value in the left, would have NULL values.
Specify the common value that is used to connect the tables (the ID of the country in that case).
Make sure that at least one of the values has to be a key in its table. In our case, it's the Country.id. The League.country_id is not unique, as there can be more than one league in the same country
JOINs, and using them incorrectly, is the most common and dangerious mistake when writing complicated queries


List of matches
In this exapmle we will show only the columns that interests us, so instead of * we will use the exact names.

Some of the cells have the same name (Country.name,League.name). We will rename them using AS.

As you can see, this query has much more joins. The reasons is because the DB is designed in a star structure - one table (Match) with all the "performance" and metrics, but only keys and IDs, while all the descriptive information stored in other tables (Country, League, Team)

Note that Team is joined twice. This is a tricky one, as while we are using the same table name, we basically bring two different copies (and rename them using AS). The reason is that we need to bring information about two different values (home_team_api_id, away_team_api_id), and if we join them to the same table, it would mean that they are equal to each other.

You will also note that the Team tables are joined using left join. The reason is that I would prefer to keep the matches in the output - even if one of the teams is missing from the Team table for some reason.

Let's do some basic analytics
Here we are starting to look at the data at more aggregated level. Instead of looking on the raw data we will start to grouping it to different levels we want to examine. In this example, we will base it on the previous query, remove the match and date information, and look at it at the country-league-season level.

The functionality we will use for that is GROUP BY, that comes between the WHERE and ORDER

Once you chose what level you want to analyse, we can devide the SELECT statement to two:

Dimensions - those are the values we describing, same that we will group by later.
Metrics - all the metrics have to be aggregated using functions. The common functions are: sum(), count(), count(distinct ...), avg(), min(), max()
Note - it is very important to use the same dimensions both in the SELECT, and in the GROUP BY. Otherwise the output might be wrong.

Another functionality that can be used after grouping, is HAVING. This adds another layer of filtering the data, this time the output of the table after the grouping. A lot of times it is used to clean the output.

Query Run Order
Now that we are familiar with most of the functionalities being used in a query, it is very important to understand the order that code runs.

As we mentioned, here is the order as it would appear in the code:

SELECT
FROM
JOIN
WHERE
GROUP BY
HAVING
ORDER BY
LIMIT
You can think of it as a two part process: First, create a new temporal table in the memory:

Define which tables to use, and connect them (FROM + JOIN)
Keep only the rows that apply to the conditions (WHERE)
Group the data by the required level (if need) (GROUP BY)
Choose what information you want to have in the new table. It can have just rawdata (if no grouping), or combination of dimensions (from the grouping), and metrics Then, choose what to show from the table:
Order the output of the new table (ORDER BY)
Add more conditions that would filter the new created table (HAVING)
Limit to number of rows - would cut it according the soring and the having filtering (LIMIT)
Sub Queries and Functions
Using subqueries is an essential tool in SQL, as it allows manipulating the data in very advanced ways without the need of any external scripts, and especially important when your tables stractured in such a way that you can't be joined directly.

In our example, I'm trying to join between a table that holds players' basic details (name, height, weight), to a table that holds more attributes. The problem is that while the first table holds one row for each player, the key in the second table is player+season, so if we do a regular join, the result would be a cartesian product, and each player's basic details would appear as many times as this player appears in the attributes table. The result would be that the average would be skewed towards players that appear many times in the attribute table.

The solution, is to use a subquery. We would need to group the attributes table, to a different key - player level only (without season). Of course we would need to decide first how we would want to combine all the attributes to a single row. I used average, but one can also decide on maximum, latest season and etc. Once both tables have the same keys, we can join them together (think of the subquery as any other table, only temporal), knowing that we won't have duplicated rows after the join.

In addition, you can see here two examples of how to use functions:

Conditional function is an important tool for data manipulation. While IF statement is very popular in other languages, SQLite is not supporting it, and it's implemented using CASE + WHEN + ELSE statement. As you can see, based on the input of the data, the query would return different results.

ROUND - straight sorward. Every SQL languages comes with a lot of usefull functions by default.
