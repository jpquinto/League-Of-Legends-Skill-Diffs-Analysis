# "Top Diff"
**Jeremy Quinto**

## Introduction

Often in video games, when outplaying their opponent in their position, will type "(position) diff". For example, when I dominate the tank player on the other team, I often type "tank diff" in the chat to let them know. 

Today we'll be looking at a dataset that contains data on different player's performances in the 2022 professional season for **League of Legends**. The dataset is very large, and contains data from all the different professional leagues. It has almost every stat you can think of in the game, such as kills, deaths, assists, barons, towers, gold earned per minute, and more. The data can be found [here](https://oracleselixir.com/tools/downloads)

In a game like League of Legends, which is very complex and intricate, the range of skill levels of players will vary greatly. The difference between the "skill floor" and the "skill ceiling" is a very big range, at all levels of the game. But this brings up an interesting question: is the average player in the professional leagues closer to the "skill floor" or the "skill ceiling"? Or in other words:

## Question
**Is the gap between the best player(s) and the average players larger than the gap between the average players and the worst players?** 

To answer this question, we will first find the MVP(s) and "LVPs" (Least Valuable Players) of the league, as well as the average players. Then we will perform a hypothesis test to see which gap is larger. 

Of course, like in other sports, the best and worst players can be subjective. But I plan to use data to create a performance metric that represents a player's overall performance. This will be a combination of statistics that we will find are most important to winning. 

### Procedure
We will do the following steps in this report:

1) Retrieve the data

2) Perform **missingness analysis** to see what values are missing from the data and why they are

3) **Clean** the data and get them in the form we want

4) Use **bivariate analysis** to create a performance metric

5) Use **univariate analysis** and our performance metric to choose the best, average, and worst players in the league

6) Perform a **hypothesis test** to answer our question


--- 

## Missingness Analysis
After retrieving the data, we perform **missingness analysis** on the data to see what columns are missing values. 

We found that for the data there were many columns, such as `doublekills`, `triplekills`, `quadrakills`, and many more were **NMAR**, (Not Missing At Random). 

We find all rows missing any of those values are missing all of them. Furthermore, we run permutation tests to test missingness between `doublekills` and `triplekills`, and got a p-value of 0, indicating that there was a strong correlation between the missingness of those columns. We also run a similar test between `doublekills` and `url`, but found no correlation.

Next, we find that there were only 4 leagues that were missing those values. These are the leagues that have *at least 1* missing value for `doublekills`:
<iframe src="assets/missing_values_plot.html" width=800 height=600 frameBorder=0></iframe>

For reference, there are over 40 total leagues in the dataset, and only 4 of them were missing values in `doublekills`, and those 4 are missing **a lot**.

Thus, it seems like the columns in `columns_with_missing` are **not missing at random** (NMAR), as we can determine if a value is missing by looking at the `league` column. 

This is partially why we will only be looking at two leagues when evaluating our players today: `LCS` and `LCK`. This makes it so we don't have to deal with  missing values in those columns. 

We end up keeping only the columns that are relevant to our question, and as it turns out, there are no missing values for these columns in the `LCS` and `LCK`!

---
## Data Cleaning
Besides the steps described in the missingness analysis section, there were many other steps in the data cleaning process.

First, we create a dataframe called `players` that has the season statistics for each player. 

The numerical statistics are their per-game averages, and we write some extra code to get the non-numerical statistics in this dataframe. We also add other statistics that we calculate ourselves, such as "games played", "minutes played", and "KDA", which is (Kills+Assists)/Deaths. 

Other steps included:

- Renaming the "result" column to "winrate"

- Rounding the numerical values to 2 decimals

Here is *part* of what our final `players` dataset looks like:

| playername   | team                | position   |   KDA |   gamesplayed |
|:-------------|:--------------------|:-----------|------:|--------------:|
| Abbedagge    | 100 Thieves         | mid        |  3.6  |            70 |
| Ablazeolive  | Golden Guardians    | mid        |  2.37 |            43 |
| Aiming       | KT Rolster          | bot        |  4.71 |            93 |
| Aria         | KT Rolster          | mid        |  3.44 |            38 |
| Arrow        | Immortals           | bot        |  2.67 |            10 |
| Baut         | Hanwha Life Esports | sup        |  2.07 |             5 |
| Bdd          | Nongshim RedForce   | mid        |  3.23 |            82 |
| Berserker    | Cloud9              | bot        |  4.82 |            59 |
| BeryL        | DRX                 | sup        |  2.47 |            94 |
| Bible        | DWG KIA             | sup        |  1.61 |             5 |

Next, note that when thinking about creating a performance metric, we must find the variables that are the most important to winning games. We will find the variables with the highest correlation with winrate.

However, League of Legends is a **team sport**. In baseball, the LA Angels have arguably the two best players in the league and haven't even made the playoffs since 2019 (😂). So finding a perfect "performance metric" may not be accurate if we base it solely on individual performance. 

So what we will do is create a dataframe called `teams`, which has the average statistics for each team. Then we will find the variables with the highest correlation with winrate, and create our performance metric from there.

