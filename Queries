--1. For each decade, how many schools produced MLB players?

select floor(yearid/10) * 10 as decade, count(DISTINCT schoolid) as num_schools
from schools
group by decade
order by decade;

--2. What are the top 5 schools that produced the most players? 

select sd.name_full, count(DISTINCT p.playerid) as num_players
from players p 
join schools s
on p.playerID = s.playerID
join school_details sd on s.schoolID = sd.schoolID
group by sd.name_full
order by num_players desc
limit 5;

--3. Per decade, what are the top 3 schools that produced the most players?

with school_rank as (select floor(s.yearid/10) * 10 as decade, name_full, count(DISTINCT s.playerid) as num_players
from schools s 
join school_details sd on s.schoolID = sd.schoolID
group by decade, name_full
order by decade),
 
final_rank as (select decade, name_full, num_players, row_number() over(partition by decade order by num_players desc) as school_sort
from school_rank)
 
select decade, name_full, num_players 
from final_rank
where school_sort <= 3;


--4. Return the top 20% of teams in terms of average annual spending.

with spend as (select teamid, yearid, sum(salary) as total_spend_per_yr			                                                      --Sums total spend per team per year
from salaries
group by teamid, yearid),
 
pct_spend as (select teamid, round(avg(total_spend_per_yr),2) as avg_spend,		                                                    --Determines average annual spend
          ntile(5) over(order by avg(total_spend_per_yr) desc) as percent		                                                      --Ranks teams 1-5 based off average annual spend
from spend
group by teamid)
 
select teamid, avg_spend
from pct_spend
where percent = 1;		--Selects top 20% of teams	

--5. For each team, calculate the cumulative sum of spending over the years.

with salary_sum as (select yearid, teamid, sum(salary) as total_spend_per_year                                                     --Calculates total spend per team per year
from salaries
group by yearid, teamid)
 
select teamid, yearid, total_spend_per_year, 
                       sum(total_spend_per_year) over(partition by teamid order by teamid, yearid) as cumulative_sum               --Calculates cumulative spend per team per year
from salary_sum;	

--6. Return the first year that each teams cumulative spending surpassed $1 billion.

with salary_sum as (select yearid, teamid, sum(salary) as total_spend_per_year                                                      --Calculates total spending per team per year
from salaries
group by yearid, teamid),
 
cum_spending as (select yearid, teamid, total_spend_per_year, 
                      sum(total_spend_per_year) over(partition by teamid order by teamid, yearid) as cumulative_sum
from salary_sum), 																                                                                                   --Calculates cumulative spending over the years
 
rank_spending as (select yearid, teamid, cumulative_sum, row_number() over(partition by teamid order by cumulative_sum) as rank
from cum_spending
where cumulative_sum > 1000000000) 												                                                                           --Ranks spending by cumulative sum only over $1 billion dollars
 
select yearid, teamid, cumulative_sum											                                                                           --Returns only the first value over $1 billion dollars
from rank_spending
where rank = 1;

--7. For each player, calculate their age at their debut game, their last game, and their career length (years). Sort from longest career to shortest.

with bday as (select nameGiven, debut, finalgame, concat(birthyear,'-',birthmonth,'-',birthday) as birthdate
from players)
 
select namegiven, debut - birthdate as debut_age, finalgame - birthdate as final_game_age, finalgame - debut as career_length
from bday
order by career_length desc;

--8. Which team did each player play on during their starting and ending years?

with ranking as (select p.namegiven, s.yearid, s.teamid, 												                                                      --Ranks the teams by year by player using window function
                 row_number() over(partition by p.nameGiven order by s.yearid asc) as starting_rank, 
row_number() over(partition by p.nameGiven order by s.yearid desc) as ending_rank
from salaries s
join players p on p.playerid = s.playerid
order by p.nameGiven, s.yearid)
 
select nameGiven,																						                                                                           --Returns each starting & ending team per player
max(case when starting_rank = 1 then teamid end) as starting_team,
max(case when ending_rank = 1 then teamid end) as ending_team
from ranking
group by namegiven
order by namegiven;

--9: Which players started and ended on the same team and also played for over a decade? 

with team_ranking as (select p.playerID, s.yearid, s.teamid, 												
                      row_number() over(partition by p.playerID order by s.yearid asc) as starting_rank, 
row_number() over(partition by p.playerID order by s.yearid desc) as ending_rank
from salaries s
join players p on p.playerid = s.playerid
order by p.playerID, s.yearid),
 
select_rank as (select playerid,
max(case when starting_rank = 1 then teamid end) as starting_team,
max(case when ending_rank = 1 then teamid end) as ending_team
from team_ranking
group by playerid
order by playerid)
 
select sr.playerid, sr.starting_team, sr.ending_team, p.finalGame - p.debut as career_length			
from select_rank sr																					
join players p on p.playerID = sr.playerid
where starting_team = ending_team 																                                                                      --Filters where a players starting team is the same as their end team
and career_length > 10																		                                                                              --Filters by players who have had a career greater than 10 years
ORDER BY career_length desc;

--10. Select the players that share the same birthday.

with bday as (select date(concat(birthyear, '-', birthmonth, '-', birthday)) as birthdate, namegiven --creates birthday in date format
              from players)
              
select birthdate, group_concat(namegiven, ', ') as players_list										                                                      --Groups the players together with a delimiter
from bday
where birthdate is not NULL																			                                                                        --Filters out players missing a full birthday
group by birthdate
order by birthdate;
							
