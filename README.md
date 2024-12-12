### The Game Plan
---

##### **Objective**:
Automate process of pulling current and historical NFL data at game level from stathead for mySQL analysis, Tableau visualizations, and python predictions.
- *Note for this to work you'll need:*
    1. Your own stathead account at ~5-10/mo for at least NFL stats otherwise you can't pull data in mass.
    2. When connecting to mySQL in the last step, the tables need to already be premade in your mySQL database. Typically after the second phase, I'll pull in the file from the csv exported into mySQL and create the table with the wizard. You can then drop the data or keep it. This now gives you a framework to connect to for step 3.

##### **Data Description**: 
- Subject to change but currently I am pulling 3 categories of data. They are team, opponent and advanced level statistics aggregated at game level.
    1. *Team:* Perspective of who you are rooting for to win. The data contains offense, special teams, vegas odds, some defense and opponent stats
    2. *Opponent:* Perspective of who you are rooting against. The data contains very similar stats as the team tables, however, for the opponents. These stats are important for gaging defensive performance of the team
    3. *Advanced:* Statheads own in-house special statistics on the offensive and defensive side of teams. For the purposes of predictions
##### **Process Overview**: 
- Below is a brief description of what is happening behind the scenes. Instead of Eveything occuring simultaneously, I've split it into 3 phases.
    1. The first phase calls a script I created titled `stathead_individual_game_crawl`. Here it will loop through all possible pages of data for the below 3 url types and pull till there is no more pages to pull data from. The downloads are sent to a specified folder as HTML files.
    2. The second phase is titled `stathead_HTML_to_CVS_export_and_cleanse.` Here the HTML files downloaded from stathead are imported back into python, combined into one table for each set of data (team, opponent, advanced), nulls are updated to 0's, finally each set of data is exported into separate CSVs.
    3. The last phase is titled `stathead_csv_to_mySQL_export` once the CSVs are cleaned in your specified folder they are then reimported one last time into a dataframe. A connection is then made to mySQL where the individual dataframes are then added into their own pre-made tables. 

---

#### Website Examples (Copy to see structure)

- **Team URL**: [Stathead - Individual Team Game Finder](https://stathead.com/football/team-game-finder.cgi?request=1&order_by=date&team_game_max=17&year_max=2024&comp_type=reg&week_num_season_max=22&timeframe=seasons&week_num_season_min=1&team_game_min=1&year_min=2021&match=team_game&cstat[1]=points&ccomp[1]=gt&cval[1]=0&cstat[2]=pass_cmp&ccomp[2]=gt&cval[2]=0&cstat[3]=rush_att&ccomp[3]=gt&cval[3]=0&cstat[4]=tot_yds&ccomp[4]=gt&cval[4]=0&cstat[5]=penalties&ccomp[5]=gt&cval[5]=0&cstat[6]=first_down&ccomp[6]=gt&cval[6]=0&cstat[7]=all_td_team&ccomp[7]=gt&cval[7]=0&cstat[8]=punt&ccomp[8]=gt&cval[8]=0&cstat[9]=kick_ret_td&ccomp[9]=gt&cval[9]=0&cstat[10]=vegas_line&ccomp[10]=gt&cval[10]=-500&offset=0)

- **Opponent URL (IP)**: [Stathead - Individual Opponent Game Finder](https://stathead.com/football/team-game-finder.cgi?request=1&team_game_max=17&week_num_season_min=1&timeframe=seasons&match=team_game&year_max=2024&order_by=date&year_min=2021&team_game_min=1&week_num_season_max=22&comp_type=reg&cstat[1]=pass_cmp_opp&ccomp[1]=gt&cval[1]=0&cstat[2]=rush_att_opp&ccomp[2]=gt&cval[2]=0&cstat[3]=tot_yds_opp&ccomp[3]=gt&cval[3]=0&cstat[4]=first_down_opp&ccomp[4]=gt&cval[4]=0&cstat[5]=all_td_opp&ccomp[5]=gt&cval[5]=0&cstat[6]=def_tgt_yds_per_att&ccomp[6]=gt&cval[6]=0&offset=0)

- **Advanced URL**: [Stathead - Individual Advanced Game Finder](https://stathead.com/football/team-game-finder.cgi?request=1&year_min=2021&match=team_game&comp_type=reg&team_game_max=17&team_game_min=1&week_num_season_min=1&year_max=2024&timeframe=seasons&order_by=date&week_num_season_max=22&cstat[1]=pass_target_yds&ccomp[1]=gt&cval[1]=-600&cstat[2]=pass_batted_passes&ccomp[2]=gt&cval[2]=-600&cstat[3]=pocket_time&ccomp[3]=gt&cval[3]=-600&cstat[4]=pass_rpo&ccomp[4]=gt&cval[4]=-600&cstat[5]=rec_yac&ccomp[5]=gt&cval[5]=-600&cstat[6]=rush_yds_before_contact&ccomp[6]=gt&cval[6]=-600&cstat[7]=rush_yds_diff&ccomp[7]=gt&cval[7]=-600&offset=0)

    - **Pattern**: The `offset` value increments by **200**. The code will increase the `offset` of the url during the looping process to capture all pages of data and exit the loop once a page has no more data to pull.

---

#### Criteria for Data Pull

- **Seasons**: 2021-2024, Regular Season.
- **Team Stats Conditions**:
  - Penalties ≥ 0
  - Rushing Att ≥ 0
  - Kickoff Return TD ≥ 0
  - Total Yardage ≥ 0
  - Points For ≥ 0
  - Punts ≥ 0
  - First Downs ≥ 0
  - Passes Completed ≥ 0
  - Point Spread ≥ -500
  - Touchdowns ≥ 0
  - Sorted By DESC date

- **Opponent Stats Conditions (IP)**:
    - DADOT >= 0
    - Opponent 1st Downs >= 0
    - Opponent Touchdowns >= 0
    - Opponent Total Yards >= 0
    - Opponent Rushing Attempts >= 0
    - Opponent Passes Completed >= 0
  
- **Advanced Stats Conditions**:
    - Rushing Yards Margin >= -600
    - RPO Plays >= -600
    - Yards After Catch >= -600
    - Passes Batted >= -600
    - Average Pocket Time >= -600
    - Rushing Yards Before Contact** >= -600
    - Intended Air Yards >= -600

#### Reminders
*Make sure when pulling data from each category the row count comes out the same upon import into mySQL. Since null values are updated to 0 and these stats on stathead are aggregated at game level, there should not be discrpancies since every team plays the same amount of games every year and my criteria shouldn't exclude any particular game.*

---
