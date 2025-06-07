# Introduction
Inspired by the terrific 24/25 UEFA Champions League semi-final tie between Inter and Barcelona, I challenged myself to quantify football entertainment.

What was the most entertaining match (excluding the final) in the 24/25 Champions League season?

# The Dataset

I compiled data for all 188 Champions League matches before the final, manually summing up, except one subtraction, the following stats for each match: 

- **Goals**
- **Shots**
- **Shots on target**
- **Big chances**
- **Expected goals**
- **Coefficient difference (subtraction)**

**Feel free to download the CSV and use the data however you like.**

**I created this dataset from publicly available data.**

Original data source: Sofascore

# Terminology

**Skip this section if you are familiar with all or any of the following terms: coefficient difference, big chances and expected goals.**

**Coefficient diffrence**

- Betting companies provide victory odds for each team before the game. Sofascore has basic Cro Bet odds integrated in their match data. I subtracted the smaller number from the bigger one to create this data.

**Big chances**

- A situation where a player should reasonably be expected to score, usually in a one-on-one scenario or from very close range when the ball has a clear path to goal and there is low to moderate pressure on the shooter. Penalties are considered big chances, except for shootout penalties.

Definition source: Opta Match Event Definitions

**Expected goals**

- Expected Goals (xG) measures the quality of a shot based on several variables such as assist type, shot angle and distance from goal, whether it was a headed shot and whether it was defined as a big chance. Adding up a player or team’s expected Goals can give us an indication of how many goals a player or team should have scored on average, given the shots they have taken. 

Definition source: Opta Match Event Definitions

# Tools I used
- **Excel (for data entry)**
- **SQL**
- **PostgreSQL**
- **Visual Studio Code**
- **Git & Github (for version control and sharing)** 

# Quantification method

- The more events (goals, shots etc.) a match produces, the more entertaining it should be in theory

- If teams are more evenly matched, fewer events are required for more entertainment

**Ranking**

- I ranked each match by goals, shots, shots on target, big chances, expected goals, and coefficient difference

- More events = better rank, smaller difference between coefficients = better rank

**Weighting**

- Not all events are equally important, so I applied these weights to each rank:

    - Goals rank * 1.6
    - Shots rank * 1.2
    - Shots on target rank * 1.3
    - Big chances rank * 1.5
    - Expected goals rank * 1.1
    - Coefficient difference rank * 1.4

- I averaged out weighted ranks, resulting in a single value called **weighted average rank** for each match

**Results**

- The match with the best weighted average rank is ranked #1 for entertainment



# Code

```sql
-- Getting the basic summed-up stats
WITH basic_stats AS (
    SELECT 
        match_id,
        phase,
        home_team,
        away_team,
        goals_combined,
        shots_combined,
        shots_on_target_combined,
        big_chances_combined,
        expected_goals_combined,
        coefficient_difference
    FROM 
        cl_match_stats
),
-- Ranking each match by each stat
ranks AS (
    SELECT 
        match_id,
        DENSE_RANK() OVER (ORDER BY goals_combined DESC) AS goals_rank,
        DENSE_RANK() OVER (ORDER BY shots_combined DESC) AS shots_rank,
        DENSE_RANK() OVER (ORDER BY shots_on_target_combined) AS shots_on_target_rank,
        DENSE_RANK() OVER (ORDER BY big_chances_combined DESC) AS big_chances_rank,
        DENSE_RANK() OVER (ORDER BY expected_goals_combined DESC) AS xg_combined_rank,
        DENSE_RANK() OVER (ORDER BY coefficient_difference ASC) AS coefficient_difference_rank
    FROM 
        cl_match_stats
),
-- Weighting the ranks
weighted_ranks as (
    SELECT 
        b.*,
        ROUND((
            r.goals_rank * 1.6 +
            r.shots_rank * 1.2 +
            r.shots_on_target_rank * 1.3 +
            r.big_chances_rank * 1.5 +
            r.xg_combined_rank * 1.1 +
            r.coefficient_difference_rank * 1.4 
        ) / 8.1, 2) AS weighted_average_rank
    -- I needed the basic stats for the final select statement
    FROM 
        basic_stats b
    -- I need the ranks CTE for weighting
    INNER JOIN 
        ranks r ON b.match_id = r.match_id
    ORDER BY
        weighted_average_rank
)

-- I listed all the columns instead of using *
-- allowing me to place the main answer column second
SELECT 
    match_id,
    DENSE_RANK() OVER (ORDER BY weighted_average_rank ASC) AS entertainment_rank,
    phase,
    home_team,
    away_team,
    goals_combined,
    shots_combined,
    shots_on_target_combined,
    big_chances_combined,
    expected_goals_combined,
    coefficient_difference,
    weighted_average_rank
FROM 
    weighted_ranks;
```

#Output(s)
I wanted to show every column and row, but the table was too large and difficult to read. To improve readability, I divided the table into three sections using the dropdown menu technique. The sections are called The Most Entertaining 5, The Rest, and The Least Entertaining 5. Although the Rest section is still quite large, this solution allows me to show every piece of data while making reading easier.

**Click the drop-down triangle before each section's name to view the output(s) you want.**

**The ranking of all 24/25 UEFA Champions League matches by entertainment (excluding the final):**

<details>
<summary> The Most Entertaining 5 </summary>

| match_id | entertainment_rank |       phase       |        home_team       |        away_team       | goals_combined | shots_combined | shots_on_target_combined | big_chances_combined | expected_goals_combined | coefficient_difference | weighted_average_rank |
|:--------:|:------------------:|:-----------------:|:----------------------:|:----------------------:|:--------------:|:--------------:|:------------------------:|:--------------------:|:-----------------------:|:----------------------:|:---------------------:|
|    93    |          1         |      Round 6      |       Atalanta BC      |     Real Madrid CF     |        5       |       30       |            15            |           9          |           5.43          |            0           |           7           |
|    112   |          2         |      Round 7      |      S.L. Benfica      |      FC Barcelona      |        9       |       32       |            16            |          13          |           6.97          |           0.9          |          7.31         |
|    187   |          3         |     Semi-final    |       Inter Milan      |      FC Barcelona      |        7       |       35       |            17            |          10          |           5.09          |           0.2          |          7.33         |
|    159   |          4         | Knockout play-off |      PSV Eindhoven     |       Juventus FC      |        4       |       40       |            14            |          11          |           5.16          |           0.8          |          7.4          |
|    125   |          5         |      Round 7      | Paris Saint-Germain FC |  Manchester City F.C.  |        6       |       35       |            15            |          12          |           4.57          |           0.2          |          8.47         |

</details>

<details>
<summary> The Rest </summary>

| match_id | entertainment_rank |       phase       |        home_team       |        away_team       | goals_combined | shots_combined | shots_on_target_combined | big_chances_combined | expected_goals_combined | coefficient_difference | weighted_average_rank |
|:--------:|:------------------:|:-----------------:|:----------------------:|:----------------------:|:--------------:|:--------------:|:------------------------:|:--------------------:|:-----------------------:|:----------------------:|:---------------------:|
|    35    |          6         |      Round 2      | RasenBallsport Leipzig |       Juventus FC      |        5       |       40       |            12            |          10          |           4.57          |           0.5          |          8.51         |
|    31    |          7         |      Round 2      |      S.L. Benfica      |     Atletico Madrid    |        4       |       23       |            10            |           7          |           4.7           |            0           |          9.84         |
|    173   |          8         |    Round of 16    |       LOSC Lille       |    Borussia Dortmund   |        3       |       28       |            12            |          10          |           4.54          |           0.2          |         10.26         |
|    84    |          9         |      Round 5      |        AS Monaco       |      S.L. Benfica      |        5       |       29       |            13            |           8          |           4.73          |          0.55          |          10.3         |
|    96    |         10         |      Round 6      | RasenBallsport Leipzig |       Aston Villa      |        5       |       36       |            14            |           6          |           4.37          |           0.4          |         10.67         |
|    147   |         11         | Knockout play-off |  Manchester City F.C.  |     Real Madrid CF     |        5       |       31       |            12            |           8          |           4.9           |           1.6          |         11.27         |
|    80    |         12         |      Round 5      |       Sporting CP      |      Arsenal F.C.      |        6       |       32       |            17            |           5          |           4.94          |           1.2          |         11.28         |
|    172   |         13         |    Round of 16    |     Liverpool F.C.     | Paris Saint-Germain FC |        1       |       40       |            11            |           7          |           4.22          |          0.65          |          11.3         |
|    63    |         14         |      Round 4      |       Sporting CP      |  Manchester City F.C.  |        5       |       29       |            12            |           8          |           5.1           |           1.7          |         11.51         |
|    188   |         15         |     Semi-final    | Paris Saint-Germain FC |      Arsenal F.C.      |        3       |       30       |            10            |           6          |           4.65          |           1.1          |         11.91         |
|    104   |         16         |      Round 6      |    Borussia Dortmund   |      FC Barcelona      |        5       |       25       |            10            |           9          |           4.77          |           1.4          |         11.99         |
|    61    |         17         |      Round 4      |     Liverpool F.C.     |   Bayer 04 Leverkusen  |        4       |       34       |            12            |           6          |           4.59          |          1.65          |         12.79         |
|    49    |         18         |      Round 3      |      FC Barcelona      |    FC Bayern Munich    |        5       |       23       |             7            |           6          |           3.27          |          0.05          |         14.95         |
|    170   |         19         |    Round of 16    |   Bayer 04 Leverkusen  |    FC Bayern Munich    |        2       |       29       |             8            |           5          |           3.55          |          0.55          |         14.99         |
|    181   |         20         |   Quarter-final   |    Aston Villa F.C.    | Paris Saint-Germain FC |        5       |       31       |            16            |           9          |           3.74          |           1.2          |         15.07         |
|    86    |         21         |      Round 5      |     Bologna FC 1909    |       LOSC Lille       |        3       |       27       |            14            |          10          |           3.61          |           0.6          |         15.16         |
|    182   |         22         |   Quarter-final   |    Borussia Dortmund   |      FC Barcelona      |        4       |       25       |            13            |           6          |           4.3           |           1.8          |         15.72         |
|    28    |         23         |      Round 2      |       Girona FC        |        Feyenoord       |        5       |       21       |             9            |           6          |           4.16          |          1.85          |         16.19         |
|    118   |         24         |      Round 7      | RasenBallsport Leipzig |       Sporting CP      |        3       |       27       |             8            |           7          |           3.24          |           0.6          |         16.38         |
|    72    |         25         |      Round 4      |      VfB Stuttgart     |       Atalanta BC      |        2       |       19       |             5            |           4          |           3.24          |           0.1          |         16.63         |
|    59    |         26         |      Round 4      |       Celtic F.C.      | RasenBallsport Leipzig |        4       |       27       |             8            |           6          |           3.07          |           0.3          |         16.77         |
|    127   |         27         |      Round 8      |    Aston Villa F.C.    |       Celtic F.C.      |        6       |       26       |            12            |           9          |           4.78          |           3.8          |         16.84         |
|     4    |         28         |      Round 1      |        AC Milan        |     Liverpool F.C.     |        4       |       31       |            13            |           5          |           3.7           |           1.7          |         16.88         |
|    70    |         29         |      Round 4      |       Inter Milan      |      Arsenal F.C.      |        1       |       27       |             5            |           4          |           2.99          |           0.1          |         17.27         |
|    183   |         30         |   Quarter-final   |       Inter Milan      |    FC Bayern Munich    |        4       |       35       |            12            |           4          |           2.89          |           0.2          |         17.33         |
|     7    |         31         |      Round 1      |     AC Sparta Praha    |  FC Red Bull Salzburg  |        3       |       22       |             7            |           4          |           3.13          |           0.5          |         17.72         |
|    19    |         32         |      Round 2      |  FC Red Bull Salzburg  |     Stade Brestois     |        4       |       28       |            13            |           5          |           3.64          |           1.8          |         18.04         |
|    143   |         33         |      Round 8      |      VfB Stuttgart     | Paris Saint-Germain FC |        5       |       23       |             8            |           4          |           3.18          |            1           |         18.14         |
|     1    |         34         |      Round 1      |       Juventus FC      |      PSV Eindhoven     |        4       |       28       |            11            |           8          |           3.96          |           2.7          |         18.22         |
|    135   |         35         |      Round 8      |       Juventus FC      |      S.L. Benfica      |        2       |       32       |            10            |           7          |           3.13          |           1.4          |         18.43         |
|    113   |         35         |      Round 7      |     Bologna FC 1909    |    Borussia Dortmund   |        3       |       20       |             8            |           4          |           3.01          |           0.2          |         18.43         |
|    134   |         36         |      Round 8      |       Inter Milan      |        AS Monaco       |        3       |       24       |             7            |           8          |           4.14          |          3.25          |         18.65         |
|    44    |         37         |      Round 3      |     Real Madrid CF     |    Borussia Dortmund   |        7       |       34       |            17            |           9          |           5.13          |           5.5          |         18.67         |
|    24    |         38         |      Round 2      |    Borussia Dortmund   |       Celtic F.C.      |        8       |       25       |            15            |           5          |           4.73          |            4           |         18.78         |
|    56    |         39         |      Round 4      |  Ĺ K Slovan Bratislava |    GNK Dinamo Zagreb   |        5       |       32       |            12            |           4          |           3.35          |            2           |         18.88         |
|    10    |         40         |      Round 1      |    Borussia Dortmund   |     Club Brugge KV     |        3       |       35       |            11            |           4          |           3.18          |           1.8          |         18.96         |
|    89    |         41         |      Round 5      |     Liverpool F.C.     |     Real Madrid CF     |        2       |       25       |            10            |           8          |           3.95          |           2.7          |         19.04         |
|    163   |         42         |    Round of 16    |      PSV Eindhoven     |      Arsenal F.C.      |        8       |       27       |            10            |          10          |           3.29          |           2.5          |         19.25         |
|    115   |         43         |      Round 7      |    FK Crvena Zvezda    |      PSV Eindhoven     |        5       |       34       |            12            |           5          |           3.49          |           2.7          |         19.75         |
|    128   |         44         |      Round 8      |      FC Barcelona      |       Atalanta BC      |        4       |       24       |            11            |           6          |           3.61          |           2.6          |         20.33         |
|    138   |         44         |      Round 8      |      PSV Eindhoven     |     Liverpool F.C.     |        5       |       21       |            10            |           5          |           2.93          |            1           |         20.33         |
|    62    |         45         |      Round 4      |     Real Madrid CF     |        AC Milan        |        4       |       37       |            19            |           6          |           5.11          |            6           |         20.36         |
|    184   |         46         |   Quarter-final   |     Real Madrid CF     |      Arsenal F.C.      |        3       |       30       |             9            |           4          |           3.46          |          2.65          |         20.41         |
|    155   |         47         | Knockout play-off |      S.L. Benfica      |        AS Monaco       |        6       |       25       |            13            |           8          |           3.18          |           2.2          |         20.47         |
|    178   |         48         |   Quarter-final   |    FC Bayern Munich    |       Inter Milan      |        3       |       30       |            11            |           6          |           3.08          |           2.3          |         21.36         |
|    52    |         49         |      Round 3      | RasenBallsport Leipzig |     Liverpool F.C.     |        1       |       30       |            14            |           6          |           3.08          |            2           |         21.54         |
|    166   |         50         |    Round of 16    |      S.L. Benfica      |      FC Barcelona      |        1       |       36       |            13            |           7          |           2.94          |           2.2          |         21.64         |
|    154   |         51         | Knockout play-off |       Atalanta BC      |     Club Brugge KV     |        4       |       37       |            15            |           7          |           4.43          |           5.7          |         21.65         |
|    160   |         52         | Knockout play-off |     Real Madrid CF     |  Manchester City F.C.  |        4       |       27       |            12            |           3          |           3.12          |           2.1          |          21.7         |
|    110   |         53         |      Round 7      |       Atalanta BC      |      SK Sturm Graz     |        5       |       30       |            10            |          12          |           6.13          |          11.8          |         21.75         |
|    150   |         54         | Knockout play-off |        AS Monaco       |      S.L. Benfica      |        1       |       29       |             9            |           2          |           2.56          |           0.5          |         21.84         |
|    54    |         55         |      Round 3      |     BSC Young Boys     |       Inter Milan      |        1       |       39       |             8            |           7          |           4.15          |           5.6          |         21.88         |
|    109   |         56         |      Round 7      |        AS Monaco       |       Aston Villa      |        1       |       22       |             9            |           7          |           2.54          |           0.4          |         21.91         |
|    148   |         57         | Knockout play-off |       Sporting CP      |    Borussia Dortmund   |        3       |       29       |            13            |           3          |           2.51          |           0.3          |         21.96         |
|    176   |         58         |    Round of 16    |     Atletico Madrid    |     Real Madrid CF     |        1       |       27       |            10            |           3          |           2.51          |          0.25          |           22          |
|    65    |         59         |      Round 4      |   FC Shakhtar Donetsk  |     BSC Young Boys     |        3       |       27       |             7            |           4          |           2.91          |          2.05          |          22.1         |
|    99    |         60         |      Round 6      |     Stade Brestois     |      PSV Eindhoven     |        1       |       29       |            10            |           5          |           3.16          |          2.65          |         22.28         |
|    37    |         61         |      Round 3      |        AS Monaco       |    FK Crvena Zvezda    |        6       |       32       |            10            |           7          |           4.34          |          6.45          |         22.47         |
|    179   |         62         |   Quarter-final   |      FC Barcelona      |    Borussia Dortmund   |        4       |       31       |            13            |           8          |           5.14          |          9.15          |         22.57         |
|    185   |         63         |     Semi-final    |      Arsenal F.C.      | Paris Saint-Germain FC |        1       |       21       |             9            |           5          |           2.79          |          1.35          |          22.6         |
|    149   |         64         | Knockout play-off |     Club Brugge KV     |       Atalanta BC      |        3       |       23       |             9            |           3          |           2.58          |           1.1          |         23.26         |
|    42    |         65         |      Round 3      |       Juventus FC      |      VfB Stuttgart     |        1       |       29       |            11            |           3          |           2.77          |           1.8          |         23.28         |
|    119   |         66         |      Round 7      |   FC Shakhtar Donetsk  |     Stade Brestois     |        2       |       19       |             9            |           4          |           2.36          |          0.25          |         23.32         |
|    140   |         67         |      Round 8      |      SK Sturm Graz     | RasenBallsport Leipzig |        1       |       24       |            10            |           5          |           2.81          |           1.9          |         23.43         |
|    144   |         68         |      Round 8      |     BSC Young Boys     |    FK Crvena Zvezda    |        1       |       26       |             6            |           2          |           2.16          |           0.1          |         23.52         |
|    90    |         69         |      Round 5      |      PSV Eindhoven     |   FC Shakhtar Donetsk  |        5       |       47       |            21            |           4          |           3.8           |          5.35          |         23.53         |
|    123   |         69         |      Round 7      |        Feyenoord       |    FC Bayern Munich    |        3       |       38       |             9            |          12          |           4.19          |          8.65          |         23.53         |
|     3    |         70         |      Round 1      |    FC Bayern Munich    |    GNK Dinamo Zagreb   |       11       |       33       |            19            |          10          |           7.95          |          24.85         |          23.6         |
|    73    |         71         |      Round 5      |     AC Sparta Praha    |     Atletico Madrid    |        6       |       28       |            12            |           6          |           3.33          |           4.1          |          23.7         |
|    95    |         72         |      Round 6      |     Club Brugge KV     |       Sporting CP      |        3       |       17       |             6            |           4          |           2.34          |           0.5          |         23.73         |
|    77    |         73         |      Round 5      |    FC Bayern Munich    | Paris Saint-Germain FC |        1       |       29       |            10            |           7          |           3.18          |          3.85          |         23.85         |
|    161   |         74         |    Round of 16    |     Club Brugge KV     |       Aston Villa      |        4       |       18       |             7            |           3          |           2.32          |           0.5          |         23.86         |
|    142   |         74         |      Round 8      |     Stade Brestois     |     Real Madrid CF     |        3       |       37       |            11            |           8          |           4.03          |          6.55          |         23.86         |
|    48    |         75         |      Round 3      |     Atletico Madrid    |       LOSC Lille       |        4       |       19       |             9            |           7          |           3.65          |           4.5          |         23.93         |
|    76    |         76         |      Round 5      |   Bayer 04 Leverkusen  |  FC Red Bull Salzburg  |        5       |       36       |            15            |           7          |           4.89          |          11.8          |         23.95         |
|    98    |         77         |      Round 6      |   FC Shakhtar Donetsk  |    FC Bayern Munich    |        6       |       33       |            13            |           8          |           4.82          |          12.83         |           24          |
|     5    |         78         |      Round 1      |     Real Madrid CF     |      VfB Stuttgart     |        4       |       37       |            15            |           6          |           4.55          |          8.65          |         24.33         |
|    29    |         78         |      Round 2      |   FC Shakhtar Donetsk  |       Atalanta BC      |        3       |       28       |             5            |           6          |           2.89          |          3.45          |         24.33         |
|    55    |         79         |      Round 4      |      PSV Eindhoven     |        Girona FC       |        4       |       32       |            15            |           3          |           2.8           |          2.45          |         24.35         |
|    18    |         80         |      Round 1      |     Stade Brestois     |      SK Sturm Graz     |        3       |       22       |             7            |           6          |           2.57          |          1.85          |          24.4         |
|    167   |         81         |    Round of 16    |    FC Bayern Munich    |   Bayer 04 Leverkusen  |        3       |       20       |             7            |           6          |           3.01          |           3.2          |         24.47         |
|    131   |         82         |      Round 8      |    FC Bayern Munich    |  Ĺ K Slovan Bratislava |        4       |       44       |            13            |          11          |           4.81          |          33.98         |         24.86         |
|    66    |         83         |      Round 4      |     AC Sparta Praha    |     Stade Brestois     |        3       |       26       |             8            |           4          |           2.12          |           0.7          |          24.9         |
|    33    |         83         |      Round 2      |       LOSC Lille       |     Real Madrid CF     |        1       |       19       |             9            |           8          |           3.37          |           4.1          |          24.9         |
|    83    |         84         |      Round 5      |      SK Sturm Graz     |        Girona FC       |        1       |       18       |             6            |           3          |           2.51          |           1.3          |         24.94         |
|    168   |         85         |    Round of 16    | Paris Saint-Germain FC |     Liverpool F.C.     |        1       |       29       |            11            |           4          |           2.05          |           0.3          |         24.98         |
|    136   |         86         |      Round 8      |       LOSC Lille       |        Feyenoord       |        7       |       21       |             8            |           4          |           2.09          |           0.7          |         25.12         |
|    60    |         87         |      Round 4      |       LOSC Lille       |       Juventus FC      |        2       |       17       |             6            |           2          |           2.23          |           0.6          |         25.59         |
|    114   |         88         |      Round 7      |     Club Brugge KV     |       Juventus FC      |        0       |       26       |            12            |           3          |           2.26          |           0.8          |         25.68         |
|    117   |         89         |      Round 7      |  Ĺ K Slovan Bratislava |      VfB Stuttgart     |        4       |       27       |             9            |          11          |           4.09          |          9.55          |         26.21         |
|    158   |         90         | Knockout play-off | Paris Saint-Germain FC |     Stade Brestois     |        7       |       33       |            12            |          10          |           4.21          |          14.65         |         26.27         |
|    81    |         91         |      Round 5      |     BSC Young Boys     |       Atalanta BC      |        7       |       27       |            11            |           5          |           3.35          |           5.6          |         26.31         |
|    145   |         92         | Knockout play-off |     Stade Brestois     | Paris Saint-Germain FC |        3       |       32       |             9            |           7          |           3.68          |           7.6          |         26.56         |
|    23    |         93         |      Round 2      |   Bayer 04 Leverkusen  |        AC Milan        |        1       |       32       |            14            |           3          |           2.82          |           3.5          |         26.58         |
|    69    |         94         |      Round 4      |    FK Crvena Zvezda    |      FC Barcelona      |        7       |       25       |            11            |           6          |           3.94          |          7.75          |         26.67         |
|     9    |         94         |      Round 1      |       Celtic F.C.      |  Ĺ K Slovan Bratislava |        6       |       24       |            13            |           9          |           4.3           |          9.75          |         26.67         |
|    101   |         95         |      Round 6      |       LOSC Lille       |      SK Sturm Graz     |        5       |       28       |            13            |           3          |           3.26          |          5.05          |         26.75         |
|    14    |         96         |      Round 1      |    FK Crvena Zvezda    |      S.L. Benfica      |        3       |       25       |             6            |           2          |           2.29          |            2           |         26.78         |
|    79    |         97         |      Round 5      |  Manchester City F.C.  |        Feyenoord       |        6       |       28       |            14            |           6          |           4.37          |          11.8          |         26.86         |
|     2    |         98         |      Round 1      |     BSC Young Boys     |       Aston Villa      |        3       |       32       |            12            |           1          |           2.63          |          3.35          |         27.21         |
|    169   |         99         |    Round of 16    |      FC Barcelona      |      S.L. Benfica      |        4       |       28       |             8            |           6          |           3.26          |          6.05          |         27.32         |
|    32    |         100        |      Round 2      |    GNK Dinamo Zagreb   |        AS Monaco       |        4       |       16       |             8            |           5          |           3.06          |          4.15          |         27.36         |
|    174   |         101        |    Round of 16    |      Arsenal F.C.      |      PSV Eindhoven     |        4       |       22       |            12            |           7          |           2.8           |          3.95          |         27.54         |
|    17    |         102        |      Round 1      |     Atletico Madrid    | RasenBallsport Leipzig |        3       |       28       |             7            |           4          |           2.46          |           3.2          |         27.59         |
|    132   |         103        |      Round 8      |       Girona FC        |      Arsenal F.C.      |        3       |       23       |             9            |           3          |           2.89          |          4.05          |         27.65         |
|    22    |         104        |      Round 2      |      FC Barcelona      |     BSC Young Boys     |        5       |       26       |             9            |           7          |           4.42          |          19.9          |         27.79         |
|    57    |         105        |      Round 4      |     Bologna FC 1909    |        AS Monaco       |        1       |       17       |             6            |           2          |           1.43          |           0.1          |         27.98         |
|    26    |         106        |      Round 2      |      PSV Eindhoven     |       Sporting CP      |        2       |       27       |             9            |           7          |           1.22          |           0.8          |         27.99         |
|    111   |         107        |      Round 7      |     Atletico Madrid    |   Bayer 04 Leverkusen  |        3       |       18       |             6            |           3          |           1.59          |           0.8          |          28.3         |
|    11    |         108        |      Round 1      |  Manchester City F.C.  |       Inter Milan      |        0       |       35       |             9            |           4          |           3.1           |            6           |         28.38         |
|    177   |         109        |   Quarter-final   |      Arsenal F.C.      |     Real Madrid CF     |        3       |       21       |            14            |           2          |           2.14          |          1.45          |          28.4         |
|    171   |         110        |    Round of 16    |       Inter Milan      |        Feyenoord       |        3       |       26       |             9            |           3          |           3.69          |           8.6          |         28.57         |
|    85    |         111        |      Round 5      |    Aston Villa F.C.    |       Juventus FC      |        0       |       19       |             5            |           2          |           1.61          |           0.8          |         28.63         |
|    94    |         112        |      Round 6      |   Bayer 04 Leverkusen  |       Inter Milan      |        1       |       23       |             5            |           3          |           1.43          |           0.9          |         28.64         |
|    64    |         113        |      Round 4      |     Club Brugge KV     |       Aston Villa      |        1       |       21       |             8            |           2          |           1.8           |           1.1          |         28.73         |
|    53    |         114        |      Round 3      |  FC Red Bull Salzburg  |    GNK Dinamo Zagreb   |        2       |       29       |             7            |           3          |           1.88          |            2           |          28.8         |
|    13    |         114        |      Round 1      |        Feyenoord       |   Bayer 04 Leverkusen  |        4       |       23       |             8            |           3          |           2.52          |           3.5          |          28.8         |
|    20    |         115        |      Round 2      |      VfB Stuttgart     |     AC Sparta Praha    |        2       |       40       |            13            |           5          |           2.71          |           5.3          |         28.94         |
|    175   |         115        |    Round of 16    |    Aston Villa F.C.    |     Club Brugge KV     |        3       |       18       |            11            |           4          |           2.57          |          3.25          |         28.94         |
|    146   |         116        | Knockout play-off |       Juventus FC      |      PSV Eindhoven     |        3       |       27       |            10            |           5          |           2.08          |           2.4          |          29.1         |
|    82    |         117        |      Round 5      |    FK Crvena Zvezda    |      VfB Stuttgart     |        6       |       24       |             9            |           5          |           2.11          |           2.7          |         29.38         |
|     8    |         118        |      Round 1      |     Bologna FC 1909    |   FC Shakhtar Donetsk  |        0       |       21       |             5            |           3          |           2.04          |            2           |         29.52         |
|    141   |         119        |      Round 8      |       Sporting CP      |     Bologna FC 1909    |        2       |       20       |             7            |           2          |           2.52          |           3.7          |         29.84         |
|    68    |         120        |      Round 4      |        Feyenoord       |  FC Red Bull Salzburg  |        4       |       23       |             7            |           2          |           2.59          |           4.4          |         29.88         |
|    139   |         121        |      Round 8      |  FC Red Bull Salzburg  |     Atletico Madrid    |        5       |       22       |            11            |           5          |           2.77          |            5           |          29.9         |
|    130   |         122        |      Round 8      |    Borussia Dortmund   |   FC Shakhtar Donetsk  |        4       |       22       |             9            |           6          |           2.72          |           5.1          |         30.07         |
|    27    |         123        |      Round 2      |  Ĺ K Slovan Bratislava |  Manchester City F.C.  |        4       |       31       |            14            |           8          |           3.82          |          28.95         |         30.42         |
|    71    |         124        |      Round 4      | Paris Saint-Germain FC |     Atletico Madrid    |        3       |       26       |            12            |           5          |           2.79          |          5.45          |         30.59         |
|    50    |         125        |      Round 3      |      S.L. Benfica      |        Feyenoord       |        4       |       30       |            14            |           6          |           2.69          |          5.45          |         30.62         |
|    97    |         125        |      Round 6      |  FC Red Bull Salzburg  | Paris Saint-Germain FC |        3       |       18       |             9            |           9          |           3.33          |          8.75          |         30.62         |
|    180   |         126        |   Quarter-final   | Paris Saint-Germain FC |    Aston Villa F.C.    |        4       |       36       |            12            |           3          |           2.69          |           5.9          |         30.65         |
|    186   |         127        |     Semi-final    |      FC Barcelona      |       Inter Milan      |        6       |       26       |            12            |           3          |           2.26          |           3.8          |         30.72         |
|    156   |         128        | Knockout play-off |    FC Bayern Munich    |       Celtic F.C.      |        2       |       28       |            13            |          10          |           3.5           |          14.65         |         30.88         |
|    74    |         129        |      Round 5      |  Ĺ K Slovan Bratislava |        AC Milan        |        5       |       21       |             8            |           5          |           3.39          |          11.8          |         30.99         |
|    165   |         130        |    Round of 16    |        Feyenoord       |       Inter Milan      |        2       |       21       |            12            |           4          |           2.7           |           4.9          |         31.22         |
|    43    |         131        |      Round 3      | Paris Saint-Germain FC |      PSV Eindhoven     |        2       |       34       |            11            |           6          |           2.82          |           7.2          |         31.27         |
|    46    |         132        |      Round 3      |       Atalanta BC      |       Celtic F.C.      |        0       |       26       |             8            |           3          |           2.64          |           5.1          |         31.35         |
|    122   |         133        |      Round 7      |       Celtic F.C.      |     BSC Young Boys     |        1       |       30       |            11            |           7          |           2.87          |           6.7          |          31.4         |
|    87    |         134        |      Round 5      |       Celtic F.C.      |     Club Brugge KV     |        2       |       16       |             6            |           3          |           2.11          |          3.15          |         31.59         |
|    91    |         135        |      Round 6      |       Girona FC        |     Liverpool F.C.     |        1       |       28       |            12            |           4          |           3.05          |          7.05          |         31.63         |
|    157   |         136        | Knockout play-off |    Borussia Dortmund   |       Sporting CP      |        0       |       26       |             6            |           2          |           2.2           |           3.9          |         31.88         |
|    16    |         137        |      Round 1      |       Atalanta BC      |      Arsenal F.C.      |        0       |       14       |             4            |           4          |           1.98          |           2.6          |         31.89         |
|    133   |         138        |      Round 8      |    GNK Dinamo Zagreb   |        AC Milan        |        3       |       26       |            10            |           3          |           2.3           |           4.4          |         31.96         |
|    137   |         139        |      Round 8      |  Manchester City F.C.  |     Club Brugge KV     |        4       |       29       |            11            |           7          |           3.15          |          13.75         |         31.99         |
|    30    |         140        |      Round 2      |    Aston Villa F.C.    |    FC Bayern Munich    |        1       |       22       |             9            |           2          |           1.95          |          3.05          |         32.48         |
|    106   |         141        |      Round 6      |       Juventus FC      |  Manchester City F.C.  |        2       |       22       |             8            |           3          |           1.39          |          2.35          |         32.49         |
|    51    |         141        |      Round 3      |  Manchester City F.C.  |     AC Sparta Praha    |        5       |       27       |            11            |           5          |           3.16          |          13.85         |         32.49         |
|    116   |         142        |      Round 7      |     Liverpool F.C.     |       LOSC Lille       |        3       |       17       |             8            |           5          |           3.05          |          8.15          |         32.73         |
|    15    |         143        |      Round 1      |        AS Monaco       |      FC Barcelona      |        3       |       22       |             9            |           2          |           1.73          |          3.05          |         32.77         |
|    124   |         144        |      Round 7      |        AC Milan        |        Girona FC       |        1       |       28       |             9            |           5          |           2.25          |           4.8          |         32.94         |
|    126   |         145        |      Round 7      |     Real Madrid CF     |  FC Red Bull Salzburg  |        6       |       23       |             9            |           3          |           3.29          |          18.3          |         32.95         |
|    92    |         146        |      Round 6      |    GNK Dinamo Zagreb   |       Celtic F.C.      |        0       |       18       |             3            |           1          |           0.96          |           1.9          |          33.1         |
|    152   |         147        | Knockout play-off |        Feyenoord       |        AC Milan        |        1       |       23       |             8            |           0          |           1.07          |            2           |         33.19         |
|    40    |         148        |      Round 3      |    Aston Villa F.C.    |     Bologna FC 1909    |        2       |       28       |            10            |           3          |           2.04          |            4           |         33.21         |
|    108   |         149        |      Round 6      |      VfB Stuttgart     |     BSC Young Boys     |        6       |       25       |             7            |           4          |           2.52          |           6.7          |         33.64         |
|    75    |         150        |      Round 5      |      FC Barcelona      |     Stade Brestois     |        3       |       21       |             8            |           5          |           2.93          |          12.85         |         34.58         |
|    36    |         151        |      Round 2      |      SK Sturm Graz     |     Club Brugge KV     |        1       |       21       |             5            |           1          |           1.08          |          3.05          |         34.75         |
|    100   |         152        |      Round 6      |     Atletico Madrid    |  Ĺ K Slovan Bratislava |        4       |       24       |             8            |           7          |           3.01          |          28.95         |         34.89         |
|    88    |         153        |      Round 5      |    GNK Dinamo Zagreb   |    Borussia Dortmund   |        3       |       22       |             7            |           4          |           1.86          |           4.6          |         35.12         |
|    102   |         154        |      Round 6      |      Arsenal F.C.      |        AS Monaco       |        3       |       23       |            10            |           7          |           2.67          |          9.65          |         35.14         |
|     6    |         155        |      Round 1      |       Sporting CP      |       LOSC Lille       |        2       |       21       |             7            |           2          |           2.03          |           4.7          |         35.47         |
|    164   |         156        |    Round of 16    |     Real Madrid CF     |     Atletico Madrid    |        3       |       19       |             9            |           0          |           0.88          |           2.7          |         35.72         |
|    12    |         157        |      Round 1      | Paris Saint-Germain FC |        Girona FC       |        1       |       29       |             6            |           4          |           2.35          |          8.65          |         36.04         |
|    45    |         158        |      Round 3      |      SK Sturm Graz     |       Sporting CP      |        2       |       26       |            11            |           4          |           2.25          |          6.45          |         36.14         |
|    21    |         159        |      Round 2      |      Arsenal F.C.      | Paris Saint-Germain FC |        2       |       16       |             7            |           1          |           1.11          |           3.4          |         36.35         |
|    58    |         160        |      Round 4      |    Borussia Dortmund   |      SK Sturm Graz     |        1       |       28       |             9            |           5          |           2.59          |          13.75         |         36.58         |
|    129   |         161        |      Round 8      |   Bayer 04 Leverkusen  |     AC Sparta Praha    |        2       |       38       |            11            |           4          |           2.5           |          14.88         |         37.05         |
|    107   |         162        |      Round 6      |        AC Milan        |    FK Crvena Zvezda    |        3       |       29       |            14            |           5          |           2.49          |          10.7          |         37.23         |
|    121   |         163        |      Round 7      |      Arsenal F.C.      |    GNK Dinamo Zagreb   |        3       |       22       |             4            |           5          |           2.61          |          23.85         |         37.35         |
|    153   |         163        | Knockout play-off |        AC Milan        |        Feyenoord       |        2       |       24       |             8            |           2          |           2.42          |          9.15          |         37.35         |
|    47    |         164        |      Round 3      |     Stade Brestois     |   Bayer 04 Leverkusen  |        2       |       19       |             6            |           1          |           1.49          |           4.7          |         37.42         |
|    162   |         165        |    Round of 16    |    Borussia Dortmund   |       LOSC Lille       |        2       |       16       |             2            |           1          |           1.15          |          4.55          |         37.69         |
|    151   |         166        | Knockout play-off |       Celtic F.C.      |    FC Bayern Munich    |        3       |       19       |             7            |           5          |           2.07          |          6.55          |         37.72         |
|    25    |         167        |      Round 2      |       Inter Milan      |    FK Crvena Zvezda    |        4       |       26       |            10            |           9          |           2.27          |          14.65         |         38.36         |
|    105   |         168        |      Round 6      |        Feyenoord       |     AC Sparta Praha    |        6       |       26       |            14            |           4          |           1.61          |          6.15          |         38.37         |
|    78    |         169        |      Round 5      |       Inter Milan      | RasenBallsport Leipzig |        1       |       19       |             5            |           3          |           1.52          |            6           |          39.2         |
|    38    |         170        |      Round 3      |        AC Milan        |     Club Brugge KV     |        4       |       24       |            10            |           1          |           1.79          |          7.05          |         39.74         |
|    41    |         171        |      Round 3      |       Girona FC        |  Ĺ K Slovan Bratislava |        2       |       21       |             9            |           2          |           2.16          |          9.75          |          40.1         |

</details>

<details>
<summary> The Least Entertaining 5 </summary>

| match_id | entertainment_rank |       phase       |        home_team       |        away_team       | goals_combined | shots_combined | shots_on_target_combined | big_chances_combined | expected_goals_combined | coefficient_difference | weighted_average_rank |
|:--------:|:------------------:|:-----------------:|:----------------------:|:----------------------:|:--------------:|:--------------:|:------------------------:|:--------------------:|:-----------------------:|:----------------------:|:---------------------:|
|    103   |         172        |      Round 6      |      S.L. Benfica      |     Bologna FC 1909    |        0       |       21       |             6            |           3          |           1.48          |           6.5          |         40.22         |
|    120   |         173        |      Round 7      |     AC Sparta Praha    |       Inter Milan      |        1       |       22       |             6            |           0          |           1.24          |          6.65          |         41.19         |
|    67    |         174        |      Round 4      |    FC Bayern Munich    |      S.L. Benfica      |        1       |       25       |            10            |           1          |           1.53          |          7.75          |         41.25         |
|    34    |         175        |      Round 2      |     Liverpool F.C.     |     Bologna FC 1909    |        2       |       21       |             9            |           2          |           1.8           |          11.8          |         42.35         |
|    39    |         176        |      Round 3      |      Arsenal F.C.      |   FC Shakhtar Donetsk  |        1       |       22       |             6            |           3          |           1.12          |          14.85         |         44.67         |

</details>

# Conclusion

By the numbers, the Round 6 match between Atalanta and Real Madrid was the most entertaining.

However, there is more to entertainment and quality in football than stats. Managerial decisions and tactical patterns matter too.

The Inter and Barca semi-final rightly received widespread praise. In this analysis, the return fixture ranked 3rd, while the first leg ranked 127th.

Neither the plaudits nor these rankings are wrong.

Numbers need context. Unquantifiable concepts need numbers.








