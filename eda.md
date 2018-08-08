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

![an image alt text]({{ site.baseurl }}/imgs/Picture1.png)
Figure 1. The first 5 rows of our DataFrame object. The column names are the official government accounts, and the rows are the followers, in order of descending recency. Notice that the first follower of the first column (@GeneralJohnK) is @ChrisEnerIdeas, which was the account we used to send requests to the Tweepy API.


Our next step involved using the Tweepy API again, this time requesting the 200 most recent tweets for each of the 3,200 followers. Our search terms and parameters are shown in Figure 2 below.

![an image alt text]({{ site.baseurl }}/imgs/Picture2.png)
Figure 2. The search terms and parameters we used to get the 200 most recent tweets for each ‘user’ in our list of 3,200 followers, ‘userlist’. It was not easy to figure out how to collect all the tweets from all the users into a single file. Our solution was to create 3,200 files and stitch them back up together afterwards.

After spending too much time trying to get all the tweets of all 3,200 followers placed into one file in a single run, we decided it would be easier to put together 3,200 files (one for each follower) by iteration and Pandas. Figure 3 shows the code we used to do this.


![an image alt text]({{ site.baseurl }}/imgs/Figure3.png)
Figure 3. The code that created a Pandas DataFrame object containing our raw dataset.

The resulting DataFrame object is shown in Figure 4. At this point, our raw dataset consisted of 107,158 observations or Tweet objects. This concluded our acquisition of raw data to use as features in our machine learning model.

![an image alt text]({{ site.baseurl }}/imgs/Figure4.png)
Figure 4. Our raw feature dataset. Each row is a Tweet object.


Our next mission was to acquire response variable data. As we mentioned in Milestone #2, we will be deriving our “true” response values from Botometer scores. Please note that the Botometer API is a government funded project and we determined it to be a relatively reliable source for benchmarking our model predictions.

We used the Botometer API to obtain scores for each follower in our list. It should be noted that the Botometer API responds with many different scores. Figure 5 shows an example of a Botometer API response, where the highlighted score is the one we chose to use for our project.

![an image alt text]({{ site.baseurl }}/imgs/Figure5.png)
Figure 5. Example of a Botometer API response. There are categories and sub-categories of scores to choose from. Our research into the meaning behind each score led us to choose the score that is highlighted.

After jumping through some hurdles, we were able to compile a DataFrame with the screen name of the followers in our list in one column, and their corresponding score in another column.

We realized at this point that a significant portion of our list of followers had scores that were NaN values. Upon further examination, we determined that a NaN score is given to Twitter accounts that have never tweeted. Since a Twitter bot that does not tweet is unable to bear malice toward society, it would not be a bot worth detecting. Therefore, we decided to assume that followers with NaN scores are all humans.

Now, a Botometer score that is not a NaN can take on continuous values between 0 and 1, and it is a measure of how likely it is that the associated Twitter account is a bot. We decided to derive our response variable as being equal to 1 if the Botometer score is greater than 0.5, and equal to 0 otherwise. Creating a new column called “bot_or_not”, our Botometer score DataFrame took on the form as shown in Figure 6.

![an image alt text]({{ site.baseurl }}/imgs/fig6.png)
Figure 6. Our Botometer dataset which contains the column, “bot_or_not”, indicating whether a ‘follower’ is a bot or not.
1.1.2	Issues Encountered
1)	The Tweepy API would only take 600 requests at a time. This is why we ended up with 600 followers for @nikkihaley. For the rest of the official government accounts, we requested 200 followers each, for three accounts on each run.
2)	The Botometer API took a really long time to get, largely due to the slow response time for each request. We received an e-mail notification from Rapid API, the provider of the Botometer API, stating that 85% of our free subscription had been consumed. This led us to be more mindful of our limited resources, and eventually we settled on 3,200 scores.

![an image alt text]({{ site.baseurl }}/imgs/fig7.png)
Figure 7. Notice from RapidAPI, the Botometer API service provider, suggesting that our rate of API calls was on par with a successful business.

This concludes the part on our data collection.
