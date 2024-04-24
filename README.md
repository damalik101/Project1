# Project1
Motivation
I'll use this script to provide introduction to data analysis using SQL language, which should be a must tool for every data scientist - both for getting access to data, but more interesting, as a simple tool for advance data analysis. The logic behind SQL is very similar to any other tool or language that used for data analysis (excel, Pandas), and for those that used to work with data, should be very intuitive.

Important Definitions
SQL is a conceptual language for working with data stored in databases. In our case, SQLite is the specific implementation. Most SQL languges share all of the capabilities in this doc. The differences are usually in performance and advances analytical funcionalities (and pricing of course). Eventually, we will use SQL lunguage to write queries that would pull data from the DB, manipulate it, sort it, and extract it.

The most important component of the DB is its tables - that's where all the data stored. Usually the data would be devided to many tables, and not stored all in one place (so designing the data stracture properly is very important). Most of this script would handle how to work with tables. Other than tables, there are some other very useful concepts/features that we won't cover here:

table creation
inserting / updating data in the DB
functions - gets a value as an input, and returns manipulation of that value (for example function that remove white spaces)
#Improts 

import numpy as np # linear algebra
import pandas as pd # data processing, CSV file I/O (e.g. pd.read_csv)
import sqlite3
import matplotlib.pyplot as plt

# Input data files are available in the "../input/" directory.
# For example, running this (by clicking run or pressing Shift+Enter) will list the files in the input directory

path = "../input/"  #Insert path here
database = path + 'database.sqlite'
First we will create the connection to the DB, and see what tables we have
The basic structure of the query is very simple: You define what you want to see after the SELECT, * means all possible columns You choose the table after the FROM You add the conditions for the data you want to use from the table(s) after the WHERE

The stracture, and the order of the sections matter, while spaces, new lines, capital words and indentation are there to make the code easier to read.

conn = sqlite3.connect(database)

tables = pd.read_sql("""SELECT *
                        FROM sqlite_master
                        WHERE type='table';""", conn)
tables
type	name	tbl_name	rootpage	sql
0	table	sqlite_sequence	sqlite_sequence	4	CREATE TABLE sqlite_sequence(name,seq)
1	table	Player_Attributes	Player_Attributes	11	CREATE TABLE "Player_Attributes" (\n\t`id`\tIN...
2	table	Player	Player	14	CREATE TABLE `Player` (\n\t`id`\tINTEGER PRIMA...
3	table	Match	Match	18	CREATE TABLE `Match` (\n\t`id`\tINTEGER PRIMAR...
4	table	League	League	24	CREATE TABLE `League` (\n\t`id`\tINTEGER PRIMA...
5	table	Country	Country	26	CREATE TABLE `Country` (\n\t`id`\tINTEGER PRIM...
6	table	Team	Team	29	CREATE TABLE "Team" (\n\t`id`\tINTEGER PRIMARY...
7	table	Team_Attributes	Team_Attributes	2	CREATE TABLE `Team_Attributes` (\n\t`id`\tINTE...
List of countries
This is the most basic query. The only must parts of a qeury is the SELECT and the FROM (assuming you want to pull from a table)

countries = pd.read_sql("""SELECT *
                        FROM Country;""", conn)
countries
id	name
0	1	Belgium
1	1729	England
2	4769	France
3	7809	Germany
4	10257	Italy
5	13274	Netherlands
6	15722	Poland
7	17642	Portugal
8	19694	Scotland
9	21518	Spain
10	24558	Switzerland
List of leagues and their country
JOIN is used when you want to connect two tables to each other. It works when you have a common key in each of them. Understanding the concept of Keys is crucial for connecting (joining) between data set (tables). A key is uniquely identifies each record (row) in a table. It can consinst of one value (cell) - usually ID, or from a combination of values that are unique in the table.

When joinin between different tables, you must:

Decide what type of join to use. The most common are:
(INNER) JOIN - keep only records that match the condition (after the ON) in both the tables, and records in both tables that do not match wouldn't appear in the output
LEFT JOIN - keep all the values from the first (left) table - in conjunction with the matching rows from the right table. The columns from the right table, that don't have matching value in the left, would have NULL values.
Specify the common value that is used to connect the tables (the ID of the country in that case).
Make sure that at least one of the values has to be a key in its table. In our case, it's the Country.id. The League.country_id is not unique, as there can be more than one league in the same country
JOINs, and using them incorrectly, is the most common and dangerious mistake when writing complicated queries

leagues = pd.read_sql("""SELECT *
                        FROM League
                        JOIN Country ON Country.id = League.country_id;""", conn)
leagues
id	country_id	name	id	name
0	1	1	Belgium Jupiler League	1	Belgium
1	1729	1729	England Premier League	1729	England
2	4769	4769	France Ligue 1	4769	France
3	7809	7809	Germany 1. Bundesliga	7809	Germany
4	10257	10257	Italy Serie A	10257	Italy
5	13274	13274	Netherlands Eredivisie	13274	Netherlands
6	15722	15722	Poland Ekstraklasa	15722	Poland
7	17642	17642	Portugal Liga ZON Sagres	17642	Portugal
8	19694	19694	Scotland Premier League	19694	Scotland
9	21518	21518	Spain LIGA BBVA	21518	Spain
10	24558	24558	Switzerland Super League	24558	Switzerland
List of teams
ORDER BY defines the sorting of the output - ascending or descending (DESC)

LIMIT, limits the number of rows in the output - after the sorting

teams = pd.read_sql("""SELECT *
                        FROM Team
                        ORDER BY team_long_name
                        LIMIT 10;""", conn)
teams
id	team_api_id	team_fifa_api_id	team_long_name	team_short_name
0	16848	8350	29	1. FC Kaiserslautern	KAI
1	15624	8722	31	1. FC Köln	FCK
2	16239	8165	171	1. FC Nürnberg	NUR
3	16243	9905	169	1. FSV Mainz 05	MAI
4	11817	8576	614	AC Ajaccio	AJA
5	11074	108893	111989	AC Arles-Avignon	ARL
6	49116	6493	1714	AC Bellinzona	BEL
7	26560	10217	650	ADO Den Haag	HAA
8	9537	8583	57	AJ Auxerre	AUX
9	9547	9829	69	AS Monaco	MON
List of matches
In this exapmle we will show only the columns that interests us, so instead of * we will use the exact names.

Some of the cells have the same name (Country.name,League.name). We will rename them using AS.

As you can see, this query has much more joins. The reasons is because the DB is designed in a star structure - one table (Match) with all the "performance" and metrics, but only keys and IDs, while all the descriptive information stored in other tables (Country, League, Team)

Note that Team is joined twice. This is a tricky one, as while we are using the same table name, we basically bring two different copies (and rename them using AS). The reason is that we need to bring information about two different values (home_team_api_id, away_team_api_id), and if we join them to the same table, it would mean that they are equal to each other.

You will also note that the Team tables are joined using left join. The reason is that I would prefer to keep the matches in the output - even if one of the teams is missing from the Team table for some reason.

ORDER defines the order of the output, and comes before the LIMIT and after the WHERE

detailed_matches = pd.read_sql("""SELECT Match.id, 
                                        Country.name AS country_name, 
                                        League.name AS league_name, 
                                        season, 
                                        stage, 
                                        date,
                                        HT.team_long_name AS  home_team,
                                        AT.team_long_name AS away_team,
                                        home_team_goal, 
                                        away_team_goal                                        
                                FROM Match
                                JOIN Country on Country.id = Match.country_id
                                JOIN League on League.id = Match.league_id
                                LEFT JOIN Team AS HT on HT.team_api_id = Match.home_team_api_id
                                LEFT JOIN Team AS AT on AT.team_api_id = Match.away_team_api_id
                                WHERE country_name = 'Spain'
                                ORDER by date
                                LIMIT 10;""", conn)
detailed_matches
id	country_name	league_name	season	stage	date	home_team	away_team	home_team_goal	away_team_goal
0	21518	Spain	Spain LIGA BBVA	2008/2009	1	2008-08-30 00:00:00	Valencia CF	RCD Mallorca	3	0
1	21525	Spain	Spain LIGA BBVA	2008/2009	1	2008-08-30 00:00:00	RCD Espanyol	Real Valladolid	1	0
2	21519	Spain	Spain LIGA BBVA	2008/2009	1	2008-08-31 00:00:00	CA Osasuna	Villarreal CF	1	1
3	21520	Spain	Spain LIGA BBVA	2008/2009	1	2008-08-31 00:00:00	RC Deportivo de La Coruña	Real Madrid CF	2	1
4	21521	Spain	Spain LIGA BBVA	2008/2009	1	2008-08-31 00:00:00	CD Numancia	FC Barcelona	1	0
5	21522	Spain	Spain LIGA BBVA	2008/2009	1	2008-08-31 00:00:00	Racing Santander	Sevilla FC	1	1
6	21523	Spain	Spain LIGA BBVA	2008/2009	1	2008-08-31 00:00:00	Real Sporting de Gijón	Getafe CF	1	2
7	21524	Spain	Spain LIGA BBVA	2008/2009	1	2008-08-31 00:00:00	Real Betis Balompié	RC Recreativo	0	1
8	21526	Spain	Spain LIGA BBVA	2008/2009	1	2008-08-31 00:00:00	Athletic Club de Bilbao	UD Almería	1	3
9	21527	Spain	Spain LIGA BBVA	2008/2009	1	2008-08-31 00:00:00	Atlético Madrid	Málaga CF	4	0
Let's do some basic analytics
Here we are starting to look at the data at more aggregated level. Instead of looking on the raw data we will start to grouping it to different levels we want to examine. In this example, we will base it on the previous query, remove the match and date information, and look at it at the country-league-season level.

The functionality we will use for that is GROUP BY, that comes between the WHERE and ORDER

Once you chose what level you want to analyse, we can devide the SELECT statement to two:

Dimensions - those are the values we describing, same that we will group by later.
Metrics - all the metrics have to be aggregated using functions. The common functions are: sum(), count(), count(distinct ...), avg(), min(), max()
Note - it is very important to use the same dimensions both in the SELECT, and in the GROUP BY. Otherwise the output might be wrong.

Another functionality that can be used after grouping, is HAVING. This adds another layer of filtering the data, this time the output of the table after the grouping. A lot of times it is used to clean the output.

leages_by_season = pd.read_sql("""SELECT Country.name AS country_name, 
                                        League.name AS league_name, 
                                        season,
                                        count(distinct stage) AS number_of_stages,
                                        count(distinct HT.team_long_name) AS number_of_teams,
                                        avg(home_team_goal) AS avg_home_team_scors, 
                                        avg(away_team_goal) AS avg_away_team_goals, 
                                        avg(home_team_goal-away_team_goal) AS avg_goal_dif, 
                                        avg(home_team_goal+away_team_goal) AS avg_goals, 
                                        sum(home_team_goal+away_team_goal) AS total_goals                                       
                                FROM Match
                                JOIN Country on Country.id = Match.country_id
                                JOIN League on League.id = Match.league_id
                                LEFT JOIN Team AS HT on HT.team_api_id = Match.home_team_api_id
                                LEFT JOIN Team AS AT on AT.team_api_id = Match.away_team_api_id
                                WHERE country_name in ('Spain', 'Germany', 'France', 'Italy', 'England')
                                GROUP BY Country.name, League.name, season
                                HAVING count(distinct stage) > 10
                                ORDER BY Country.name, League.name, season DESC
                                ;""", conn)
leages_by_season
country_name	league_name	season	number_of_stages	number_of_teams	avg_home_team_scors	avg_away_team_goals	avg_goal_dif	avg_goals	total_goals
0	England	England Premier League	2015/2016	38	20	1.492105	1.207895	0.284211	2.700000	1026
1	England	England Premier League	2014/2015	38	20	1.473684	1.092105	0.381579	2.565789	975
2	England	England Premier League	2013/2014	38	20	1.573684	1.194737	0.378947	2.768421	1052
3	England	England Premier League	2012/2013	38	20	1.557895	1.239474	0.318421	2.797368	1063
4	England	England Premier League	2011/2012	38	20	1.589474	1.215789	0.373684	2.805263	1066
5	England	England Premier League	2010/2011	38	20	1.623684	1.173684	0.450000	2.797368	1063
6	England	England Premier League	2009/2010	38	20	1.697368	1.073684	0.623684	2.771053	1053
7	England	England Premier League	2008/2009	38	20	1.400000	1.078947	0.321053	2.478947	942
8	France	France Ligue 1	2015/2016	38	20	1.436842	1.089474	0.347368	2.526316	960
9	France	France Ligue 1	2014/2015	38	20	1.410526	1.081579	0.328947	2.492105	947
10	France	France Ligue 1	2013/2014	38	20	1.415789	1.039474	0.376316	2.455263	933
11	France	France Ligue 1	2012/2013	38	20	1.468421	1.076316	0.392105	2.544737	967
12	France	France Ligue 1	2011/2012	38	20	1.473684	1.042105	0.431579	2.515789	956
13	France	France Ligue 1	2010/2011	38	20	1.342105	1.000000	0.342105	2.342105	890
14	France	France Ligue 1	2009/2010	38	20	1.389474	1.021053	0.368421	2.410526	916
15	France	France Ligue 1	2008/2009	38	20	1.286842	0.971053	0.315789	2.257895	858
16	Germany	Germany 1. Bundesliga	2015/2016	34	18	1.565359	1.264706	0.300654	2.830065	866
17	Germany	Germany 1. Bundesliga	2014/2015	34	18	1.588235	1.166667	0.421569	2.754902	843
18	Germany	Germany 1. Bundesliga	2013/2014	34	18	1.748366	1.411765	0.336601	3.160131	967
19	Germany	Germany 1. Bundesliga	2012/2013	34	18	1.591503	1.343137	0.248366	2.934641	898
20	Germany	Germany 1. Bundesliga	2011/2012	34	18	1.660131	1.199346	0.460784	2.859477	875
21	Germany	Germany 1. Bundesliga	2010/2011	34	18	1.647059	1.274510	0.372549	2.921569	894
22	Germany	Germany 1. Bundesliga	2009/2010	34	18	1.513072	1.316993	0.196078	2.830065	866
23	Germany	Germany 1. Bundesliga	2008/2009	34	18	1.699346	1.222222	0.477124	2.921569	894
24	Italy	Italy Serie A	2015/2016	38	20	1.471053	1.105263	0.365789	2.576316	979
25	Italy	Italy Serie A	2014/2015	38	20	1.498681	1.187335	0.311346	2.686016	1018
26	Italy	Italy Serie A	2013/2014	38	20	1.536842	1.186842	0.350000	2.723684	1035
27	Italy	Italy Serie A	2012/2013	38	20	1.494737	1.144737	0.350000	2.639474	1003
28	Italy	Italy Serie A	2011/2012	38	20	1.511173	1.072626	0.438547	2.583799	925
29	Italy	Italy Serie A	2010/2011	38	20	1.431579	1.081579	0.350000	2.513158	955
30	Italy	Italy Serie A	2009/2010	38	20	1.542105	1.068421	0.473684	2.610526	992
31	Italy	Italy Serie A	2008/2009	38	20	1.521053	1.078947	0.442105	2.600000	988
32	Spain	Spain LIGA BBVA	2015/2016	38	20	1.618421	1.126316	0.492105	2.744737	1043
33	Spain	Spain LIGA BBVA	2014/2015	38	20	1.536842	1.118421	0.418421	2.655263	1009
34	Spain	Spain LIGA BBVA	2013/2014	38	20	1.631579	1.118421	0.513158	2.750000	1045
35	Spain	Spain LIGA BBVA	2012/2013	38	20	1.686842	1.184211	0.502632	2.871053	1091
36	Spain	Spain LIGA BBVA	2011/2012	38	20	1.678947	1.084211	0.594737	2.763158	1050
37	Spain	Spain LIGA BBVA	2010/2011	38	20	1.636842	1.105263	0.531579	2.742105	1042
38	Spain	Spain LIGA BBVA	2009/2010	38	20	1.600000	1.113158	0.486842	2.713158	1031
39	Spain	Spain LIGA BBVA	2008/2009	38	20	1.660526	1.236842	0.423684	2.897368	1101
<matplotlib.axes._subplots.AxesSubplot at 0x7f32ab696b70>

<matplotlib.axes._subplots.AxesSubplot at 0x7f32ab574748>

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

players_height = pd.read_sql("""SELECT CASE
                                        WHEN ROUND(height)<165 then 165
                                        WHEN ROUND(height)>195 then 195
                                        ELSE ROUND(height)
                                        END AS calc_height, 
                                        COUNT(height) AS distribution, 
                                        (avg(PA_Grouped.avg_overall_rating)) AS avg_overall_rating,
                                        (avg(PA_Grouped.avg_potential)) AS avg_potential,
                                        AVG(weight) AS avg_weight 
                            FROM PLAYER
                            LEFT JOIN (SELECT Player_Attributes.player_api_id, 
                                        avg(Player_Attributes.overall_rating) AS avg_overall_rating,
                                        avg(Player_Attributes.potential) AS avg_potential  
                                        FROM Player_Attributes
                                        GROUP BY Player_Attributes.player_api_id) 
                                        AS PA_Grouped ON PLAYER.player_api_id = PA_Grouped.player_api_id
                            GROUP BY calc_height
                            ORDER BY calc_height
                                ;""", conn)
players_height
calc_height	distribution	avg_overall_rating	avg_potential	avg_weight
0	165.0	74	67.365543	73.327754	139.459459
1	168.0	118	67.500518	73.124182	144.127119
2	170.0	403	67.726903	73.379056	147.799007
3	173.0	530	66.980272	72.848746	152.824528
4	175.0	1188	66.805204	72.258774	156.111953
5	178.0	1489	66.367212	71.943339	160.665547
6	180.0	1388	66.419053	71.846394	165.261527
7	183.0	1954	66.634380	71.754555	170.167861
8	185.0	1278	66.928964	71.833475	174.636933
9	188.0	1305	67.094253	72.151949	179.278161
10	191.0	652	66.997649	71.846159	184.791411
11	193.0	470	67.485141	72.459225	188.795745
12	195.0	211	67.425619	72.615373	196.464455
/opt/conda/lib/python3.6/site-packages/pandas/plotting/_core.py:1716: UserWarning: Pandas doesn't allow columns to be created via a new attribute name - see https://pandas.pydata.org/pandas-docs/stable/indexing.html#attribute-access
  series.name = label
<matplotlib.axes._subplots.AxesSubplot at 0x7f32ab4fc048>
