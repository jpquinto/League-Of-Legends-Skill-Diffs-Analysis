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
After retrieving the data, we performed **missingness analysis** on the data to see what columns are missing values. 

We found that for the data there were many columns, such as `doublekills`, `triplekills`, `quadrakills`, and many more were **NMAR**, (Not Missing At Random). 

We found all rows missing any of those values were missing all of them. Furthermore, we ran permutation tests to test missingness between `doublekills` and `triplekills`, and got a p-value of 0, indicating that there was a strong correlation between the missingness of those columns. We also ran a similar test between `doublekills` and `url`, but found no correlation.

Next, we found that there were only 4 leagues that were missing those values. These are the leagues that had *at least 1* missing value for `doublekills`:
<iframe src="assets/missing_values_plot.html" width=800 height=600 frameBorder=0></iframe>

