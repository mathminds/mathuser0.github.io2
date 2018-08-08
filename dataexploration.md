---
layout: Page
title: Data Exploration
permalink: /data_exploration/

---


1.2	Data Exploration  
The following describes our decision-making process in preparing a dataset suitable for machine learning. Please kindly note that some of the decisions were made based on limited resources and time constraints. We hope to be able to explore further and extract more features from our dataset for the final project report.

1.2.1	Initial Explorations  
Each row in our feature dataset is a Tweet object. Some of the columns have values that are nested dictionaries, and, in some cases, these nest dictionaries contain keys whose values are again nested dictionaries. Note that it is often the case that a Tweet object contains many NaN values, and in some cases, a column value is itself another nested Tweet object. In Table 1 below, the main features that we extracted and what they represent are described.


Column Name	|Description
created_at	|Time the tweet was created
id	|ID of this Tweet
in_reply_to_screen_name	|If this Tweet is a reply, contains original Tweet's author
lang	|Machine-detected language of the Tweet text
possibly_sensitive	|Indicates that this Tweet contains a link
source	|Utility used to post this Tweet
is_quote_status	|Indicates whether this is a quoted Tweet
retweet_count	|Number of times this Tweet has been retweeted
favorite_count	|Indicates how many times this Tweet has been liked by Twitter users
truncated	|Indicates whether the value of the text parameter was truncated
text	|The text content of the Tweet

Table 1. Columns that are of interest to us and their content descriptions.


Of the columns that contain nested dictionaries, we decided to only look at the ‘user’ and ‘metadata’ columns for now. If time permits, we would like to further explore using the values in the nested dictionaries of columns ‘coordinates’, ‘place’, ‘entities’, ‘extended_entities’ for the additional information they provide, as well as the columns ‘retweeted_status’ and  ‘quoted_status’ which contain Tweet objects if the current Tweet is a retweet or quote tweet, respectively.


The keys that we chose to extract from the nested dictionary in the ‘user’ column are listed in Table 2 along with their description.

|Column Name|	Description|
id_str	|String representation of the unique identifier for this User
name	|Display name of user
screen_name	|Screen name of user
location	|user-defined location for this account
url	|URL provided by the user in association with their profile
description	|User-defined UTF-8 string describing their account
verified	|When true, indicates that the user has a verified account
followers_count	|Number of followers this account currently has. Under duress may display 0
friends_count	|Number of users this account is following
listed_count	|Number of public lists that this user is a member of
favourites_count	|Number of Tweets this user has liked in the account’s lifetime
statuses_count	|Number of Tweets (including retweets) issued by the user
created_at	|UTC datetime that the user account was created on Twitter
geo_enabled	 |When true, indicates that the user has enabled the possibility of geotagging their Tweets
lang	 |User’s self-declared user interface language
profile_image_url_https	 |A HTTP-based URL pointing to the user’s profile image

Table 2. Information contained in the nested dictionaries in the ‘user’ column. These are the keys that we added to our features dataset.

Because some of the column names in Table 2 coincide (in terms of column name) with columns names in the original Tweet object, the string “user_” was added to the front of the column names of the columns that we got from the nest dictionaries in ‘user’. In the process of adding the ‘user’ columns to our dataset, there were two error entries identified which were of indices 103723 and 103724. Both of the errors occurred due to the ‘user’ values being NaN. Both rows were dropped, and the dataset was reduced from 107,158 rows to 107,156 rows. At this point, the columns in our dataset were:

'created_at', 'id', 'in_reply_to_screen_name', 'lang', 'possibly_sensitive', 'source', 'is_quote_status', 'retweet_count', 'favorite_count', 'truncated', 'text', ‘user_id_str’, ‘user_name’, ‘user_screen_name’, ‘user_location’, ‘user_url’, ‘user_description’, ‘user_verified’, ‘user_followers_count’, ‘user_friends_count’, ‘user_listed_count’, ‘user_favouries_count’, ‘user_statuses_count’, ‘user_created_at’, ‘user_geo_enabled’, ‘user_lang’, ‘user_profile_image_url_https’

Since we are dealing with the ‘english’ Botometer score, it made sense to remove all rows that were not in English. Selecting only the rows that had ‘lang’ column value equal to ‘en’ and dropping the others reduced the number of rows further to 91,423 rows. Then, selecting only the rows that had ‘user_lang’ column values of ‘en’ further reduced the number of rows to 89,306 rows. We created a function to do this which is shown in Figure 8.


Figure 8. Function created to select certain rows. The function was used to get the rows that have ‘en’ as their ‘lang’ value, and then it was used again to get the rows that have ‘en’ as their ‘user_lang’ value. Note that ‘lang’ is a column of the Tweet object , whereas ‘user_lang’ was obtained from the nested dictionary within the ‘user’ column of the Tweet object. Notice how they are not necessarily the same.

Next, using functions we created, the columns that contained a single value were identified and dropped as can be seen in Figure 9.


Figure 9. Identification and dropping of single valued columns.

Then, the columns for which more than half the values were NaNs were dropped as can be seen in Figure 10.


Figure 10. Identifying and dropping columns with too many Nan values.

Finally, we were ready to add the response variable column to our dataset. The plan was, for each row, get the author’s Twitter screen name (from the ‘user_screen_name’ column), find the corresponding ‘bot_or_not’ value in our Botometer dataset, and add it to our feature dataset in a new column also named ‘bot_or_not’.

Unfortunately, we encountered an error during this process, which we found to be because of duplicate entried in the Botometer dataset. It turns out there were people who were followers of more than one of the official government accounts in our list. This led to them being included in our Botometer dataset multiple times, and as a result Pandas had a hard time deciding which one of the multiple rows to use. It was somewhat surprising to find out that Pandas couldn’t decide between the duplicates, even though they all had the same ‘bot_or_not’ value.

The fix for this issue turned out to be pretty simple. We removed the duplicate rows from the Botometer dataset by using the method ‘drop_duplicates()’ to get rid of the error.

Another error we encountered were users in our feature dataset that were not listed in our Botometer dataset. The rows for these users were unable to obtain a ‘bot_or_not’ classification, thus resulting in errors. These users were identified to be ‘jcarrhc11’, ‘DreaminDanielle’, ‘NicholasHaddad96’, and ‘PolitEvanRonda’. The rows associated with these accounts were deleted, bring down the number of rows to 89,111.

At this point, we figured our feature dataset must have duplicates too, and got rid of them as well, which brought our final number of rows to 82,243.
 
1.2.2	Data Cleaning  
We decided that, for our Milestone 3 report, we would only include numerical predictors, which were not that many in our dataset at this point. We decided to get more numerical predictors.

1.2.2.1	Basic Feature Extraction from Text  
To extract numerical features from the ‘text’ column, we did some basic feature extraction by referring to the following url: https://www.analyticsvidhya.com/blog/2018/02/the-different-methods-deal-text-data-predictive-python/.

 The following features were extracted by applying basic feature extraction methods to the ‘text’ column values. No other columns were involved.

•	word_count – number of words in the Tweet text
•	char_count – number of characters in the Tweet text
•	avg_word – average word length in the Tweet text
•	stopwords – number of stopwords in Tweet text
•	num_hashtags – number of hashtags in Tweet text
•	numerics – number of numerics in Tweet text
•	upper – number of upper case words in Tweet text

It is worth noting that we had to install/import the nltk module, and then download the stopwords corpus in order to create the ‘stopwords’ column. The code we used to extract these features are shown in Figure 11 below.


Figure 11. Basic feature extraction on the 'text' column of our feature dataset.

 
1.2.2.2	Pre-Processing of Text Data for Advanced Feature Extraction
In order to do text analysis, it is necessary to pre-process the text data, so that more meaningful information can be extracted. The following procedures were done in order to prepare the text data for advanced feature extraction.

•	Convert all words to lower case only
•	Remove punctuation
•	Remove stopwords
•	Lemmatization (convert to root word)

The code we used to do this is shown in Figure 12.



Figure 12. Pre-processing of text data to prepare it for advanced feature extraction.


It should be noted that we wanted to do spelling correction too, but due to this procedure being very slow, we skipped it for our Milestone 3 report. However, we plan to do spelling correction for our final report. Our research into this procedure seems to suggest that it can have a significant impact on the quality of the extracted advanced features.
 
1.2.2.3	Advanced Feature Extraction
For Milestone 3, we chose to include two advanced feature columns: ‘sentiment_polarity’ and ‘sentiment_subjectivity’. These columns were obtained using the sentiment() method of the textblob module as shown in Figure 13.



Figure 13. Sentiment predictors added to our baseline dataset.


For the record, we also succeeded in creating a TF-IDF matrix of 1,000 words, but we decided not to use it for our baseline model because it added too many columns and we were unsure about how to utilize it. We plan to look further into making use of TF-IDF matrix in our final report. Figure 14 shows the code we used to get our TF-IDF matrix, and Figure 15 shows the TF-IDF matrix we were able to create.



Figure 14. Code we used to create a TF-IDF matrix on our dataset.



Figure 15. The TF-IDF matrix we generated. Notice how most of the values are 0. The highlighted value in row=2, col=’poll’ indicates that the TF-IDF matrix was indeed correctly made.


1.2.3	Reconciliation
There were several things we had to postpone mostly due to time constraints, but some also due to computational limits. TF-IDF is a feature that we really want to include in our final report. Our current and very limited understanding is that the TF-IDF is a sparse matrix that carries in each row information about the significance of a word in the Tweet in relation to the entire corpus of Tweets in our dataset. Further research will reveal how it will be best applied to our model.

Another advanced feature we found interesting yet unable to include in this report is Word2Vec. We have downloaded the Stanford Glove Twitter Word2Vec libraries and look forward to utilizing it in the next two weeks.
