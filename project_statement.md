---
layout: Page
title: Project Statement
permalink: /project_statement
---

Our previous project statement and goals from Milestone 2 are listed in italic font below, with our updated revisions and/or comments typed in bold font following the arrow.

1.	Our dataset will consist of two main parts: a set of tweets by known bots and a set of tweets by known human accounts.
 Our current dataset assumes a user to be a bot if its Botometer score is greater than 0.5.  In reality, we do not know with 100% certainty whether the users in our dataset are bot or humans.

2.	The set of tweets by known bots will be acquired as the tweets originated from the users in the list of known twitter bots provided at the following URL. https://gist.github.com/Plazmaz/da7cd6d427718c104a6791d250aab946#file-bots-txt
 The list of Twitter bots provided at the URL above contained several deleted accounts, and the accuracy of the list was deemed to be questionable. So, instead of choosing users from the URL above, we acquired the tweets of users that recently began following select official government Twitter accounts. The Botometer score for these users are assumed to be reliable for discerning between Twitter bots and humans.

3.	In the case that this is not feasible due to limited resources, we will collect at least 30,000 bot tweets to use in our data analysis.
 We were able to acquire 107,158 Tweets initially for our raw dataset. Subsequent data cleaning and processing revealed erroneous data, duplicate data, and insufficient data issues. After removing handling these issues by removing them from our dataset, we had 82,224 Tweets to train our machine learning model on.

4.	The set of tweets by known human accounts will be collected from the twitter accounts of famous people, and will be collected in equal amount as the set of bot tweets.
 We were able to collect Twitter accounts of famous people, but to broaden our scope of data, and to ensure that Tweets from bots would be included in our dataset as well, we expanded our search to include Tweets from the followers of famous people. Unfortunately, it did not turn out to be the case that the number of bots and humans are equal in our dataset. However, we will explore the possibility of creating a third category for our response variable, representing Twitter accounts for which the determination of their bot-like or human-like nature is inconclusive.

5.	We have access to a Python script which can search for keywords in past tweets as well as new tweets in real-time. This script will be our main tool for acquiring the Twitter dataset.
 We utilized the Tweepy API to create our dataset. We also utilized the Botometer API to generate our response variable.
6.	The tweets consist of various metadata, some of which will be selected as features in our models.  Our dependent variable will be a categorical variable representing one of three options—a human, a bot, or an inconclusive source—as the originator of a tweet.
 Features were selected and extracted for our baseline model from the Tweet data. The dependent variable for our baseline model is an indicator variable with only two options—bot or not. We will look into expanded the number of categories for our response variable in our final report.

7.	We will use the Botometer API which is available at https://github.com/IUNetSci/botometer-python.git to benchmark and improve our models.
 We have been using the Botometer API and derived our “true” response variable values from the Botometer score.

8.	Therefore, the aim of this project is to create models that use machine learning techniques to analyze tweets data to classify the author as a human, bot, or inconclusive source.
 The aim of our project still remains to create machine learning models that can take Tweet data as input and classify the author as human or bot. We will explore the inconclusive option as well for our final report.

9.	We plan to incorporate classification models and natural language processing (topic modeling and sentiment analysis) in our machine learning algorithms.
 Our baseline model incorporates a logistic regression classification model and is trained on predictors such as sentiment polarity and sentiment subjectivity that were extracted via natural language processing techniques. For our final report, we plan to incorporate additional natural language processing features such as TF-IDF matrices and word vectors created using Word2Vec.

10.	Our models will be evaluated by using the Botometer Python API to benchmark and improve our models.
 This will continue to be our main method of model evaluation.


For our final report, we will aim to find a model that has an AUC score greater than 0.7063 (see baseline model performance evaluation in Chapter 3). We plan to compare k-NN classification models of various k-values, Linear Discriminant Analysis and Quadratic Discriminant Analysis models, and we will also explore adding more advanced features as well as look for possible model performance improvement by excluding some of the features used in our baseline model.
Additionally, if we have time to spare, we may try to compare our model predictions with the same type of models produced by commercial machine learning tool providers such as Amazon Web Services. We are curious to see how our self-created models might compare to commercial level models. It would be quite nice to find our logistic regression classification model performing similarly to a logistic regression classification model created on AWS.
