# "Top Diff"
**Jeremy Quinto**

## Introduction

This project was done for DSC80 at UCSD, 2023.

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

#### `players`

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

However, League of Legends is a **team sport**. In baseball, the LA Angels have arguably the two best players in the league and haven't even made the playoffs since 2019. So finding a perfect "performance metric" may not be accurate if we base it solely on individual performance. 

So what we will do is create a dataframe called `teams`, which has the average statistics for each team. Then we will find the variables with the highest correlation with winrate, and create our performance metric from there.

We made similar adjustments to what we did for the `players` dataframe. Here is *part* of what our final `teams` dataset looks like:

#### `teams`

| teamname                |   winrate |   KDA |   gamesplayed |
|:------------------------|----------:|------:|--------------:|
| 100 Thieves             |     0.566 | 3.944 |            76 |
| 100 Thieves Academy     |     0.581 | 3.522 |           117 |
| 100 Thieves Next        |     0.766 | 5.04  |            77 |
| 1907 Fenerbahçe Academy |     0.667 | 3.821 |             3 |
| 300                     |     0.417 | 3.083 |            12 |
| 42 Gaming               |     0.375 | 2.131 |             8 |
| 5 Ronin                 |     0.31  | 2.06  |            42 |
| 5 Ronin Academy         |     0.176 | 1.894 |            34 |
| 9z Team                 |     0.413 | 2.794 |            46 |
| AGD E-Sports            |     0     | 1.27  |             3 |

And that concluded our data cleaning!

--- 
## Bivariate Analysis: Making a Performance Metric
Now we are ready to start creating our **performance metric** using our `teams` dataframe. 

The first variable that comes to mind when winning games it *getting kills and not dying*. So of course, we look at the correlation between `KDA` and `winrate`. 

<iframe src="assets/kdaVwinrate1.html" width=800 height=600 frameBorder=0></iframe>

We can see a lot of teams have a winrate of `1`, which is impressive! But this is because many teams only played a couple games. After filtering out teams that didn't reach the cutoff I made, this was the results:

<iframe src="assets/kdaVwinrate2.html" width=800 height=600 frameBorder=0></iframe>

As expected, there is a very clear correlation between `KDA` and `winrate`, so we will use `KDA` in our performance metric. This makes sense, since killing the enemy and dying less definitely makes winning the game easier.

We used the following code to get the correlation coefficients for each column:

```
correlation_values = teams.select_dtypes(include='number').corr()['winrate']
# Sort the correlation values in descending order
sorted_correlation = correlation_values.abs().sort_values(ascending=False)
sorted_correlation.head(30)
```

These all make sense. For example, obviously taking more towers would result in more wins, as you have to take towers to win the game. But a lot of these statistics are team-wide, and the original dataset does not have individual player data for these. 

### Deciding The Performance Metric: W-SCORE

So based on the correlation coefficients calculated above, our performance metric will be:

(**STD KDA** X 0.9) + (**STD EARNED GPM** X 0.88) + (**STD GOLD DIFF AT 15** X 0.75) + (**STD XP DIFF AT 15** X 0.73) + (**STD CS DIFF AT 15** X 0.64)

This is the sum of the top 5 player-specific statistics in terms of correlation coefficient with winrate, **standardized**, and weighted by their respective correlation coefficients. Of course, this isn't a *perfect* performance metric, as there are many intangible things that players can do to be better. But this is the best analysis we can do using the statistics we have. 

Going forward, we'll refer to this statistic as **"W-Score"**.

---
## Univariate Analysis: Who is the best? (and worst?)

So now that we have a performance metric, we can find out the best and worst players in the LCS in terms of W-Score. 

First of all, we don't want outliers in our data, so we will set a minimum amount of games played as 10. 

Then, we calculated the W-Score of each player and added it to the dataframe. 

| playername   | team              | position   |    WSCORE |
|:-------------|:------------------|:-----------|----------:|
| Abbedagge    | 100 Thieves       | mid        |  0.130822 |
| Ablazeolive  | Golden Guardians  | mid        | -1.37417  |
| Aiming       | KT Rolster        | bot        |  4.03204  |
| Aria         | KT Rolster        | mid        | -2.12278  |
| Arrow        | Immortals         | bot        | -3.79693  |
| Bdd          | Nongshim RedForce | mid        | -0.360007 |
| Berserker    | Cloud9            | bot        |  3.7277   |
| BeryL        | DRX               | sup        |  0.149048 |
| Biofrost     | Dignitas          | sup        | -1.48645  |
| Bjergsen     | Team Liquid       | mid        |  5.21788  |

Below is the distribution of the WSCOREs for the league:

<iframe src="assets/wscore.html" width=800 height=600 frameBorder=0></iframe>

Of course, since we used standardized data, our WSCORE is roughly normally distributed. 

### Interesting Aggregates: Positions

Now we want to want to choose the players with the best WSCORE of the bunch. But there's a problem: League of Legends has different roles, or **positions**. We can expect that a player in the support role may not have the same amount of kills as someone from the ACD role, for example. 

Let's take a look at the distribution of `WSCORE` for support players.

<iframe src="assets/wscore_sup.html" width=800 height=600 frameBorder=0></iframe>

As we can see, the `WSCORE` for support players is a lot lower than that of other roles. So when we choose the best, worst, and average players, we must choose the best, worst, and average players in each role. 

| position   |     WSCORE |        KDA |   earned gpm |   golddiffat15 |   xpdiffat15 |   csdiffat15 |
|:-----------|-----------:|-----------:|-------------:|---------------:|-------------:|-------------:|
| bot        |  0.904918  |  0.453928  |     0.94434  |     -0.217684  |   -0.0825527 |   -0.173609  |
| jng        | -0.0795993 |  0.0552768 |    -0.31044  |      0.137101  |    0.0112628 |    0.0512364 |
| mid        |  1.10015   |  0.461663  |     0.626748 |      0.0658577 |    0.0865614 |    0.0320888 |
| sup        | -1.39659   | -0.165469  |    -1.63583  |      0.0863344 |    0.0236002 |    0.171689  |
| top        | -0.350066  | -0.707908  |     0.434024 |     -0.0394479 |   -0.0266024 |   -0.0716953 |

It seems that `mid` and `bot` (ADC) players have the highest statistics, while `sup` and `jng` (jungle) players seem to have the lowest. But that doesn't mean that support and jungle players are worse, it just means their role tends to do less of these statistics. So we must adjust WSCORE to compare players to their individual roles, not just the league averages. 

So now we will calculated **PAWSCORE**, or **Position Adjusted W-Score**. We use the same formula as W-Score, but when standardizing, we standardize using the player's position instead of the entire dataset. 

Here is a look at *some* our data with `PAWSCORE` added:

| playername   |    WSCORE |   PAWSCORE |
|:-------------|----------:|-----------:|
| Aiming       |  4.03204  |  3.03437   |
| Arrow        | -3.79693  | -4.47901   |
| Berserker    |  3.7277   |  2.61146   |
| Cheoni       | -4.78286  | -5.12006   |
| Danny        |  3.6619   |  2.81346   |
| Deft         |  0.864622 | -0.0148235 |
| FBI          |  0.320103 | -0.319738  |
| Ghost        | -1.00998  | -1.94507   |
| Gumayusi     |  4.79718  |  3.76698   |
| Hans SamD    | -0.385392 | -1.11788   |

As you can see, there is a big difference between `WSCORE` and `PAWSCORE`. We can take a look at the distribution of `PAWSCORE`

<iframe src="assets/pawscore.html" width=800 height=600 frameBorder=0></iframe>

This distribution is a little less normal than the `WSCORE` distribution, which makes sense. 

The player on the far right is a player named Chovy. 

It seems that statistically, Chovy was the best player by a wide margin in `PAWSCORE`, at least for the 2022 season. A quick Google search says that Chovy is certainly *one* of the best `mid` players in the league, so this is a good sign for our `PAWSCORE` metric. In the season we are looking at, Chovy lead his team to the finals but ended up losing. 

Now that we have the `PAWSCORE` metric, we can answer our question: **Is the gap between the best player(s) and the average players larger than the gap between the average players and the worst players?**

---
# Hypothesis Testing: Answering the Question

These are our hypotheses:

$H_0$ (null hypothesis): The gap between the best player and the average player is **not larger** than the gap between the average player and the worst players.

$H_1$ (alt. hypothesis): The gap between the best player and the average player **is larger** than the gap between the average player and the worst players.

We will use a **significance level** of **5%**. 

Our **test statistic** is going to be:

(gap between top and average) - (gap between average and worst)

=

(mean PAWSCORE of top players - league average PAWSCORE) - (league average PAWSCORE - mean PAWSCORE of bottom players)

This is the difference between the gap between the best and average players, and the gap between the average and worst players. 

This can be simplified knowing that the league average `PAWSCORE` is 0:

(mean PAWSCORE of top players) - abs(mean PAWSCORE of bottom players)



We will look at the top 30 and bottom 30 players in terms of `PAWSCORE`. Note that the players in the top 30 all have a positive `PAWSCORE`, and the players in the bottom 30 all have a negative `PAWSCORE`. 30 players in either tier is a reasonable amount of players to be considered the "best of the league" and the "worst of the league". (*Note*: changing these numbers didn't change the final result anyway)

Calculating the gaps: for `top_players`, the gap is just their `PAWSCORE`. For the `bottom_players`, the gap is the absolute value of their `PAWSCORE`. We can store these values to compare in `ABS PAWSCORE`:

Our **observed test statistic** is the absolute difference between the means of `ABS PAWSCORE` for `top_players` and `bottom_players`. **This value ended up being 0.334001**.

We performed a permutation test, shuffling values to get the distribution of the test statistic. 
```
n_repetitions = 100

differences = []
for _ in range(n_repetitions):
    # Step 1: Shuffle the PAWSCORES and store them in a DataFrame
    with_shuffled = combined_players.assign(Shuffled_PAW=np.random.permutation(combined_players['ABS PAWSCORE']))
    
    # Step 2: Compute the test statistic
    group_means = (
        with_shuffled
        .groupby('top')
        .mean()
        .loc[:, 'Shuffled_PAW']
    )
    difference = group_means.diff().iloc[-1]
    
    differences.append(difference)
```

The distribution of the test statistic we found can be found below. The red line is the observed statistic. 

<iframe src="assets/empirical_distribution.html" width=800 height=600 frameBorder=0></iframe>

Computing our **p-value**:

```
p_value = np.mean(differences >= observed_diff)
```
We obtained our **p-value = 0.19**.

---
## Conclusion

This means that in this case, we fail to reject the null hypothesis. In other words, **there is not sufficient evidence to support the claim that the gap between the best players and the average player is higher than the gap between the average player and the worst players, specifically in the LCS and LCK 2022 season**. 

What does this mean? This doesn't necesarilly mean that the average player in the league is as close to the bottom as they are to the top, we can't know for sure. The gap between the highest player in the league and the average player *could* still be higher, but there is not sufficient enough evidence to point to this.

Thus, the relative locations of the "skill floor" and "skill ceilings" remain unknown.

However, we have at least come up with an interesting performance metric to rate League of Legends players! Here is the sorted list of the top 25 players from the 2022 LCS and LCK season, based on the `PAWSCORE` metric, if you're curious:

| playername   | team             | position   |   PAWSCORE |
|:-------------|:-----------------|:-----------|-----------:|
| Chovy        | Gen.G            | mid        |   10.8312  |
| Prince       | Liiv SANDBOX     | bot        |    6.79801 |
| Summit       | Cloud9           | top        |    6.07203 |
| Oner         | T1               | jng        |    5.97414 |
| huhi         | 100 Thieves      | sup        |    5.39983 |
| Hans Sama    | Team Liquid      | bot        |    5.26718 |
| Bjergsen     | Team Liquid      | mid        |    5.17724 |
| Blaber       | Cloud9           | jng        |    5.10206 |
| Ruler        | Gen.G            | bot        |    4.73252 |
| Peanut       | Gen.G            | jng        |    4.48142 |
| Keria        | T1               | sup        |    4.36877 |
| Zeus         | T1               | top        |    4.25127 |
| Bwipo        | Team Liquid      | top        |    4.24592 |
| Gumayusi     | T1               | bot        |    3.76698 |
| Doran        | Gen.G            | top        |    3.73132 |
| Ssumday      | 100 Thieves      | top        |    3.6527  |
| Pridestalkr  | Golden Guardians | jng        |    3.62718 |
| Moham        | Kwangdong Freecs | sup        |    3.54426 |
| Nuguri       | DWG KIA          | top        |    3.37927 |
| Inspired     | Evil Geniuses    | jng        |    3.30361 |
| Lehends      | Gen.G            | sup        |    3.15099 |
| Jensen       | Cloud9           | mid        |    3.08008 |
| Aiming       | KT Rolster       | bot        |    3.03437 |
| BeryL        | DRX              | sup        |    2.83118 |
| Danny        | Evil Geniuses    | bot        |    2.81346 |


# Thanks for reading!
