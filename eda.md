---
layout: page
title: "Exploratory Data Analysis"
permalink: /eda/



---
# Exploratory Data Analysis
## Data Collection
Our feature dataset was collected as follows.

First, we selected the following 14 official government accounts on Twitter:

 1)	Homeland Security (@DHSgov) - Department of Homeland Security

 2)	USTR (@UStraderep) - Office of the U.S. Trade Representative

 3)	Energy Department (@ENERGY) – U.S. Department of Energy
 4)	Donald Trump (@realDonaldTrump) - 45th President
 5)	Barack Obama (@BarackObama) - 45th President
 6)	Robert S. Mueller (@BobSMueller) – Head of Special Counsel investigation
 7)	John Kelly (@GeneralJohnK) – White House Chief of Staff
 8)	Ben Carson (@SecretaryCarson) – 17th Secretary of Housing & Urban Development
 9)	Rick Perry (@Secretaryperry) – 14th Secretary of Energy
10)	Alex Acosta (@SecretaryAcosta) – 27th Secretary of Labor
11)	Kirstjen Nielsen (@SecNielsen) – 6th Secretary of Homeland Security
12)	Alex Azar (@SecAzar) – 24th Secretary of Health & Human Services
13)	Linda McMahon (@SBALinda) – 25th Administrator of SBA
14)	Nikki Haley (@nikkihaley) – 29th U.S. Ambassador to the United Nations

Using the Tweepy API, we queried for the screen names of the 200 most recent followers for each of the accounts in the list above except @nikkihaley, for which we acquired the screen names of the 600 most recent followers. In total, we gathered 3,200 screen names of Twitter users who have shown interest in government affairs.


Figure 1. The first 5 rows of our DataFrame object. The column names are the official government accounts, and the rows are the followers, in order of descending recency. Notice that the first follower of the first column (@GeneralJohnK) is @ChrisEnerIdeas, which was the account we used to send requests to the Tweepy API.


Our next step involved using the Tweepy API again, this time requesting the 200 most recent tweets for each of the 3,200 followers. Our search terms and parameters are shown in Figure 2 below.


Figure 2. The search terms and parameters we used to get the 200 most recent tweets for each ‘user’ in our list of 3,200 followers, ‘userlist’. It was not easy to figure out how to collect all the tweets from all the users into a single file. Our solution was to create 3,200 files and stitch them back up together afterwards.

After spending too much time trying to get all the tweets of all 3,200 followers placed into one file in a single run, we decided it would be easier to put together 3,200 files (one for each follower) by iteration and Pandas. Figure 3 shows the code we used to do this.
