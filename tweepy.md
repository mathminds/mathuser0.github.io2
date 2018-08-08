---
layout: page
permalink: /tweepy
title: Tweepy Code

---



# Chapter 1. Data Collection

## 1.1 Import Libraries and Setup Authentication


```python
import botometer
import numpy as np
import pandas as pd
import json
import sys
import jsonpickle
import os
import tweepy


# from nltk.corpus import stopwords

# Setting up API_KEY and API_SECRET
auth = tweepy.AppAuthHandler("9Gpcxva2RwolHrDLdhRhYlVln", "ylLbTVYjTz6Px2AGV6W662QDKjYDdCuAO3aa6ybStlrCOguK0b")

# Setting up tweepy.API object
api = tweepy.API(auth, wait_on_rate_limit=True, wait_on_rate_limit_notify=True)

# Botometer Account Setup
mashape_key = "3Jb4MhTkKnmshUOMY639XsisVlVGp1SQgbFjsnj42uy0GsYZ1T"
twitter_app_auth = {
    'consumer_key': '9Gpcxva2RwolHrDLdhRhYlVln',
    'consumer_secret': 'ylLbTVYjTz6Px2AGV6W662QDKjYDdCuAO3aa6ybStlrCOguK0b',
    'access_token': '867610424690188289-sA0uIW3KLVMro7I7JJmygJXRDhJMWj6',
    'access_token_secret': 'eoBTKR3dGstSziJ6AXr8sdUWdYSOaJXCsbPPKr3adMBE0',
  }

# Setting up Botometer object for easy retrieval of Botometer scores
bom = botometer.Botometer(wait_on_ratelimit=True,
                              mashape_key=mashape_key,
                              **twitter_app_auth)
```


```python
# Function to get the 'english' Botometer score
def get_bm_score(screen_name):  
    score = bom.check_account('@'+screen_name)['scores']['english']
    return score
```


```python
def toDataFrame(list_of_dictionaries,columns_to_get):

    Dataset=pd.DataFrame()

    for column in columns_to_get:
            Dataset[column]=[dictionary[column] for dictionary in list_of_dictionaries]

    return Dataset
```

# Put twitter accounts into `politician_list` to get their recent followers


```python
politician_list=['DHSgov','UStraderep','ENERGY','realDonaldTrump',
                 'BarackObama','BobSMueller','GeneralJohnK','SecretaryCarson',
                 'Secretaryperry','SecretaryAcosta','SecNielsen','SecAzar',
                 'SBALinda','nikkihaley']

# Get most recent 200 followers
number_of_followers_to_retrieve=200

# Initialize dictionary of followers
follower_dict = dict()

# Iterate over list of politicians
for politician in politician_list:

    follower_list=[]
    for item in tweepy.Cursor(api.followers, id=politician).items(number_of_followers_to_retrieve):
        follower_list.append(item.screen_name)

    follower_dict[politician]=follower_list

fol_df=pd.DataFrame(follower_dict)


```

    Rate limit reached. Sleeping for: 505


# The following cell will get the botometer scores for the followers in the dataframe from above and create a column for them


```python
fol_df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>GeneralJohnK</th>
      <th>realdonaldtrump</th>
      <th>barackobama</th>
      <th>BobSMueller</th>
      <th>SBALinda</th>
      <th>secazar</th>
      <th>secretaryperry</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>ChrisEnerIdeas</td>
      <td>laurieg47</td>
      <td>Emilly_2618</td>
      <td>Melanie6563</td>
      <td>Epiiietaph</td>
      <td>HocketApersil</td>
      <td>BarryJamesCavey</td>
    </tr>
    <tr>
      <th>1</th>
      <td>fbibug</td>
      <td>LanbowW</td>
      <td>DeepakC41236137</td>
      <td>MichielBoland</td>
      <td>asmacadabra1</td>
      <td>tishna3d</td>
      <td>baonhi_nuyen</td>
    </tr>
    <tr>
      <th>2</th>
      <td>MishaJo26609942</td>
      <td>klasique_honour</td>
      <td>PokuMensah3</td>
      <td>4loveofcocco</td>
      <td>Johnson_Mmartin</td>
      <td>AdamAndreassen</td>
      <td>Epiiietaph</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Clint12Michelle</td>
      <td>skylarhamilton7</td>
      <td>cocotsui1</td>
      <td>PonteVedraMan</td>
      <td>RealBillDuke</td>
      <td>anil_shanbhag</td>
      <td>JavierNubia</td>
    </tr>
    <tr>
      <th>4</th>
      <td>bustin1791</td>
      <td>BreGabryela</td>
      <td>skylarhamilton7</td>
      <td>jomac23532643</td>
      <td>LeannTerresa</td>
      <td>isupporttrump45</td>
      <td>paxjak</td>
    </tr>
  </tbody>
</table>
</div>




```python
score_columns = [col+"_follower_scores" for col in fol_df.columns]
for score in score_columns:
    for i in range(len(fol_df)):
        try:
            fol_df[score].at[i]=get_bm_score(fol_df[score[:-16]].at[i])
        except:
            fol_df[score].at[i]=np.nan

fol_df.to_csv("followers"+'_'.join(score_columns)+".csv")            
```

# I ended up with something that looks like this...


```python
fol_df.head(50)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>AmbJohnBolton</th>
      <th>AmbJohnBolton_follower_scores</th>
      <th>HillaryClinton</th>
      <th>HillaryClinton_follower_scores</th>
      <th>SecPompeo</th>
      <th>SecPompeo_follower_scores</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>rbtess2</td>
      <td>0.252653</td>
      <td>SVibani</td>
      <td>NaN</td>
      <td>assetdesignnetw</td>
      <td>0.092867</td>
    </tr>
    <tr>
      <th>1</th>
      <td>GcvC8bLpwBspwKb</td>
      <td>NaN</td>
      <td>b8gOhYbVvMISiP4</td>
      <td>NaN</td>
      <td>zempodzem</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2</th>
      <td>brianeml</td>
      <td>0.135583</td>
      <td>Phiweh44825809</td>
      <td>NaN</td>
      <td>VORKurdstan</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3</th>
      <td>ChesterJasper2</td>
      <td>0.924094</td>
      <td>DJELSAYED1</td>
      <td>NaN</td>
      <td>GcvC8bLpwBspwKb</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4</th>
      <td>jwsocal</td>
      <td>0.236884</td>
      <td>Rafael41673883</td>
      <td>NaN</td>
      <td>ghost2_s</td>
      <td>0.800598</td>
    </tr>
    <tr>
      <th>5</th>
      <td>HB549</td>
      <td>0.860168</td>
      <td>ZahidAh40507722</td>
      <td>NaN</td>
      <td>farmah13</td>
      <td>0.590513</td>
    </tr>
    <tr>
      <th>6</th>
      <td>moviedream</td>
      <td>0.156855</td>
      <td>vaibhavtakale4</td>
      <td>NaN</td>
      <td>ProjectPolitics</td>
      <td>0.421517</td>
    </tr>
    <tr>
      <th>7</th>
      <td>gator4829</td>
      <td>0.057807</td>
      <td>YasHieLAydz</td>
      <td>0.650690</td>
      <td>jonmalta86</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>8</th>
      <td>RoozehK</td>
      <td>0.904074</td>
      <td>Shankar81565657</td>
      <td>0.341219</td>
      <td>SimonAleez</td>
      <td>0.590513</td>
    </tr>
    <tr>
      <th>9</th>
      <td>victorguzun</td>
      <td>0.100309</td>
      <td>limzwezwe</td>
      <td>NaN</td>
      <td>Karla91919363</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>10</th>
      <td>isharamu5678</td>
      <td>0.341219</td>
      <td>kibonekha</td>
      <td>NaN</td>
      <td>ChesterJasper2</td>
      <td>0.924094</td>
    </tr>
    <tr>
      <th>11</th>
      <td>NoCOITUSwCOTUS1</td>
      <td>0.168473</td>
      <td>PDX_Drewber</td>
      <td>0.360652</td>
      <td>jwsocal</td>
      <td>0.236884</td>
    </tr>
    <tr>
      <th>12</th>
      <td>Danhersod</td>
      <td>0.092867</td>
      <td>kalkiforreal</td>
      <td>0.953082</td>
      <td>HB549</td>
      <td>0.860168</td>
    </tr>
    <tr>
      <th>13</th>
      <td>JREscoto</td>
      <td>0.940212</td>
      <td>BuchwaldPer</td>
      <td>NaN</td>
      <td>fleetingscript</td>
      <td>0.911223</td>
    </tr>
    <tr>
      <th>14</th>
      <td>ak_gnosis</td>
      <td>0.100309</td>
      <td>ShoaibB45044079</td>
      <td>NaN</td>
      <td>gator4829</td>
      <td>0.057807</td>
    </tr>
    <tr>
      <th>15</th>
      <td>mab33000</td>
      <td>0.073433</td>
      <td>RamzyNola</td>
      <td>0.975730</td>
      <td>JamesLo59598728</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>16</th>
      <td>videocretorx1</td>
      <td>0.849586</td>
      <td>NathanWhitcomb6</td>
      <td>0.963289</td>
      <td>victorguzun</td>
      <td>0.100309</td>
    </tr>
    <tr>
      <th>17</th>
      <td>mohtasham1984</td>
      <td>NaN</td>
      <td>BentekeSackie</td>
      <td>NaN</td>
      <td>UncommonSense50</td>
      <td>0.108277</td>
    </tr>
    <tr>
      <th>18</th>
      <td>myraj28</td>
      <td>0.286212</td>
      <td>anaveleznegmai1</td>
      <td>NaN</td>
      <td>mardekhast</td>
      <td>0.180767</td>
    </tr>
    <tr>
      <th>19</th>
      <td>AdamGruber5</td>
      <td>0.067835</td>
      <td>mengbowen1017</td>
      <td>0.834413</td>
      <td>mercedehmohseni</td>
      <td>0.286212</td>
    </tr>
    <tr>
      <th>20</th>
      <td>AMAZONJUDE1</td>
      <td>0.421517</td>
      <td>JamesOt69011316</td>
      <td>0.979459</td>
      <td>YakhoubAdoum1</td>
      <td>0.917888</td>
    </tr>
    <tr>
      <th>21</th>
      <td>SHAYAN68781586</td>
      <td>0.971343</td>
      <td>hamzanoor79</td>
      <td>NaN</td>
      <td>obeytimmerx</td>
      <td>0.053331</td>
    </tr>
    <tr>
      <th>22</th>
      <td>ChhunNaty</td>
      <td>0.975730</td>
      <td>haleyta38533072</td>
      <td>0.180767</td>
      <td>WangMariah</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>23</th>
      <td>Schultz08928330</td>
      <td>0.207428</td>
      <td>CarolMartinovi1</td>
      <td>0.973625</td>
      <td>RoozehK</td>
      <td>0.888218</td>
    </tr>
    <tr>
      <th>24</th>
      <td>bruin08usn</td>
      <td>0.944833</td>
      <td>_lolaolutayo</td>
      <td>0.904074</td>
      <td>NoCOITUSwCOTUS1</td>
      <td>0.168473</td>
    </tr>
    <tr>
      <th>25</th>
      <td>Kipnis4Congress</td>
      <td>0.421517</td>
      <td>Cuongse7ven1</td>
      <td>0.904074</td>
      <td>azphilosopher</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>26</th>
      <td>NVWomenForTrump</td>
      <td>0.236884</td>
      <td>Leo19741813</td>
      <td>NaN</td>
      <td>abdelhamideco</td>
      <td>0.669827</td>
    </tr>
    <tr>
      <th>27</th>
      <td>Muhamma42185988</td>
      <td>NaN</td>
      <td>OduyeboT</td>
      <td>0.911223</td>
      <td>Danhersod</td>
      <td>0.092867</td>
    </tr>
    <tr>
      <th>28</th>
      <td>so_flee_to_Alah</td>
      <td>0.953082</td>
      <td>ggeidyb</td>
      <td>0.740529</td>
      <td>MNmurlz</td>
      <td>0.030069</td>
    </tr>
    <tr>
      <th>29</th>
      <td>Valkyerie1</td>
      <td>0.156855</td>
      <td>KING60269603</td>
      <td>NaN</td>
      <td>stealbach</td>
      <td>0.800598</td>
    </tr>
    <tr>
      <th>30</th>
      <td>28585776Enzo</td>
      <td>NaN</td>
      <td>ErikaHe65496370</td>
      <td>NaN</td>
      <td>georgewhittak17</td>
      <td>0.286212</td>
    </tr>
    <tr>
      <th>31</th>
      <td>RG70203818</td>
      <td>0.849586</td>
      <td>loto38156750</td>
      <td>NaN</td>
      <td>Trumpist_MAGA</td>
      <td>0.911223</td>
    </tr>
    <tr>
      <th>32</th>
      <td>Lindsay63513307</td>
      <td>NaN</td>
      <td>iJlnOSGyBuSHZNo</td>
      <td>NaN</td>
      <td>Hellomeauxdeaux</td>
      <td>0.062634</td>
    </tr>
    <tr>
      <th>33</th>
      <td>Athena_OnTheAir</td>
      <td>0.360652</td>
      <td>Can49308972</td>
      <td>0.917888</td>
      <td>JREscoto</td>
      <td>0.940212</td>
    </tr>
    <tr>
      <th>34</th>
      <td>juniorjapa_</td>
      <td>NaN</td>
      <td>IgreementU</td>
      <td>NaN</td>
      <td>ChincoDe</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>35</th>
      <td>johnbra56438373</td>
      <td>0.896414</td>
      <td>chriSmiles125</td>
      <td>0.380552</td>
      <td>organiclover789</td>
      <td>0.669827</td>
    </tr>
    <tr>
      <th>36</th>
      <td>InstituteLogPol</td>
      <td>0.527497</td>
      <td>mzgilda1</td>
      <td>0.949115</td>
      <td>mab33000</td>
      <td>0.073433</td>
    </tr>
    <tr>
      <th>37</th>
      <td>Janice4Trump</td>
      <td>0.168473</td>
      <td>ArmenHovsepya12</td>
      <td>0.879462</td>
      <td>videocretorx1</td>
      <td>0.849586</td>
    </tr>
    <tr>
      <th>38</th>
      <td>BetsyKristine</td>
      <td>NaN</td>
      <td>CathalHardiman</td>
      <td>0.896414</td>
      <td>cassockstamper</td>
      <td>0.896414</td>
    </tr>
    <tr>
      <th>39</th>
      <td>JustTrischa</td>
      <td>0.168473</td>
      <td>Kunwar2155</td>
      <td>0.506192</td>
      <td>chloewa80217964</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>40</th>
      <td>christinavhilt2</td>
      <td>0.949115</td>
      <td>Arif3335577</td>
      <td>0.917888</td>
      <td>Rezafakhari3</td>
      <td>0.800598</td>
    </tr>
    <tr>
      <th>41</th>
      <td>grumpkin2</td>
      <td>0.360652</td>
      <td>AboJana36640063</td>
      <td>NaN</td>
      <td>AMAZONJUDE1</td>
      <td>0.421517</td>
    </tr>
    <tr>
      <th>42</th>
      <td>82Nguyendan</td>
      <td>0.949115</td>
      <td>ImadGhanem8</td>
      <td>NaN</td>
      <td>mohtasham1984</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>43</th>
      <td>RobertUsher19</td>
      <td>0.929867</td>
      <td>brianarcous</td>
      <td>0.953082</td>
      <td>SHAYAN68781586</td>
      <td>0.971343</td>
    </tr>
    <tr>
      <th>44</th>
      <td>adcor1975</td>
      <td>NaN</td>
      <td>innocentmofya1</td>
      <td>NaN</td>
      <td>ChhunNaty</td>
      <td>0.975730</td>
    </tr>
    <tr>
      <th>45</th>
      <td>sephorajan</td>
      <td>0.975730</td>
      <td>7x2SkFKeZoI4Jvs</td>
      <td>NaN</td>
      <td>MarthaEnciso7</td>
      <td>0.252653</td>
    </tr>
    <tr>
      <th>46</th>
      <td>ChrisEnerIdeas</td>
      <td>NaN</td>
      <td>KurtisMcStrange</td>
      <td>0.590513</td>
      <td>NVWomenForTrump</td>
      <td>0.236884</td>
    </tr>
    <tr>
      <th>47</th>
      <td>arulmachany</td>
      <td>0.688418</td>
      <td>liu_jackchen</td>
      <td>NaN</td>
      <td>Muhamma42185988</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>48</th>
      <td>DerendaPlunkett</td>
      <td>NaN</td>
      <td>Mohamed74465471</td>
      <td>0.631055</td>
      <td>Lubertt</td>
      <td>0.800598</td>
    </tr>
    <tr>
      <th>49</th>
      <td>CarlForrest</td>
      <td>0.100309</td>
      <td>YusufAhmadAliy1</td>
      <td>0.740529</td>
      <td>mrkujo50</td>
      <td>0.116795</td>
    </tr>
  </tbody>
</table>
</div>



# Given a list of users, get the 200 most recent tweets by each user and create a json file for each user.


```python
# Creating list of followers
df_users=pd.read_csv("follower_data_7-24.csv", index_col=0)
users_list=list(df_users.follower)
```


```python
for user in users_list:
    searchQuery = 'From:'+user     # Our search term which requests for Tweets from 'user'
    maxTweets = 200                # Get max 200 tweets per 'user'
    tweetsPerQry = 100             # This is the max the API permits
    fName = 'tweets_'+user+'.json' # We stored each user's tweets in a text file. This resulted in 3,200 files...


    sinceId = None
    max_id = -1
    tweetCount = 0
    print("Downloading max {0} tweets".format(maxTweets))
    with open(fName, 'w') as f:
        while tweetCount < maxTweets:
            try:
                if (max_id <= 0):
                    if (not sinceId):
                        new_tweets = api.search(q=searchQuery, count=tweetsPerQry)
                    else:
                        new_tweets = api.search(q=searchQuery, count=tweetsPerQry, since_id=sinceId)
                else:
                    if (not sinceId):
                        new_tweets = api.search(q=searchQuery, count=tweetsPerQry,max_id=str(max_id - 1))
                    else:
                        new_tweets = api.search(q=searchQuery, count=tweetsPerQry,max_id =str(max_id - 1),
                                                since_id=sinceId)
                if not new_tweets:
                    print("No more tweets found")
                    break
                for tweet in new_tweets:
                    f.write(jsonpickle.encode(tweet._json, unpicklable=False) +
                            '\n')
                tweetCount += len(new_tweets)
                print("Downloaded {0} tweets".format(tweetCount))
                max_id = new_tweets[-1].id
            except tweepy.TweepError as e:
                # Just exit if any error
                print("some error : " + str(e))
                break
    print ("Downloaded {0} tweets, Saved to {1}".format(tweetCount, fName))
```

    Downloading max 200 tweets
    Downloaded 100 tweets
    Downloaded 200 tweets
    Downloaded 200 tweets, Saved to tweets_rbtess2.json
    Downloading max 200 tweets
    Downloaded 100 tweets
    Downloaded 200 tweets
    Downloaded 200 tweets, Saved to tweets_brianeml.json
    Downloading max 200 tweets
    Downloaded 2 tweets
    No more tweets found
    Downloaded 2 tweets, Saved to tweets_ChesterJasper2.json


## Add each tweet as a row to a dataframe named `df_tweets`


```python
tweets={}
for user in users_list:
    fName = 'tweets_'+user+'.json'
    with open(fName, 'r') as f:
        for i,line in enumerate(f):
            tweets[user+'_'+str(i)]=json.loads(line)

df_tweets = pd.DataFrame(tweets).T
```


```python
df_tweets.to_csv('0726_tweets.csv')
```

<hr>


```python
pd.re
```


```python
list(df_tweets.columns)
```




    ['contributors',
     'coordinates',
     'created_at',
     'entities',
     'extended_entities',
     'favorite_count',
     'favorited',
     'geo',
     'id',
     'id_str',
     'in_reply_to_screen_name',
     'in_reply_to_status_id',
     'in_reply_to_status_id_str',
     'in_reply_to_user_id',
     'in_reply_to_user_id_str',
     'is_quote_status',
     'lang',
     'metadata',
     'place',
     'possibly_sensitive',
     'quoted_status',
     'quoted_status_id',
     'quoted_status_id_str',
     'retweet_count',
     'retweeted',
     'retweeted_status',
     'source',
     'text',
     'truncated',
     'user',
     'withheld_in_countries']




```python
['contributors_enabled', 'created_at', 'default_profile', 'default_profile_image', 'description', 'entities', 'favourites_count', 'follow_request_sent', 'followers_count', 'following', 'friends_count', 'geo_enabled', 'has_extended_profile', 'id', 'id_str', 'is_translation_enabled', 'is_translator', 'lang', 'listed_count', 'location', 'name', 'notifications', 'profile_background_color', 'profile_background_image_url', 'profile_background_image_url_https', 'profile_background_tile', 'profile_image_url', 'profile_image_url_https', 'profile_link_color', 'profile_sidebar_border_color', 'profile_sidebar_fill_color', 'profile_text_color', 'profile_use_background_image', 'protected', 'screen_name', 'statuses_count', 'time_zone', 'translator_type', 'url', 'utc_offset', 'verified']
```




    ['contributors_enabled',
     'created_at',
     'default_profile',
     'default_profile_image',
     'description',
     'entities',
     'favourites_count',
     'follow_request_sent',
     'followers_count',
     'following',
     'friends_count',
     'geo_enabled',
     'has_extended_profile',
     'id',
     'id_str',
     'is_translation_enabled',
     'is_translator',
     'lang',
     'listed_count',
     'location',
     'name',
     'notifications',
     'profile_background_color',
     'profile_background_image_url',
     'profile_background_image_url_https',
     'profile_background_tile',
     'profile_image_url',
     'profile_image_url_https',
     'profile_link_color',
     'profile_sidebar_border_color',
     'profile_sidebar_fill_color',
     'profile_text_color',
     'profile_use_background_image',
     'protected',
     'screen_name',
     'statuses_count',
     'time_zone',
     'translator_type',
     'url',
     'utc_offset',
     'verified']




```python
def toDataFrame(tweets):

    DataSet = pd.DataFrame()

    DataSet['screen_name'] = [tweet.screen_name for tweet in tweets]
    DataSet['tweetText'] = [tweet.text for tweet in tweets]
    DataSet['tweetRetweetCt'] = [tweet.retweet_count for tweet in tweets]
    DataSet['tweetFavoriteCt'] = [tweet.favorite_count for tweet in tweets]
    DataSet['tweetSource'] = [tweet.source for tweet in tweets]
    DataSet['tweetCreated'] = [tweet.created_at for tweet in tweets]
    DataSet['userID'] = [tweet.user.id for tweet in tweets]
    DataSet['userScreen'] = [tweet.user.screen_name for tweet in tweets]
    DataSet['userName'] = [tweet.user.name for tweet in tweets]
    DataSet['userCreateDt'] = [tweet.user.created_at for tweet in tweets]
    DataSet['userDesc'] = [tweet.user.description for tweet in tweets]
    DataSet['userFollowerCt'] = [tweet.user.followers_count for tweet in tweets]
    DataSet['userFriendsCt'] = [tweet.user.friends_count for tweet in tweets]
    DataSet['userLocation'] = [tweet.user.location for tweet in tweets]
    DataSet['userTimezone'] = [tweet.user.time_zone for tweet in tweets]

    return DataSet
```


```python
dict1
```




    {'contributors_enabled': False,
     'created_at': 'Wed Aug 30 18:28:30 +0000 2017',
     'default_profile': True,
     'default_profile_image': False,
     'description': 'Follower of YESHUA  JESUS the CHRIST',
     'entities': {'description': {'urls': []}},
     'favourites_count': 1900,
     'follow_request_sent': None,
     'followers_count': 50,
     'following': None,
     'friends_count': 404,
     'geo_enabled': False,
     'has_extended_profile': False,
     'id': 902961240691101696,
     'id_str': '902961240691101696',
     'is_translation_enabled': False,
     'is_translator': False,
     'lang': 'en',
     'listed_count': 0,
     'location': '',
     'name': 'prepare mathew 25',
     'notifications': None,
     'profile_background_color': 'F5F8FA',
     'profile_background_image_url': None,
     'profile_background_image_url_https': None,
     'profile_background_tile': False,
     'profile_image_url': 'http://pbs.twimg.com/profile_images/903036839359107072/Emu94UdD_normal.jpg',
     'profile_image_url_https': 'https://pbs.twimg.com/profile_images/903036839359107072/Emu94UdD_normal.jpg',
     'profile_link_color': '1DA1F2',
     'profile_sidebar_border_color': 'C0DEED',
     'profile_sidebar_fill_color': 'DDEEF6',
     'profile_text_color': '333333',
     'profile_use_background_image': True,
     'protected': False,
     'screen_name': 'fbibug',
     'statuses_count': 5172,
     'time_zone': None,
     'translator_type': 'none',
     'url': None,
     'utc_offset': None,
     'verified': False}




```python
df_tweets=pd.read_csv("follower_tweets_data_7-24.csv")
```

    /anaconda/lib/python3.6/site-packages/IPython/core/interactiveshell.py:2717: DtypeWarning: Columns (1,2,7,8,16,25,29) have mixed types. Specify dtype option on import or set low_memory=False.
      interactivity=interactivity, compiler=compiler, result=result)



```python
df_text=df_tweets[['text']]

```


```python
df_text['word_count']=df_text['text'].apply(lambda x: len(str(x).split(" ")))
```

    /anaconda/lib/python3.6/site-packages/ipykernel_launcher.py:1: SettingWithCopyWarning:
    A value is trying to be set on a copy of a slice from a DataFrame.
    Try using .loc[row_indexer,col_indexer] = value instead

    See the caveats in the documentation: http://pandas.pydata.org/pandas-docs/stable/indexing.html#indexing-view-versus-copy
      """Entry point for launching an IPython kernel.



```python
df_text['char_count']=df_text['text'].str.len()
```

    /anaconda/lib/python3.6/site-packages/ipykernel_launcher.py:1: SettingWithCopyWarning:
    A value is trying to be set on a copy of a slice from a DataFrame.
    Try using .loc[row_indexer,col_indexer] = value instead

    See the caveats in the documentation: http://pandas.pydata.org/pandas-docs/stable/indexing.html#indexing-view-versus-copy
      """Entry point for launching an IPython kernel.



```python
df_tweets.columns
```




    Index(['Unnamed: 0', 'contributors', 'coordinates', 'created_at', 'entities',
           'extended_entities', 'favorite_count', 'favorited', 'geo', 'id',
           'id_str', 'in_reply_to_screen_name', 'in_reply_to_status_id',
           'in_reply_to_status_id_str', 'in_reply_to_user_id',
           'in_reply_to_user_id_str', 'is_quote_status', 'lang', 'metadata',
           'place', 'possibly_sensitive', 'quoted_status', 'quoted_status_id',
           'quoted_status_id_str', 'retweet_count', 'retweeted',
           'retweeted_status', 'source', 'text', 'truncated', 'user',
           'withheld_in_countries'],
          dtype='object')




```python
df_text['screen_name']=df_tweets['Unnamed: 0']
```

    /anaconda/lib/python3.6/site-packages/ipykernel_launcher.py:1: SettingWithCopyWarning:
    A value is trying to be set on a copy of a slice from a DataFrame.
    Try using .loc[row_indexer,col_indexer] = value instead

    See the caveats in the documentation: http://pandas.pydata.org/pandas-docs/stable/indexing.html#indexing-view-versus-copy
      """Entry point for launching an IPython kernel.



```python
df_text['username']=df_text['screen_name'].apply(lambda x:x[:x.rfind("_")])
```

    /anaconda/lib/python3.6/site-packages/ipykernel_launcher.py:1: SettingWithCopyWarning:
    A value is trying to be set on a copy of a slice from a DataFrame.
    Try using .loc[row_indexer,col_indexer] = value instead

    See the caveats in the documentation: http://pandas.pydata.org/pandas-docs/stable/indexing.html#indexing-view-versus-copy
      """Entry point for launching an IPython kernel.



```python
df_text.columns=['text','word_count','char_count','order_recent','screen_name']
```


```python
df_text['order']=df_text['order_recent'].apply(lambda x:x[x.rfind("_")+1:])
```

    /anaconda/lib/python3.6/site-packages/ipykernel_launcher.py:1: SettingWithCopyWarning:
    A value is trying to be set on a copy of a slice from a DataFrame.
    Try using .loc[row_indexer,col_indexer] = value instead

    See the caveats in the documentation: http://pandas.pydata.org/pandas-docs/stable/indexing.html#indexing-view-versus-copy
      """Entry point for launching an IPython kernel.



```python
df_text=df_text.drop('order_recent',axis=1)
```


```python
def avg_word(sentence):
    try:
        words = sentence.split()
    except:
        return np.nan
    return (sum(len(word) for word in words)/len(words))

```


```python
df_text['avg_word']=df_text['text'].apply(lambda x: avg_word(x))
```


```python
df_text[df_text.avg_word.isna()==True]
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>text</th>
      <th>word_count</th>
      <th>char_count</th>
      <th>screen_name</th>
      <th>order</th>
      <th>avg_word</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>103724</th>
      <td>NaN</td>
      <td>1</td>
      <td>NaN</td>
      <td>intensidad y/o lluvias en localidades ubicadas</td>
      <td>intensidad y/o lluvias en localidades ubicadas…</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_text=df_text.drop(103724)
```


```python
len(df_text)
```




    107157




```python
df_text['words']=df_text.text.apply(lambda x: x.split())
```




    0         [VAMC'S, R, TRAINED, MURDERERS, THE, SMART, WH...
    1         [https://t.co/KwImJ6hKrh, no, one, owns, land,...
    2         [RT, @ROHLL5:, 🔁, RT, if, you, agree, #44, .@P...
    3         [Wait, next, is, Burnie, you, will, love, comi...
    4         [It's, simple, landlords, get, rich, quick, sc...
    5         [He, will, be, a, new, member, of, the, royal,...
    6         [Il, come, visit, u, it, the, penitentiary, CO...
    7         [Or, if, comi, COMEY, and, Mac, dogg, McCabe, ...
    8         [If, his, lips, R, MOVING, HE, IS, LIEING, and...
    9         [Judges, and, prosecuters, all, must, be, repl...
    10                      [So, evil, https://t.co/kvEXIjfleL]
    11        [It's, a, team, and, this, one, worked, for, d...
    12        [White, House, chief, of, staff, John, Kellyre...
    13        [@myhtopoeic, Watch, for, a, microwave, weapon...
    14        [How, much, money, and, deals, did, PRESIDENT,...
    15        [Due, to, known, bad, drugs, used, and, to, hi...
    16        [@VeteransHealth, I, am, sure, thankful, for, ...
    17        [Send, this, law, legislating, judge, to, the,...
    18        [If, u, were, President, and, NFL, owner, refu...
    19        [Not, long, now, no, one, wants, to, be, slave...
    20        [https://t.co/etom1Oxb2Q, oh, LOOKIE, NY, via,...
    21        [He, has, not, killed, to, many, not, to, much...
    22        [What, a, stretch., No, king, David, went, int...
    23        [So, is, scum, sucker, spying, ass, SENATOR, J...
    24        [Your, interest, in, paying, interest, https:/...
    25        [Yo, sanctions, have., Ever, been, strong, eno...
    26                     [Not., was, https://t.co/5AtRADKwtT]
    27              [He, gave, it, up, https://t.co/gVw3aZnNMW]
    28        [Well, like, your, peon, asses, should, be, re...
    29        [Ted, u, R, and, general, Michael, hidden, say...
                                    ...                        
    107128    [@soccer_gcc, ترا, هذي, طريقه, سهله, جدا, يكتب...
    107129                       [@ufmradio, هذا, مو, كنه, مات]
    107130                     [@nayef965, اتفق, جملة, وتفصيلا]
    107131    [RT, @al7umaide:, السلام, عليكم, لو, تكرمتو, م...
    107132    [@T8VxPG6PQ4kiMkm, @PhoneAmh1, @RAH_M12, @wana...
    107133    [@PhoneAmh1, @RAH_M12, @wanaassah, @sasmari473...
    107134    [#مستمرين_ومقاطعين_المراعي, اكثر, المشاركين, ت...
    107135    [@PhoneAmh1, @RAH_M12, @wanaassah, @sasmari473...
    107136    [@MSDAR_NEWS, تكلفة, المعيشه, ممكن, لكن, الي, ...
    107137    [@UberFacts, No, one, feae, Fridays, we, only,...
    107138    [@sadeghZibakalam, معلومه, که, مردم, به, خواست...
    107139    [@sadeghZibakalam, معلومه, که, بعد, از, براندا...
    107140            [#NewProfilePic, https://t.co/dCCfJrLuKB]
    107141                            [@ImanMunal, Naked, True]
    107142    [⠀, ⠀, ⠀, ⠀, ⠀, ⠀, ⠀, ⠀, ⠀, ⠀⠀⠀, SMILE, ⠀⠀, Ev...
    107143    [RT, @arewashams:, Every, woman, deserves, a, ...
    107144    [RT, @Tweets2Motivate:, Today, is, a, great, d...
    107145                [@Binat_Aji, https://t.co/HckHH1DFVR]
    107146    [RT, @HQNigerianArmy:, The, (COAS), Lt, Gen, T...
    107147    [@HEDankwambo, @BusinessDayNg, Congratulations...
    107148                     [@ayeesh_chuchu, kut, lanlakam.]
    107149    [RT, @umanafs:, Yakai, Dan, Uwa!!, Kasani, Sal...
    107150    [RT, @CleverQuotez:, Never, argue, with, an, i...
    107151    [RT, @TrackHateSpeech:, FAKE, NEWS!, 6, people...
    107152    [@Madame_Flowy, Why, do, You, ask, all, this, ...
    107153                [@itswarenbuffett, Kind, regard, Sir]
    107154                          [@umanafs, Amin, ya, Allah]
    107155               [@th_saleem, Hallaka, kobo, at, gombe]
    107156    [Just, posted, a, photo, https://t.co/NZryHlilWG]
    107157                                 [@rssurjewala, Nice]
    Name: words, Length: 107157, dtype: object




```python
stop = stopwords.words('english')

df_text['stopwords']=df_text.text.apply(lambda x: len([x for x in x.split() if x in stop]))
```


```python
df_text.stopwords.max()
```




    18




```python
df=df_text
```


```python
df['hashtags']=df['text'].apply(lambda x: [x for x in x.split() if x.startswith('#')])
```


```python
df['num_hashtags']=df['text'].apply(lambda x: len([x for x in x.split() if x.startswith('#')]))
```


```python
df['numerics']=df.text.apply(lambda x: len([x for x in x.split() if x.isdigit()]))

```


```python
df['upper']=df['text'].apply(lambda x: len([y for y in x.split() if y.isupper()]))
```

# Part 2. Pre-Processing

## 2.1 Lower Case


```python
df.text=df.text.apply(lambda x: ' '.join(x.lower() for x in x.split()))
```

## 2.2 Remove Punctuation


```python
df.text=df.text.str.replace('[^\w\s]','')
```

## 2.3 Remove Stopwords


```python
df.text=df.text.apply(lambda x:' '.join(y for y in x.split() if y not in stop))
```

## 2.4 Remove Common Words


```python
freq = pd.Series(' '.join(df.text).split()).value_counts()[:100]
```


```python
freq
```




    rt                 62622
    realdonaldtrump     9581
    trump               9025
    president           4745
    amp                 4702
    us                  3716
    people              3706
    one                 3398
    like                3346
    dont                2905
    putin               2865
    russia              2730
    would               2701
    obama               2682
    know                2641
    get                 2573
    de                  2313
    potus               2277
    im                  2205
    fbi                 2123
    time                2090
    foxnews             2051
    russian             2037
    democrats           2021
    see                 1894
    new                 1842
    today               1839
    american            1836
    never               1836
    fisa                1821
                       ...  
    really              1185
    first               1184
    world               1167
    state               1160
    mueller             1157
    didnt               1157
    fake                1154
    ever                1147
    cnn                 1131
    could               1131
    thats               1129
    que                 1118
    2                   1110
    every               1106
    hes                 1097
    much                1095
    breaking            1059
    clinton             1044
    must                1042
    white               1029
    youre               1024
    got                 1017
    god                 1011
    way                 1005
    w                    992
    trumps               989
    support              959
    believe              954
    dbongino             942
    yes                  927
    Length: 100, dtype: int64



## 2.5 Spelling Correction


```python
from textblob import TextBlob
```


```python
# This takes a long time to do
#df['text_spell_corrected']=df.text.apply(lambda x: str(TextBlob(x).correct()))
```

## 2.6 Stemming - Removal of Suffices like 'ing', 'ly', 's', etc.


```python
from nltk.stem import PorterStemmer
```


```python
# st = PorterStemmer()
# df.text.apply(lambda x: " ".join([st.stem(word) for word in x.split()]))
```

## 2.7 Lemmatization - Converting to Root Word


```python
from textblob import Word
```


```python
df['lemmatized'] = df.text.apply(lambda x: ' '.join([Word(word).lemmatize() for word in x.split()]))
```

# Part 3. Advanced Text Processing

## 3.1 N-grams

N-grams are the combination of multiple words used together. Ngrams with N=1 are called unigrams. Similarly, bigrams (N=2), trigrams (N=3) and so on can also be used.

Unigrams do not usually contain as much information as compared to bigrams and trigrams. The basic principle behind n-grams is that they capture the language structure, like what letter or word is likely to follow the given one. The longer the n-gram (the higher the n), the more context you have to work with. Optimum length really depends on the application – if your n-grams are too short, you may fail to capture important differences. On the other hand, if they are too long, you may fail to capture the “general knowledge” and only stick to particular cases.


TextBlob(df.text[0]).ngrams(2)
## 3.2 Term Frequency

Term frequency is simply the ratio of the count of a word present in a sentence, to the length of the sentence.

Therefore, we can generalize term frequency as:

TF = (Number of times term T appears in the particular row) / (number of terms in that row)


```python
tf1 = (df.text[0:20]).apply(lambda x: pd.value_counts(x.split(" "))).sum(axis=0).reset_index()
```


```python
tf1.columns=['words','tf']
```


```python
tf1
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>words</th>
      <th>tf</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>went</td>
      <td>2.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>college</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>sales</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>dope</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>trained</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>5</th>
      <td>httpstco4aluzu5crs</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>6</th>
      <td>smart</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>7</th>
      <td>2</td>
      <td>3.0</td>
    </tr>
    <tr>
      <th>8</th>
      <td>classes</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>9</th>
      <td>wa</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>10</th>
      <td>r</td>
      <td>2.0</td>
    </tr>
    <tr>
      <th>11</th>
      <td>murderers</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>12</th>
      <td>murdering</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>13</th>
      <td>poor</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>14</th>
      <td>vamcs</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>15</th>
      <td>cor</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>16</th>
      <td>httpstcozs4reg46iq</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>17</th>
      <td>america</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>18</th>
      <td>must</td>
      <td>2.0</td>
    </tr>
    <tr>
      <th>19</th>
      <td>land</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>20</th>
      <td>u</td>
      <td>6.0</td>
    </tr>
    <tr>
      <th>21</th>
      <td>corporation</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>22</th>
      <td>united</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>23</th>
      <td>httpstcokwimj6hkrh</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>24</th>
      <td>pay</td>
      <td>2.0</td>
    </tr>
    <tr>
      <th>25</th>
      <td>states</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>26</th>
      <td>one</td>
      <td>3.0</td>
    </tr>
    <tr>
      <th>27</th>
      <td>owns</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>28</th>
      <td>rt</td>
      <td>2.0</td>
    </tr>
    <tr>
      <th>29</th>
      <td>maga</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>179</th>
      <td>window</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>180</th>
      <td>judge</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>181</th>
      <td>dumpster</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>182</th>
      <td>law</td>
      <td>2.0</td>
    </tr>
    <tr>
      <th>183</th>
      <td>poop</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>184</th>
      <td>send</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>185</th>
      <td>httpstcokf9ujp3kbw</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>186</th>
      <td>shoot</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>187</th>
      <td>legislating</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>188</th>
      <td>thru</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>189</th>
      <td>national</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>190</th>
      <td>total</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>191</th>
      <td>fine</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>192</th>
      <td>would</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>193</th>
      <td>httpstcolxuemb10tx</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>194</th>
      <td>owner</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>195</th>
      <td>anthem</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>196</th>
      <td>refused</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>197</th>
      <td>disrespect</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>198</th>
      <td>players</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>199</th>
      <td>nfl</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>200</th>
      <td>back</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>201</th>
      <td>break</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>202</th>
      <td>spiri</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>203</th>
      <td>master</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>204</th>
      <td>slave</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>205</th>
      <td>wants</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>206</th>
      <td>httpstcod1o30tpr05</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>207</th>
      <td>georgetta</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>208</th>
      <td>chicken</td>
      <td>1.0</td>
    </tr>
  </tbody>
</table>
<p>209 rows × 2 columns</p>
</div>



## 3.3 Inverse Document Frequency

The intuition behind inverse document frequency (IDF) is that a word is not of much use to us if it’s appearing in all the documents.

Therefore, the IDF of each word is the log of the ratio of the total number of rows to the number of rows in which that word is present.

IDF = log(N/n), where, N is the total number of rows and n is the number of rows in which the word was present.

So, let’s calculate IDF for the same tweets for which we calculated the term frequency.


```python
for i, word in enumerate(tf1['words']):
    tf1.loc[i,'idf']=np.log(df.shape[0]/(len(df[df.text.str.contains(word)])))
```


```python
df.upper.max()
```




    31




```python
df.numerics[2]
```




    1




```python
df.words[2]
```




    ['RT',
     '@ROHLL5:',
     '🔁',
     'RT',
     'if',
     'you',
     'agree',
     '#44',
     '.@POTUS',
     'Did',
     'more',
     'harm',
     'than',
     'good',
     'in',
     '8',
     'years',
     'serving',
     'our',
     'country.',
     '#ObamaLegacy',
     '#PatriotsUnite',
     '#MAGA…']




```python
df_users=pd.read_csv("follower_data_7-24.csv", index_col=0)
```


```python
df_users=pd.read_csv("follower_data_7-24.csv", index_col=0)
list(df_users.follower)
```




    ['ChrisEnerIdeas',
     'fbibug',
     'MishaJo26609942',
     'Clint12Michelle',
     'bustin1791',
     'DPgrandaughter',
     'imnos482',
     'marketmartha',
     'BlueSk13s',
     'Ximena74560472',
     'US_Marj',
     'GEPLLC18',
     'cjdabney1',
     'kwosborne',
     'TapHopQuocDanVN',
     'kriscerz',
     'ronaldj100',
     'hebert53754386',
     'Jamesonsmimi',
     'johnfnfunny',
     '92stag',
     'utz43',
     'Am73A',
     'gibsonsingley',
     'lanenadesign',
     'Michell24701779',
     'kevinjbuttner',
     'stiffjoints',
     'WandaCa71748637',
     'yishkemishke',
     'ciaran_carroll',
     'CamillaJoy53',
     'dcarinc',
     'ricantw',
     'ClaflinVickie',
     'pistach01',
     'tanya58430323',
     'Hardogeat',
     'billstorms',
     'ribovic_terri',
     'Coylerspoiler',
     'FlorianBulica',
     'AngelaEanon',
     'Aaron54860652',
     'morigdl',
     'law6',
     'kimiw506',
     'TetonScott',
     'OwensAshby',
     'DanCox05341052',
     'luwiccalu',
     'TomRoyActor',
     'ReadsHemingway',
     'tjadiska',
     'InGodwetrust29Q',
     'oc_chrissie',
     'GiboneyDavid',
     'dreamatrobs',
     'samanthsmith1',
     'Austinginger',
     'CA_LaJolla',
     'DebSpicer3',
     'cypaz94',
     'eileenr47236946',
     'wny63',
     'familyfirst2018',
     'RobertS4042',
     'DavidCinalli1',
     'peachesforall',
     'ZanderPigDog',
     'aldridtl',
     'patreehugger57',
     'oldmanluvsmineo',
     'Bunky7777',
     'jones33462996',
     'HopeAlways888',
     'blkmtnauto',
     'patriot_angel',
     'ReetzDavidAllen',
     'Bateman66Mrs',
     'Leog1442',
     '4LynBliss',
     'msann43',
     'jjeffhamman',
     'donnamaynes',
     'rickadam0',
     'hayatti10',
     'realTammyC',
     'JNotrump',
     'norlaskan',
     'fisherhaven92',
     'hairyvnvdh',
     'USAMarineMomma',
     'TRHargrave',
     'redpillmasshole',
     'huskerdiva',
     'jtstevensaz',
     'Atomic_Blonde__',
     'PatriotChic33',
     'MAXIMFREEMAN',
     'severianlives',
     'Kianna__99',
     'LaanHooi',
     'countryboyexec',
     'BrendaKaye_USA',
     'buckoakwell',
     'Suresh0000008',
     'tracec66',
     'AndreaWeslien',
     'ArturoAlonzoRi1',
     'ACrazyWoman',
     'lockednloaded33',
     'BigDog2k2',
     'Maguire12J',
     'marie_tellefsen',
     'ExecGQ',
     'JimShaf',
     'Mosleygirl7',
     'mikemartin123',
     'saweetone2010',
     'steph_cherry',
     'MrsAutonomous',
     'marybleuz205',
     'MkhatriAmine',
     'RedBuster77',
     'pratt1111',
     'jewelrybycin',
     'HEYHEATH1',
     'bob_cat711',
     'bobtomm1',
     'realDCLady',
     'leatherneck7051',
     'magates68',
     'JUDYKNY',
     'davidbcrumbs',
     'trdrdoc3',
     'Myhopeissure',
     'amrebellion',
     'CanneryCathouse',
     'jerryt39',
     'kevin_kallal',
     'PamReese19',
     'keep_US_great',
     'RandallKraft',
     'dyer_sampson',
     'kathieblackwe14',
     'MAGAcindyKAG',
     'stonebluffranch',
     'khemtrails',
     'SavageTooney',
     'PattiaGarder',
     'LauraChin',
     'shegone7',
     'anolamawmaw',
     'DennisMaroules',
     'iamcmax',
     'kim1124',
     'Brian77Harris',
     'JenniferLMetro',
     'DaveTho86176055',
     'dultra1',
     'khann0608',
     'AwakeningCoach1',
     'C_Marie_Stella',
     'joann_obrien_',
     'Dmwalker71',
     'MamasueFuentes',
     'joycombass',
     '73Stroppe',
     'palmibc',
     'McCarleyTrey',
     'friedrich5',
     'RickDoyle',
     'ChicagoVVI',
     'Nickko888',
     'ConnieH15',
     'lilypie54',
     'rayann2320',
     'EdgeLSeawolf',
     'Sonee333Sonny',
     'Iamcmaxwell',
     'Blondiebraids',
     'lfishyfroggie',
     'KCoastalSand',
     'JamesPowell41',
     'EG01206893',
     'Michael29722106',
     'Telford_Russian',
     'Swaa88',
     'talyahxoxo1',
     'cherylsalvati',
     'lvnlf267',
     'bash569',
     'RickRickbard21',
     'hopeful188',
     'jcmarvar',
     'BurghDynamite',
     'maga4CAC',
     'evelyn_wydra',
     'Mdnotadoctor64',
     'laurieg47',
     'LanbowW',
     'klasique_honour',
     'skylarhamilton7',
     'BreGabryela',
     'huizar_u',
     'Flyy_Boyy_Q600',
     'Taldobenjamin',
     'LuanLuan102',
     'cheng_sophal',
     '4kJlRVcTYxISGWb',
     'Delfino96969529',
     'Varibabu4',
     'vinoth46847221',
     'x_roma2040',
     'DeWayne02460455',
     'clark35534423',
     'FSMEhL0sRycSyQV',
     'carl_henr10',
     'jrtuitele26',
     'ebaduu',
     'SundaryLuiary',
     'Christi28783044',
     'ForKisses2',
     'cocotsui1',
     'BelloWilliams5',
     'yehang8',
     'Abigail68165497',
     'TVanguard1',
     'donnie_galyean',
     'Amany51174915',
     'Atahual41545318',
     'TahirTabassum4',
     'helena_is_out',
     'bl81799429',
     'FotunatoJohnny',
     'GucciKookieBear',
     'Darrenjr6',
     'TngLnhThin2',
     'LaurensBancroft',
     'MassielElimar',
     'Sharanj28029508',
     'CarminiaYrala',
     'Mutasim96332100',
     'JohnBrathwait11',
     'OlyaYyz',
     'vigneshvgs444',
     'skafader',
     'Cande27219434',
     'PoonZJ_DZ',
     'amgjefe',
     'Alsayiad2',
     'NanaThelma',
     'ann_juniper',
     'joysoysays',
     'minnhi11',
     'lan937571',
     'meepsheepkthnx',
     'Rajiv82466178',
     'tbuckner36',
     'ZheLiu18',
     'kat39020289',
     'Perry77185675',
     'feb630fe266248d',
     'mashaan_305',
     'moatasim_khalid',
     '0uMF4QET0vd9fBM',
     'PawanKu30894525',
     'ManuelM55314030',
     'faygu3652',
     'mrakmal007',
     'theblast805',
     'AimeeZong2',
     '3DuirdtVHS9yyJm',
     'rubenvalero23',
     'Pedro71735381',
     'duongvanvan1',
     'ZJDesigns',
     'adamemamooli',
     'ApproacRational',
     'Mohamed68123066',
     'EduX_Torres',
     'LayemoMicheal',
     '_Curtis1011',
     'Ann56420927',
     'jacavendano',
     'minion02151',
     'nlv_dave',
     'BoeserForMVP',
     'bjpotts123',
     'LeonaTapp',
     'vietha53854414',
     'abomahm39154117',
     'EFlyer87',
     'AbiodunAbigail5',
     'sportsjunkie841',
     'OctavioMena6',
     'Harry84588018',
     'PennyEshleman1',
     'daniloj2205',
     'daniloj2205',
     'JhonLpe77896979',
     'mamp933',
     'GlombMax',
     'guara41',
     'ZezoGabr4',
     'Ahmed33094376',
     'HalonaHunt',
     'Mahnuelskin',
     'ahlam_errajy',
     'AbdoMoh66950581',
     'anufaie',
     'pleazestopme',
     'OksanaKulakov',
     'xiao74072112',
     'hunterschramm',
     'PandianSiddarth',
     'haraldbachmann3',
     'WilhelmScream6',
     'zzz840025',
     'juggalocracy',
     'jpgrrule111',
     'KalyanR11722052',
     'Rishi20051',
     'ashtonsancharra',
     'kurse0820',
     'cnerad',
     'sinha_pramila',
     'Carl64869520',
     'mKQcboDXP0WteqL',
     'Jase26968501',
     'Vamsi78944118',
     'PPalecek',
     'Dialmond',
     'Luis79124428',
     'hpark83',
     'Waled62316713',
     'mike_forsyth18',
     'BodalaSarat',
     'kaotik57272690K',
     'JuanelCurioso2',
     'mcmurray461',
     'Jesusmartinag14',
     'LauraBethVIP',
     'v6J9eXqQSMfRjGy',
     'bynbss',
     'AhmedAs160',
     'MarleneEGarcia7',
     'dwi2899',
     'ColarossiRobert',
     'taryn34520375',
     'katelynharrod18',
     'malikqa28117529',
     'PankajBunkar7',
     'izoEWMpRTSnZNZI',
     'EDSTER1',
     'Stability35',
     'PronpimonPutbu1',
     'Penelop96428840',
     'lin09541371',
     'acf1234',
     'yulviani_dwi',
     'maniacruzz',
     'SGE44755383',
     'ScottStaples16',
     'Abdulla66703109',
     'AhirShrimant',
     'huyensuga93',
     'Goosey_Woosey',
     'jcarrhc',
     'ChemChantrea',
     'tayahamaya',
     'nick160maria',
     'Siobhan84876053',
     'marcoau38999902',
     'Sandra23495441',
     'ShivAga76081050',
     'koppoi1992',
     'Khalida64593357',
     'janaraphi',
     'BryanP18',
     'bonny_konjore',
     'based_zamasu',
     'IshfaqHiraj10',
     'petracar10',
     'gussyatim',
     'jacobom123',
     'JosieXinyue',
     'MarkCru67361189',
     'TSVBmnJINqmSInd',
     'DOIC6qvF4SVm9Ml',
     'SaintLonglive',
     'UUpFdRnHM9wzlD5',
     'MayerlisPerez17',
     'salopga',
     'kevinfe66751509',
     'deepakd57743679',
     'ccl0826',
     'thegulley',
     'Ismael51041661',
     'Emilly_2618',
     'DeepakC41236137',
     'PokuMensah3',
     'cocotsui1',
     'skylarhamilton7',
     'Taldobenjamin',
     'ariannax1008',
     'ayva34436066',
     'Varibabu4',
     'vinoth46847221',
     'Beyondlynn1',
     'DanielC15972930',
     'Giselle96310012',
     'cheng_sophal',
     'POro1904',
     'BelloWilliams5',
     'LingLingHello',
     'SaqibJaweedSyed',
     'mendesvih12',
     'SANICHOLS_DU',
     'SantyDieg02',
     'Bahlesolla1',
     'AnimashaunGbol1',
     'realharrymadrid',
     'Atahual41545318',
     'GomesSi53642231',
     'babaralibakhsh',
     'briabou',
     'Joel96482420',
     'thisisspiffy',
     'j_e7f',
     'OrMktn',
     'Miko2114',
     'JohnBrathwait11',
     'ZanageeArtis',
     'Rashidu64794460',
     'ManuelM55314030',
     'mirnarsan',
     'DineDidinedi',
     'Juliana50327407',
     'nicolelsolis',
     'MassielElimar',
     'WayneGr02590947',
     'BrendaM49400017',
     'eleidyg',
     'FukasawaYo',
     'ToddBower09',
     'HouldThe',
     'mooddeprived',
     'Hafza16347571',
     'theatreaddict7',
     'errahul26',
     'albarosamolina2',
     'badboiEezi',
     'chrisknow3',
     'sumnyanS',
     'MaleekRhine',
     'jacavendano',
     'NiggaHype',
     'daothuha8',
     'ShatteredPrism_',
     'Tavoec',
     'malikqa28117529',
     'JavierGG_08',
     'tayahamaya',
     'Diana90280075',
     'Mahnuelskin',
     'Kc0779213086',
     'RajKuma98524538',
     'NAKorsak',
     'nancydaniela29',
     'Tesha93146368',
     'ReedsonAdrien11',
     'Mrs_C_A_L_M',
     '_kaya82',
     'adammiyara17',
     'Kingsamd83',
     'ashtonsancharra',
     'JBrandonP88',
     'yamiiiortizzz',
     'Rishi20051',
     'PandianSiddarth',
     'Jennife40158930',
     'hpark83',
     'Pradeepchimour1',
     'ISaraaguilar',
     'Sajidhu35240898',
     'Edward13079643',
     'arturfreitasp11',
     'MoetChandonR7',
     'OlmedoMonar',
     'feritas_de',
     'irissaqui',
     'jcampan5',
     'FarzandMalik2',
     'DeivisonBorges4',
     'Mnak27633155',
     'Asad73851197',
     'Luis79124428',
     'Jeanbos37762282',
     'dwi2899',
     'SaintLonglive',
     'JB__43',
     'Brisa79631341',
     'DhaZOEcitdIUahB',
     'PankajBunkar7',
     'ayaezzat220',
     'Abdulla29044154',
     'EDalton_2',
     'webermax221',
     'Joseph210203',
     'NometryTrigger',
     'Luz00916247',
     'UnembellishedU',
     'hwelloyoonie',
     'sweethoneyjai',
     'mapoke4',
     'arizonatweeting',
     'cristina_sahra',
     'Vanessa08326597',
     'bpdoulas',
     'AmberAl22328719',
     'marcoau38999902',
     'maniacruzz',
     'juliagosullivan',
     'ChemChantrea',
     'huyensuga93',
     'Joy4216',
     'Kempslice',
     'phuongt78695951',
     'LSqyeeze',
     'CosplayRosali',
     'Critico82',
     'AmberBarak',
     'LaminJa53384782',
     'Vantroy14',
     'Sami82901747',
     'Sebasti65141568',
     'petracar10',
     '15755357557John',
     'justinwoodart',
     'Hitfgup',
     'Josephi30738083',
     'GurlalS30912504',
     'ileadkristin',
     'Daniell69713339',
     'VonnJones1',
     'JazmnMedina16',
     'Sandrusz1',
     'ChartleyG',
     'MicheleCoker3',
     'Trinniii_',
     'Guilher00901118',
     'xu54821668',
     'gechboi',
     'imperial_gino',
     'Jessica45153720',
     'emmewright5202',
     'stroupy50',
     'JosephGodoy15',
     'AkhilaJahnavi',
     'Kionara3',
     'akarat18',
     'windyalisaa',
     'salopga',
     'ihsirarti',
     'DebbieR81006015',
     'npartei',
     'DarielsJoyas',
     'DarkEricka7w7',
     '_s_e_t_h_e_',
     'kei13113123',
     'Annanic51184436',
     'acampors',
     'HuntelJake',
     'Hampsterpunk1',
     'Mariko51995138',
     'BeckyBrams',
     'BC2254684509',
     'zhao951814506',
     'LeeLeeBibbs',
     'm_zobaran',
     'dafnefrancoh',
     'Lucifer46210348',
     'Samy43124227',
     'Kaiden21323244',
     'NarcisoLic',
     'arivera5467',
     'Rahi32700116',
     'AssociationAfg',
     'emmalulers',
     'sinha_pramila',
     'Adriana50724996',
     'Larrymarmalaide',
     'elvinarmdn',
     'Sushant53302071',
     'JoseRa67699638',
     'Kamales99925306',
     'DanatSCF',
     'KrogmanCres',
     'Melanie6563',
     'MichielBoland',
     '4loveofcocco',
     'PonteVedraMan',
     'jomac23532643',
     'Lucianlifetv',
     'MANUELBROKER',
     'falann_',
     'RaShadDaScorpio',
     'stevendrozd',
     'vabelle2010',
     'Vard',
     'STrammell302',
     'NYCFunnyGirl1',
     'tkiefer4',
     'CindyS2',
     'Dakota2176453',
     'ONECreativeLab',
     'Sheckkd',
     'nedartrizzo1',
     'Lavery',
     'JoeClitherow',
     'LoonsfootRachel',
     'PrashShanbhag',
     'KathyDePalma3',
     'Russ20182516',
     'LabradorRose2',
     'LindaLaam',
     'mamunhussein',
     'Rhonda07600828',
     '15TLF',
     'janeneweber',
     'IngloriousHRC',
     'TrumpTraitor2',
     'steven_pinney',
     'watchdogpolitic',
     'JonathanFrenzel',
     'GaryStamfordCT',
     'mag7373',
     'NancyBevan119',
     'Nherget',
     'DGhost11',
     'GingerHavel',
     'curran_tim',
     'fishbulb18',
     'OcrHunter',
     'yogeshhwks',
     'KobeSwish',
     'DBlack_Mountain',
     'MichaelSapir',
     'Iamtired11',
     'Samantha114007',
     'DanaGuefen',
     'rtdonna112246',
     'KalifKasaba',
     'theresa62605716',
     'LindaRobinson32',
     'ChicoFathead',
     'mellymagscopy17',
     'maryhilliard45',
     'jewellstudent',
     'stephrick13',
     'Abouamira9',
     'HeidiYip22',
     'dumoulin_joel',
     'tammywi77033933',
     'Brooklynsweet2',
     'YoussefBiari3',
     'pawe_osuch',
     'AJACKSARIES',
     'takedowntrump6',
     'Susweca2',
     'GypseBautista',
     'autismally',
     'ArchanaSivaram2',
     'rtschump',
     'JoseSan56323901',
     'K0RANlSBURNING',
     '6875Joe',
     'eganomixxxxxxxx',
     'mk_cherek',
     'McClamDarren',
     'ApriLovesFox',
     'Jbord23',
     'realarmeydean',
     'Porter19143',
     'ImayememO',
     'imnos482',
     'hippycowgirlme',
     'Sally36040646',
     'AutumnS31760148',
     'Kencutter321bo1',
     'KathyBateman11',
     'David06712047',
     'HuibBoxtel',
     'SellMiaFL',
     'Deb_Prothero',
     'baldyriek',
     'LindaWa80147858',
     'deliverance_l',
     'CONald_PLUMP',
     'neeyalornodeal',
     'MiamiNanci',
     'Veronic85916251',
     'RhodesyLu',
     'Tut84086992',
     'MyGardenLady',
     'immers303',
     '64_hepcat',
     'RealAmyV',
     'EddyD03932096',
     'RumHamzah',
     'Jarnocan',
     'yvelaine',
     'staunsk',
     'jecosgrove',
     'kkowie',
     'NotForYou',
     'soberealestate',
     'EMSalasME',
     'roger28661379',
     'crushmycurls',
     'ginger_army',
     'YouknowMeMan1',
     'Dire_Parakeet',
     'Just_JD38',
     'TherealHez',
     'solanaskyes',
     'billybreathes71',
     'Luulua',
     'BaylessMarilyn',
     'Fr_GerryPehler',
     '1TileStoneMan',
     'Tutu15346383',
     'SandiRipley',
     'Fmrgreenberet',
     'RealTalk_Rasaq',
     'Don_Bellunduno',
     'RonRomero16',
     'kevinpeter0327',
     'Woodybiscuit',
     'NBlack815',
     'BarbaraR75',
     'pambamaol',
     'spellmejewel',
     'syrus88864704',
     'dnjdnsmmdm',
     'Dona41483051',
     'feloraffy',
     'TapRmanmLg',
     'appleface321',
     'rickspressroom',
     'snickerdog',
     'Telitabobita',
     'AmySielaff',
     'Carey23699750',
     'bjyj52',
     'messinagirl',
     'svhr1',
     'CindyZHolland',
     'scjaram1',
     'sirenzstiletto',
     'Trlrprksqrl',
     'Global_Academic',
     'J_Rogers_13',
     'kanester1970',
     'Jackiej19541',
     'judeeksha',
     'eneberman',
     'col48646698',
     'jcm1jam1',
     'OliverHoss',
     'TheLockGuy',
     'djkronyx',
     'lrspsthm',
     'billtho27763947',
     'spiker_r',
     'antom3559',
     'Nadesh13',
     'BassMagic17',
     'danielgarr_uk',
     'madge_22',
     'JG7Texas',
     'JeanneZaksheske',
     'JmonaReviews',
     'Magic_Meds',
     'davicharbonneau',
     'Snopeldee',
     'jenieloo',
     'ElizabethPrimr3',
     'TRSDC',
     'cgholguin',
     'thereadkids',
     'Christi58823523',
     'matttttaber',
     'HeatherLynnB',
     'IndivisibleVV',
     'stuffnsew',
     'phillipsted',
     'Kocummin',
     'Epiiietaph',
     'asmacadabra1',
     'Johnson_Mmartin',
     'RealBillDuke',
     'LeannTerresa',
     'Denny0504',
     'BoroOhioChamber',
     'bradley92425823',
     'sharmabipul1',
     'SophiaMingSun',
     'Nicolas88258752',
     'saleslady21',
     'daveminsky',
     'abdbedoo',
     'mhabolore',
     'fiverr_t',
     'bbentzftr',
     'BSeanghort',
     'dimitri_pence',
     'RumanaAkhter2',
     'Awhy69605436',
     'Trump4VT',
     'myarmswideopen',
     'theceoshangout',
     'NLAR_TANK_GANG',
     'finitoo2',
     'SaumGuy',
     'chosenlyric',
     'GathukiNgahu',
     'genocsp',
     'universalonlin6',
     'therealcommof40',
     'DeplorableDrag1',
     'SMGRealEstate',
     'DonFran92996245',
     'pankajbhartia2',
     'JamesMuraguriN1',
     'soram_jack',
     'BayesLevi',
     'JohnBitondo',
     'enambadol',
     'BtcSoft',
     'TheBossActions',
     'emreb_78',
     'JamesOt69011316',
     'MILONk8',
     'AMAZONJUDE1',
     'owuorvince',
     'Lovevilasis',
     'JennyMi60134518',
     'eman5205',
     'AmericaCiscoKid',
     'ceo4jennylee',
     'SoundMoney2017',
     'dlsnycmtl',
     'TJO1776',
     'BDawg120279',
     'CarolBa88945394',
     'tweetybird2009',
     'mikasparkle',
     'RobbieR71059269',
     'Tahathe3',
     'EPolkosnik',
     'TrudyGerstner',
     'daveosborne301',
     'AnjumTanika',
     'decaturlegends',
     'Angelmimiromy',
     'ManagerLegend1',
     'abdelpmf',
     'NishatS37562728',
     'Jahedul70235424',
     'Hesamgheybi3',
     'SatonePankaj',
     'RickStevens8',
     'TechnicalVirtu1',
     'lesliejjxx',
     'ionaskon',
     'mcprime25',
     'ChenaultJerry',
     'jakariya006',
     'kdwr1991',
     'Kailas02428564',
     'TaylorH70949680',
     'MythicHero888',
     'xieqifei',
     'JoeyA1788',
     'k_bito',
     'Alexsmithgrath1',
     'ShaikSa09579458',
     'MennaFini',
     'MomsforMorals',
     'hadred181',
     'ikennaogbuanu',
     'Matins37997870',
     'FredWhi92587143',
     'SintradoT',
     'william54779520',
     'MShuxin',
     'BrickWall368',
     'downeypalmer',
     'AshleyHendrich',
     'amaloneMO',
     'DJ60235685',
     'LYKINN',
     'CocknfireDC',
     'Bajrang84451368',
     'KimLeaMller2',
     'CoffeeM63496142',
     'Rickcosta18',
     'BenNowell2',
     'ystwgz',
     'kaylamolander',
     'Charlot31492583',
     'MdRejwanHossai3',
     'JamesKI79366023',
     'sul10x',
     'LJSECU',
     'HibbityJbob',
     'MylesNow',
     'PRYDECommunica1',
     'houck_sherrie',
     'mdhafizulislam0',
     'bluederby1',
     'laurafortified',
     'Chuber82',
     'mcrichierich',
     'dodsonusn',
     'DILIPRO84941461',
     'SgtUsmc1371',
     'OresanyaFavour',
     'sgremminger',
     'Gastonmdaguerre',
     'mats_patrick',
     'Jeffery30786578',
     'bellesbullies',
     'EthansDrunkMom',
     'RaincoatJedi',
     'TheMikeJaeger',
     'dwskipper13',
     'OusmanT10747909',
     'AngeloTheAggie',
     'GBFbxVcMp5bJlbm',
     'GotzeTommy',
     'DeepakJ73901913',
     'mediusbip',
     'casanovasexy89',
     'tokibdesigner1',
     'CommonHorseSen2',
     'lundarmstrong',
     'Alfo74782963',
     'NWestSunshine',
     'AbudilWww',
     'julawu2',
     'RoyBlunt',
     '_NetImpact',
     'Diswyn',
     'BusyJBusy',
     'teri24740',
     'erykahmisan',
     'NahokoA',
     'fran_chambers',
     'Byron69800218',
     'babe_elder',
     'buwaba1',
     'camogary',
     'buckeyejay79',
     'NelsonCBrendan1',
     'MikeRCatania',
     'ansbata',
     'marygerdt',
     'drewbuff6',
     'slmcguire58',
     'earntopcoins',
     'antonkarpp',
     'JohnOmuyonga',
     'ezekielbhatti22',
     'DirkClinton',
     'wrightsophia704',
     'SanFranciscoSis',
     'FranklinBen13',
     'designers00',
     'RECathey',
     'trevtreker',
     'MineheadNanset',
     'TJ_Cox',
     'TheKronah',
     'RoueRandom',
     'SophiaMountain',
     'Eric50900185',
     'KravchenkoMd',
     'ochoajrdaniel',
     'sovereignity77',
     'khurramsaleem04',
     'Air7sparky',
     'MacdonaldChua',
     'Businessguide_B',
     'JosephMSobocin1',
     'sills82',
     'Whiskyandcigars',
     ...]




```python
results=[]
for user in list(df_users.follower)[:5]:

    fName = 'tweets_'+user+'.json' # we'll store the tweets in a text file.
    with open(fName, 'r') as f:
        for i,line in enumerate(f):
            results.append(json.loads(line))

```


```python
len(results)
```




    164




```python
Dataset=toDataFrame(results)
```


    ---------------------------------------------------------------------------

    AttributeError                            Traceback (most recent call last)

    <ipython-input-92-9338351ab4c4> in <module>()
    ----> 1 Dataset=toDataFrame(results)


    <ipython-input-72-faf30aeb64d8> in toDataFrame(tweets)
          3     DataSet = pd.DataFrame()
          4
    ----> 5     DataSet['tweetID'] = [tweet.id for tweet in tweets]
          6     DataSet['tweetText'] = [tweet.text for tweet in tweets]
          7     DataSet['tweetRetweetCt'] = [tweet.retweet_count for tweet in tweets]


    <ipython-input-72-faf30aeb64d8> in <listcomp>(.0)
          3     DataSet = pd.DataFrame()
          4
    ----> 5     DataSet['tweetID'] = [tweet.id for tweet in tweets]
          6     DataSet['tweetText'] = [tweet.text for tweet in tweets]
          7     DataSet['tweetRetweetCt'] = [tweet.retweet_count for tweet in tweets]


    AttributeError: 'dict' object has no attribute 'id'



```python
for result in results:
    df_try=pd.DataFrame(result)
```


```python
df_try
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>contributors</th>
      <th>coordinates</th>
      <th>created_at</th>
      <th>entities</th>
      <th>favorite_count</th>
      <th>favorited</th>
      <th>geo</th>
      <th>id</th>
      <th>id_str</th>
      <th>in_reply_to_screen_name</th>
      <th>...</th>
      <th>is_quote_status</th>
      <th>lang</th>
      <th>metadata</th>
      <th>place</th>
      <th>retweet_count</th>
      <th>retweeted</th>
      <th>source</th>
      <th>text</th>
      <th>truncated</th>
      <th>user</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>contributors_enabled</th>
      <td>None</td>
      <td>None</td>
      <td>Mon Jul 16 13:47:55 +0000 2018</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1018854745799385089</td>
      <td>1018854745799385089</td>
      <td>None</td>
      <td>...</td>
      <td>False</td>
      <td>und</td>
      <td>NaN</td>
      <td>None</td>
      <td>0</td>
      <td>False</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>⁦@realDonaldTrump⁩ ⁦@DonaldJTrumpJr⁩ ⁦@liberty...</td>
      <td>False</td>
      <td>False</td>
    </tr>
    <tr>
      <th>created_at</th>
      <td>None</td>
      <td>None</td>
      <td>Mon Jul 16 13:47:55 +0000 2018</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1018854745799385089</td>
      <td>1018854745799385089</td>
      <td>None</td>
      <td>...</td>
      <td>False</td>
      <td>und</td>
      <td>NaN</td>
      <td>None</td>
      <td>0</td>
      <td>False</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>⁦@realDonaldTrump⁩ ⁦@DonaldJTrumpJr⁩ ⁦@liberty...</td>
      <td>False</td>
      <td>Mon Mar 12 05:56:08 +0000 2018</td>
    </tr>
    <tr>
      <th>default_profile</th>
      <td>None</td>
      <td>None</td>
      <td>Mon Jul 16 13:47:55 +0000 2018</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1018854745799385089</td>
      <td>1018854745799385089</td>
      <td>None</td>
      <td>...</td>
      <td>False</td>
      <td>und</td>
      <td>NaN</td>
      <td>None</td>
      <td>0</td>
      <td>False</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>⁦@realDonaldTrump⁩ ⁦@DonaldJTrumpJr⁩ ⁦@liberty...</td>
      <td>False</td>
      <td>True</td>
    </tr>
    <tr>
      <th>default_profile_image</th>
      <td>None</td>
      <td>None</td>
      <td>Mon Jul 16 13:47:55 +0000 2018</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1018854745799385089</td>
      <td>1018854745799385089</td>
      <td>None</td>
      <td>...</td>
      <td>False</td>
      <td>und</td>
      <td>NaN</td>
      <td>None</td>
      <td>0</td>
      <td>False</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>⁦@realDonaldTrump⁩ ⁦@DonaldJTrumpJr⁩ ⁦@liberty...</td>
      <td>False</td>
      <td>False</td>
    </tr>
    <tr>
      <th>description</th>
      <td>None</td>
      <td>None</td>
      <td>Mon Jul 16 13:47:55 +0000 2018</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1018854745799385089</td>
      <td>1018854745799385089</td>
      <td>None</td>
      <td>...</td>
      <td>False</td>
      <td>und</td>
      <td>NaN</td>
      <td>None</td>
      <td>0</td>
      <td>False</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>⁦@realDonaldTrump⁩ ⁦@DonaldJTrumpJr⁩ ⁦@liberty...</td>
      <td>False</td>
      <td></td>
    </tr>
    <tr>
      <th>entities</th>
      <td>None</td>
      <td>None</td>
      <td>Mon Jul 16 13:47:55 +0000 2018</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1018854745799385089</td>
      <td>1018854745799385089</td>
      <td>None</td>
      <td>...</td>
      <td>False</td>
      <td>und</td>
      <td>NaN</td>
      <td>None</td>
      <td>0</td>
      <td>False</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>⁦@realDonaldTrump⁩ ⁦@DonaldJTrumpJr⁩ ⁦@liberty...</td>
      <td>False</td>
      <td>{'description': {'urls': []}}</td>
    </tr>
    <tr>
      <th>favourites_count</th>
      <td>None</td>
      <td>None</td>
      <td>Mon Jul 16 13:47:55 +0000 2018</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1018854745799385089</td>
      <td>1018854745799385089</td>
      <td>None</td>
      <td>...</td>
      <td>False</td>
      <td>und</td>
      <td>NaN</td>
      <td>None</td>
      <td>0</td>
      <td>False</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>⁦@realDonaldTrump⁩ ⁦@DonaldJTrumpJr⁩ ⁦@liberty...</td>
      <td>False</td>
      <td>26</td>
    </tr>
    <tr>
      <th>follow_request_sent</th>
      <td>None</td>
      <td>None</td>
      <td>Mon Jul 16 13:47:55 +0000 2018</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1018854745799385089</td>
      <td>1018854745799385089</td>
      <td>None</td>
      <td>...</td>
      <td>False</td>
      <td>und</td>
      <td>NaN</td>
      <td>None</td>
      <td>0</td>
      <td>False</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>⁦@realDonaldTrump⁩ ⁦@DonaldJTrumpJr⁩ ⁦@liberty...</td>
      <td>False</td>
      <td>None</td>
    </tr>
    <tr>
      <th>followers_count</th>
      <td>None</td>
      <td>None</td>
      <td>Mon Jul 16 13:47:55 +0000 2018</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1018854745799385089</td>
      <td>1018854745799385089</td>
      <td>None</td>
      <td>...</td>
      <td>False</td>
      <td>und</td>
      <td>NaN</td>
      <td>None</td>
      <td>0</td>
      <td>False</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>⁦@realDonaldTrump⁩ ⁦@DonaldJTrumpJr⁩ ⁦@liberty...</td>
      <td>False</td>
      <td>5</td>
    </tr>
    <tr>
      <th>following</th>
      <td>None</td>
      <td>None</td>
      <td>Mon Jul 16 13:47:55 +0000 2018</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1018854745799385089</td>
      <td>1018854745799385089</td>
      <td>None</td>
      <td>...</td>
      <td>False</td>
      <td>und</td>
      <td>NaN</td>
      <td>None</td>
      <td>0</td>
      <td>False</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>⁦@realDonaldTrump⁩ ⁦@DonaldJTrumpJr⁩ ⁦@liberty...</td>
      <td>False</td>
      <td>None</td>
    </tr>
    <tr>
      <th>friends_count</th>
      <td>None</td>
      <td>None</td>
      <td>Mon Jul 16 13:47:55 +0000 2018</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1018854745799385089</td>
      <td>1018854745799385089</td>
      <td>None</td>
      <td>...</td>
      <td>False</td>
      <td>und</td>
      <td>NaN</td>
      <td>None</td>
      <td>0</td>
      <td>False</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>⁦@realDonaldTrump⁩ ⁦@DonaldJTrumpJr⁩ ⁦@liberty...</td>
      <td>False</td>
      <td>49</td>
    </tr>
    <tr>
      <th>geo_enabled</th>
      <td>None</td>
      <td>None</td>
      <td>Mon Jul 16 13:47:55 +0000 2018</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1018854745799385089</td>
      <td>1018854745799385089</td>
      <td>None</td>
      <td>...</td>
      <td>False</td>
      <td>und</td>
      <td>NaN</td>
      <td>None</td>
      <td>0</td>
      <td>False</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>⁦@realDonaldTrump⁩ ⁦@DonaldJTrumpJr⁩ ⁦@liberty...</td>
      <td>False</td>
      <td>False</td>
    </tr>
    <tr>
      <th>has_extended_profile</th>
      <td>None</td>
      <td>None</td>
      <td>Mon Jul 16 13:47:55 +0000 2018</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1018854745799385089</td>
      <td>1018854745799385089</td>
      <td>None</td>
      <td>...</td>
      <td>False</td>
      <td>und</td>
      <td>NaN</td>
      <td>None</td>
      <td>0</td>
      <td>False</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>⁦@realDonaldTrump⁩ ⁦@DonaldJTrumpJr⁩ ⁦@liberty...</td>
      <td>False</td>
      <td>False</td>
    </tr>
    <tr>
      <th>hashtags</th>
      <td>None</td>
      <td>None</td>
      <td>Mon Jul 16 13:47:55 +0000 2018</td>
      <td>[]</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1018854745799385089</td>
      <td>1018854745799385089</td>
      <td>None</td>
      <td>...</td>
      <td>False</td>
      <td>und</td>
      <td>NaN</td>
      <td>None</td>
      <td>0</td>
      <td>False</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>⁦@realDonaldTrump⁩ ⁦@DonaldJTrumpJr⁩ ⁦@liberty...</td>
      <td>False</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>id</th>
      <td>None</td>
      <td>None</td>
      <td>Mon Jul 16 13:47:55 +0000 2018</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1018854745799385089</td>
      <td>1018854745799385089</td>
      <td>None</td>
      <td>...</td>
      <td>False</td>
      <td>und</td>
      <td>NaN</td>
      <td>None</td>
      <td>0</td>
      <td>False</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>⁦@realDonaldTrump⁩ ⁦@DonaldJTrumpJr⁩ ⁦@liberty...</td>
      <td>False</td>
      <td>973075145827999744</td>
    </tr>
    <tr>
      <th>id_str</th>
      <td>None</td>
      <td>None</td>
      <td>Mon Jul 16 13:47:55 +0000 2018</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1018854745799385089</td>
      <td>1018854745799385089</td>
      <td>None</td>
      <td>...</td>
      <td>False</td>
      <td>und</td>
      <td>NaN</td>
      <td>None</td>
      <td>0</td>
      <td>False</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>⁦@realDonaldTrump⁩ ⁦@DonaldJTrumpJr⁩ ⁦@liberty...</td>
      <td>False</td>
      <td>973075145827999744</td>
    </tr>
    <tr>
      <th>is_translation_enabled</th>
      <td>None</td>
      <td>None</td>
      <td>Mon Jul 16 13:47:55 +0000 2018</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1018854745799385089</td>
      <td>1018854745799385089</td>
      <td>None</td>
      <td>...</td>
      <td>False</td>
      <td>und</td>
      <td>NaN</td>
      <td>None</td>
      <td>0</td>
      <td>False</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>⁦@realDonaldTrump⁩ ⁦@DonaldJTrumpJr⁩ ⁦@liberty...</td>
      <td>False</td>
      <td>False</td>
    </tr>
    <tr>
      <th>is_translator</th>
      <td>None</td>
      <td>None</td>
      <td>Mon Jul 16 13:47:55 +0000 2018</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1018854745799385089</td>
      <td>1018854745799385089</td>
      <td>None</td>
      <td>...</td>
      <td>False</td>
      <td>und</td>
      <td>NaN</td>
      <td>None</td>
      <td>0</td>
      <td>False</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>⁦@realDonaldTrump⁩ ⁦@DonaldJTrumpJr⁩ ⁦@liberty...</td>
      <td>False</td>
      <td>False</td>
    </tr>
    <tr>
      <th>iso_language_code</th>
      <td>None</td>
      <td>None</td>
      <td>Mon Jul 16 13:47:55 +0000 2018</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1018854745799385089</td>
      <td>1018854745799385089</td>
      <td>None</td>
      <td>...</td>
      <td>False</td>
      <td>und</td>
      <td>und</td>
      <td>None</td>
      <td>0</td>
      <td>False</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>⁦@realDonaldTrump⁩ ⁦@DonaldJTrumpJr⁩ ⁦@liberty...</td>
      <td>False</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>lang</th>
      <td>None</td>
      <td>None</td>
      <td>Mon Jul 16 13:47:55 +0000 2018</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1018854745799385089</td>
      <td>1018854745799385089</td>
      <td>None</td>
      <td>...</td>
      <td>False</td>
      <td>und</td>
      <td>NaN</td>
      <td>None</td>
      <td>0</td>
      <td>False</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>⁦@realDonaldTrump⁩ ⁦@DonaldJTrumpJr⁩ ⁦@liberty...</td>
      <td>False</td>
      <td>en</td>
    </tr>
    <tr>
      <th>listed_count</th>
      <td>None</td>
      <td>None</td>
      <td>Mon Jul 16 13:47:55 +0000 2018</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1018854745799385089</td>
      <td>1018854745799385089</td>
      <td>None</td>
      <td>...</td>
      <td>False</td>
      <td>und</td>
      <td>NaN</td>
      <td>None</td>
      <td>0</td>
      <td>False</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>⁦@realDonaldTrump⁩ ⁦@DonaldJTrumpJr⁩ ⁦@liberty...</td>
      <td>False</td>
      <td>0</td>
    </tr>
    <tr>
      <th>location</th>
      <td>None</td>
      <td>None</td>
      <td>Mon Jul 16 13:47:55 +0000 2018</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1018854745799385089</td>
      <td>1018854745799385089</td>
      <td>None</td>
      <td>...</td>
      <td>False</td>
      <td>und</td>
      <td>NaN</td>
      <td>None</td>
      <td>0</td>
      <td>False</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>⁦@realDonaldTrump⁩ ⁦@DonaldJTrumpJr⁩ ⁦@liberty...</td>
      <td>False</td>
      <td></td>
    </tr>
    <tr>
      <th>name</th>
      <td>None</td>
      <td>None</td>
      <td>Mon Jul 16 13:47:55 +0000 2018</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1018854745799385089</td>
      <td>1018854745799385089</td>
      <td>None</td>
      <td>...</td>
      <td>False</td>
      <td>und</td>
      <td>NaN</td>
      <td>None</td>
      <td>0</td>
      <td>False</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>⁦@realDonaldTrump⁩ ⁦@DonaldJTrumpJr⁩ ⁦@liberty...</td>
      <td>False</td>
      <td>BullBustin1791</td>
    </tr>
    <tr>
      <th>notifications</th>
      <td>None</td>
      <td>None</td>
      <td>Mon Jul 16 13:47:55 +0000 2018</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1018854745799385089</td>
      <td>1018854745799385089</td>
      <td>None</td>
      <td>...</td>
      <td>False</td>
      <td>und</td>
      <td>NaN</td>
      <td>None</td>
      <td>0</td>
      <td>False</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>⁦@realDonaldTrump⁩ ⁦@DonaldJTrumpJr⁩ ⁦@liberty...</td>
      <td>False</td>
      <td>None</td>
    </tr>
    <tr>
      <th>profile_background_color</th>
      <td>None</td>
      <td>None</td>
      <td>Mon Jul 16 13:47:55 +0000 2018</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1018854745799385089</td>
      <td>1018854745799385089</td>
      <td>None</td>
      <td>...</td>
      <td>False</td>
      <td>und</td>
      <td>NaN</td>
      <td>None</td>
      <td>0</td>
      <td>False</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>⁦@realDonaldTrump⁩ ⁦@DonaldJTrumpJr⁩ ⁦@liberty...</td>
      <td>False</td>
      <td>F5F8FA</td>
    </tr>
    <tr>
      <th>profile_background_image_url</th>
      <td>None</td>
      <td>None</td>
      <td>Mon Jul 16 13:47:55 +0000 2018</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1018854745799385089</td>
      <td>1018854745799385089</td>
      <td>None</td>
      <td>...</td>
      <td>False</td>
      <td>und</td>
      <td>NaN</td>
      <td>None</td>
      <td>0</td>
      <td>False</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>⁦@realDonaldTrump⁩ ⁦@DonaldJTrumpJr⁩ ⁦@liberty...</td>
      <td>False</td>
      <td>None</td>
    </tr>
    <tr>
      <th>profile_background_image_url_https</th>
      <td>None</td>
      <td>None</td>
      <td>Mon Jul 16 13:47:55 +0000 2018</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1018854745799385089</td>
      <td>1018854745799385089</td>
      <td>None</td>
      <td>...</td>
      <td>False</td>
      <td>und</td>
      <td>NaN</td>
      <td>None</td>
      <td>0</td>
      <td>False</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>⁦@realDonaldTrump⁩ ⁦@DonaldJTrumpJr⁩ ⁦@liberty...</td>
      <td>False</td>
      <td>None</td>
    </tr>
    <tr>
      <th>profile_background_tile</th>
      <td>None</td>
      <td>None</td>
      <td>Mon Jul 16 13:47:55 +0000 2018</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1018854745799385089</td>
      <td>1018854745799385089</td>
      <td>None</td>
      <td>...</td>
      <td>False</td>
      <td>und</td>
      <td>NaN</td>
      <td>None</td>
      <td>0</td>
      <td>False</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>⁦@realDonaldTrump⁩ ⁦@DonaldJTrumpJr⁩ ⁦@liberty...</td>
      <td>False</td>
      <td>False</td>
    </tr>
    <tr>
      <th>profile_image_url</th>
      <td>None</td>
      <td>None</td>
      <td>Mon Jul 16 13:47:55 +0000 2018</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1018854745799385089</td>
      <td>1018854745799385089</td>
      <td>None</td>
      <td>...</td>
      <td>False</td>
      <td>und</td>
      <td>NaN</td>
      <td>None</td>
      <td>0</td>
      <td>False</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>⁦@realDonaldTrump⁩ ⁦@DonaldJTrumpJr⁩ ⁦@liberty...</td>
      <td>False</td>
      <td>http://pbs.twimg.com/profile_images/9757469503...</td>
    </tr>
    <tr>
      <th>profile_image_url_https</th>
      <td>None</td>
      <td>None</td>
      <td>Mon Jul 16 13:47:55 +0000 2018</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1018854745799385089</td>
      <td>1018854745799385089</td>
      <td>None</td>
      <td>...</td>
      <td>False</td>
      <td>und</td>
      <td>NaN</td>
      <td>None</td>
      <td>0</td>
      <td>False</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>⁦@realDonaldTrump⁩ ⁦@DonaldJTrumpJr⁩ ⁦@liberty...</td>
      <td>False</td>
      <td>https://pbs.twimg.com/profile_images/975746950...</td>
    </tr>
    <tr>
      <th>profile_link_color</th>
      <td>None</td>
      <td>None</td>
      <td>Mon Jul 16 13:47:55 +0000 2018</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1018854745799385089</td>
      <td>1018854745799385089</td>
      <td>None</td>
      <td>...</td>
      <td>False</td>
      <td>und</td>
      <td>NaN</td>
      <td>None</td>
      <td>0</td>
      <td>False</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>⁦@realDonaldTrump⁩ ⁦@DonaldJTrumpJr⁩ ⁦@liberty...</td>
      <td>False</td>
      <td>1DA1F2</td>
    </tr>
    <tr>
      <th>profile_sidebar_border_color</th>
      <td>None</td>
      <td>None</td>
      <td>Mon Jul 16 13:47:55 +0000 2018</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1018854745799385089</td>
      <td>1018854745799385089</td>
      <td>None</td>
      <td>...</td>
      <td>False</td>
      <td>und</td>
      <td>NaN</td>
      <td>None</td>
      <td>0</td>
      <td>False</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>⁦@realDonaldTrump⁩ ⁦@DonaldJTrumpJr⁩ ⁦@liberty...</td>
      <td>False</td>
      <td>C0DEED</td>
    </tr>
    <tr>
      <th>profile_sidebar_fill_color</th>
      <td>None</td>
      <td>None</td>
      <td>Mon Jul 16 13:47:55 +0000 2018</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1018854745799385089</td>
      <td>1018854745799385089</td>
      <td>None</td>
      <td>...</td>
      <td>False</td>
      <td>und</td>
      <td>NaN</td>
      <td>None</td>
      <td>0</td>
      <td>False</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>⁦@realDonaldTrump⁩ ⁦@DonaldJTrumpJr⁩ ⁦@liberty...</td>
      <td>False</td>
      <td>DDEEF6</td>
    </tr>
    <tr>
      <th>profile_text_color</th>
      <td>None</td>
      <td>None</td>
      <td>Mon Jul 16 13:47:55 +0000 2018</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1018854745799385089</td>
      <td>1018854745799385089</td>
      <td>None</td>
      <td>...</td>
      <td>False</td>
      <td>und</td>
      <td>NaN</td>
      <td>None</td>
      <td>0</td>
      <td>False</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>⁦@realDonaldTrump⁩ ⁦@DonaldJTrumpJr⁩ ⁦@liberty...</td>
      <td>False</td>
      <td>333333</td>
    </tr>
    <tr>
      <th>profile_use_background_image</th>
      <td>None</td>
      <td>None</td>
      <td>Mon Jul 16 13:47:55 +0000 2018</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1018854745799385089</td>
      <td>1018854745799385089</td>
      <td>None</td>
      <td>...</td>
      <td>False</td>
      <td>und</td>
      <td>NaN</td>
      <td>None</td>
      <td>0</td>
      <td>False</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>⁦@realDonaldTrump⁩ ⁦@DonaldJTrumpJr⁩ ⁦@liberty...</td>
      <td>False</td>
      <td>True</td>
    </tr>
    <tr>
      <th>protected</th>
      <td>None</td>
      <td>None</td>
      <td>Mon Jul 16 13:47:55 +0000 2018</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1018854745799385089</td>
      <td>1018854745799385089</td>
      <td>None</td>
      <td>...</td>
      <td>False</td>
      <td>und</td>
      <td>NaN</td>
      <td>None</td>
      <td>0</td>
      <td>False</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>⁦@realDonaldTrump⁩ ⁦@DonaldJTrumpJr⁩ ⁦@liberty...</td>
      <td>False</td>
      <td>False</td>
    </tr>
    <tr>
      <th>result_type</th>
      <td>None</td>
      <td>None</td>
      <td>Mon Jul 16 13:47:55 +0000 2018</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1018854745799385089</td>
      <td>1018854745799385089</td>
      <td>None</td>
      <td>...</td>
      <td>False</td>
      <td>und</td>
      <td>recent</td>
      <td>None</td>
      <td>0</td>
      <td>False</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>⁦@realDonaldTrump⁩ ⁦@DonaldJTrumpJr⁩ ⁦@liberty...</td>
      <td>False</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>screen_name</th>
      <td>None</td>
      <td>None</td>
      <td>Mon Jul 16 13:47:55 +0000 2018</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1018854745799385089</td>
      <td>1018854745799385089</td>
      <td>None</td>
      <td>...</td>
      <td>False</td>
      <td>und</td>
      <td>NaN</td>
      <td>None</td>
      <td>0</td>
      <td>False</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>⁦@realDonaldTrump⁩ ⁦@DonaldJTrumpJr⁩ ⁦@liberty...</td>
      <td>False</td>
      <td>bustin1791</td>
    </tr>
    <tr>
      <th>statuses_count</th>
      <td>None</td>
      <td>None</td>
      <td>Mon Jul 16 13:47:55 +0000 2018</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1018854745799385089</td>
      <td>1018854745799385089</td>
      <td>None</td>
      <td>...</td>
      <td>False</td>
      <td>und</td>
      <td>NaN</td>
      <td>None</td>
      <td>0</td>
      <td>False</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>⁦@realDonaldTrump⁩ ⁦@DonaldJTrumpJr⁩ ⁦@liberty...</td>
      <td>False</td>
      <td>194</td>
    </tr>
    <tr>
      <th>symbols</th>
      <td>None</td>
      <td>None</td>
      <td>Mon Jul 16 13:47:55 +0000 2018</td>
      <td>[]</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1018854745799385089</td>
      <td>1018854745799385089</td>
      <td>None</td>
      <td>...</td>
      <td>False</td>
      <td>und</td>
      <td>NaN</td>
      <td>None</td>
      <td>0</td>
      <td>False</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>⁦@realDonaldTrump⁩ ⁦@DonaldJTrumpJr⁩ ⁦@liberty...</td>
      <td>False</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>time_zone</th>
      <td>None</td>
      <td>None</td>
      <td>Mon Jul 16 13:47:55 +0000 2018</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1018854745799385089</td>
      <td>1018854745799385089</td>
      <td>None</td>
      <td>...</td>
      <td>False</td>
      <td>und</td>
      <td>NaN</td>
      <td>None</td>
      <td>0</td>
      <td>False</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>⁦@realDonaldTrump⁩ ⁦@DonaldJTrumpJr⁩ ⁦@liberty...</td>
      <td>False</td>
      <td>None</td>
    </tr>
    <tr>
      <th>translator_type</th>
      <td>None</td>
      <td>None</td>
      <td>Mon Jul 16 13:47:55 +0000 2018</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1018854745799385089</td>
      <td>1018854745799385089</td>
      <td>None</td>
      <td>...</td>
      <td>False</td>
      <td>und</td>
      <td>NaN</td>
      <td>None</td>
      <td>0</td>
      <td>False</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>⁦@realDonaldTrump⁩ ⁦@DonaldJTrumpJr⁩ ⁦@liberty...</td>
      <td>False</td>
      <td>none</td>
    </tr>
    <tr>
      <th>url</th>
      <td>None</td>
      <td>None</td>
      <td>Mon Jul 16 13:47:55 +0000 2018</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1018854745799385089</td>
      <td>1018854745799385089</td>
      <td>None</td>
      <td>...</td>
      <td>False</td>
      <td>und</td>
      <td>NaN</td>
      <td>None</td>
      <td>0</td>
      <td>False</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>⁦@realDonaldTrump⁩ ⁦@DonaldJTrumpJr⁩ ⁦@liberty...</td>
      <td>False</td>
      <td>None</td>
    </tr>
    <tr>
      <th>urls</th>
      <td>None</td>
      <td>None</td>
      <td>Mon Jul 16 13:47:55 +0000 2018</td>
      <td>[]</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1018854745799385089</td>
      <td>1018854745799385089</td>
      <td>None</td>
      <td>...</td>
      <td>False</td>
      <td>und</td>
      <td>NaN</td>
      <td>None</td>
      <td>0</td>
      <td>False</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>⁦@realDonaldTrump⁩ ⁦@DonaldJTrumpJr⁩ ⁦@liberty...</td>
      <td>False</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>user_mentions</th>
      <td>None</td>
      <td>None</td>
      <td>Mon Jul 16 13:47:55 +0000 2018</td>
      <td>[{'id': 25073877, 'id_str': '25073877', 'indic...</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1018854745799385089</td>
      <td>1018854745799385089</td>
      <td>None</td>
      <td>...</td>
      <td>False</td>
      <td>und</td>
      <td>NaN</td>
      <td>None</td>
      <td>0</td>
      <td>False</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>⁦@realDonaldTrump⁩ ⁦@DonaldJTrumpJr⁩ ⁦@liberty...</td>
      <td>False</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>utc_offset</th>
      <td>None</td>
      <td>None</td>
      <td>Mon Jul 16 13:47:55 +0000 2018</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1018854745799385089</td>
      <td>1018854745799385089</td>
      <td>None</td>
      <td>...</td>
      <td>False</td>
      <td>und</td>
      <td>NaN</td>
      <td>None</td>
      <td>0</td>
      <td>False</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>⁦@realDonaldTrump⁩ ⁦@DonaldJTrumpJr⁩ ⁦@liberty...</td>
      <td>False</td>
      <td>None</td>
    </tr>
    <tr>
      <th>verified</th>
      <td>None</td>
      <td>None</td>
      <td>Mon Jul 16 13:47:55 +0000 2018</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1018854745799385089</td>
      <td>1018854745799385089</td>
      <td>None</td>
      <td>...</td>
      <td>False</td>
      <td>und</td>
      <td>NaN</td>
      <td>None</td>
      <td>0</td>
      <td>False</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>⁦@realDonaldTrump⁩ ⁦@DonaldJTrumpJr⁩ ⁦@liberty...</td>
      <td>False</td>
      <td>False</td>
    </tr>
  </tbody>
</table>
<p>47 rows × 24 columns</p>
</div>




```python


            tweets2[user+'_'+str(i)]=json.loads(line)


df_tweets = pd.DataFrame(tweets2).T
```


```python
asdf=pd.DataFrame(dict(df_tweets.user.values))
```


    ---------------------------------------------------------------------------

    ValueError                                Traceback (most recent call last)

    <ipython-input-104-8a65a23dde75> in <module>()
    ----> 1 asdf=pd.DataFrame(dict(df_tweets.user.values))


    ValueError: dictionary update sequence element #0 has length 1332; 2 is required



```python
stop = stopwords.words('english')
```


    ---------------------------------------------------------------------------

    LookupError                               Traceback (most recent call last)

    /anaconda/lib/python3.6/site-packages/nltk/corpus/util.py in __load(self)
         79             except LookupError as e:
    ---> 80                 try: root = nltk.data.find('{}/{}'.format(self.subdir, zip_name))
         81                 except LookupError: raise e


    /anaconda/lib/python3.6/site-packages/nltk/data.py in find(resource_name, paths)
        652     resource_not_found = '\n%s\n%s\n%s' % (sep, msg, sep)
    --> 653     raise LookupError(resource_not_found)
        654


    LookupError:
    **********************************************************************
      Resource 'corpora/stopwords.zip/stopwords/' not found.  Please
      use the NLTK Downloader to obtain the resource:  >>>
      nltk.download()
      Searched in:
        - '/Users/chrislee/nltk_data'
        - '/usr/share/nltk_data'
        - '/usr/local/share/nltk_data'
        - '/usr/lib/nltk_data'
        - '/usr/local/lib/nltk_data'
    **********************************************************************


    During handling of the above exception, another exception occurred:


    LookupError                               Traceback (most recent call last)

    <ipython-input-61-3b5c63e2db5c> in <module>()
    ----> 1 stop = stopwords.words('english')


    /anaconda/lib/python3.6/site-packages/nltk/corpus/util.py in __getattr__(self, attr)
        114             raise AttributeError("LazyCorpusLoader object has no attribute '__bases__'")
        115
    --> 116         self.__load()
        117         # This looks circular, but its not, since __load() changes our
        118         # __class__ to something new:


    /anaconda/lib/python3.6/site-packages/nltk/corpus/util.py in __load(self)
         79             except LookupError as e:
         80                 try: root = nltk.data.find('{}/{}'.format(self.subdir, zip_name))
    ---> 81                 except LookupError: raise e
         82
         83         # Load the corpus.


    /anaconda/lib/python3.6/site-packages/nltk/corpus/util.py in __load(self)
         76         else:
         77             try:
    ---> 78                 root = nltk.data.find('{}/{}'.format(self.subdir, self.__name))
         79             except LookupError as e:
         80                 try: root = nltk.data.find('{}/{}'.format(self.subdir, zip_name))


    /anaconda/lib/python3.6/site-packages/nltk/data.py in find(resource_name, paths)
        651     sep = '*' * 70
        652     resource_not_found = '\n%s\n%s\n%s' % (sep, msg, sep)
    --> 653     raise LookupError(resource_not_found)
        654
        655


    LookupError:
    **********************************************************************
      Resource 'corpora/stopwords' not found.  Please use the NLTK
      Downloader to obtain the resource:  >>> nltk.download()
      Searched in:
        - '/Users/chrislee/nltk_data'
        - '/usr/share/nltk_data'
        - '/usr/local/share/nltk_data'
        - '/usr/lib/nltk_data'
        - '/usr/local/lib/nltk_data'
    **********************************************************************



```python
from nltk.corpus import stopwords
```


```python
stop=set(stopwords.words('english'))
```


    ---------------------------------------------------------------------------

    LookupError                               Traceback (most recent call last)

    /anaconda/lib/python3.6/site-packages/nltk/corpus/util.py in __load(self)
         79             except LookupError as e:
    ---> 80                 try: root = nltk.data.find('{}/{}'.format(self.subdir, zip_name))
         81                 except LookupError: raise e


    /anaconda/lib/python3.6/site-packages/nltk/data.py in find(resource_name, paths)
        652     resource_not_found = '\n%s\n%s\n%s' % (sep, msg, sep)
    --> 653     raise LookupError(resource_not_found)
        654


    LookupError:
    **********************************************************************
      Resource 'corpora/stopwords.zip/stopwords/' not found.  Please
      use the NLTK Downloader to obtain the resource:  >>>
      nltk.download()
      Searched in:
        - '/Users/chrislee/nltk_data'
        - '/usr/share/nltk_data'
        - '/usr/local/share/nltk_data'
        - '/usr/lib/nltk_data'
        - '/usr/local/lib/nltk_data'
    **********************************************************************


    During handling of the above exception, another exception occurred:


    LookupError                               Traceback (most recent call last)

    <ipython-input-67-6b0174d93592> in <module>()
    ----> 1 stop=set(stopwords.words('english'))


    /anaconda/lib/python3.6/site-packages/nltk/corpus/util.py in __getattr__(self, attr)
        114             raise AttributeError("LazyCorpusLoader object has no attribute '__bases__'")
        115
    --> 116         self.__load()
        117         # This looks circular, but its not, since __load() changes our
        118         # __class__ to something new:


    /anaconda/lib/python3.6/site-packages/nltk/corpus/util.py in __load(self)
         79             except LookupError as e:
         80                 try: root = nltk.data.find('{}/{}'.format(self.subdir, zip_name))
    ---> 81                 except LookupError: raise e
         82
         83         # Load the corpus.


    /anaconda/lib/python3.6/site-packages/nltk/corpus/util.py in __load(self)
         76         else:
         77             try:
    ---> 78                 root = nltk.data.find('{}/{}'.format(self.subdir, self.__name))
         79             except LookupError as e:
         80                 try: root = nltk.data.find('{}/{}'.format(self.subdir, zip_name))


    /anaconda/lib/python3.6/site-packages/nltk/data.py in find(resource_name, paths)
        651     sep = '*' * 70
        652     resource_not_found = '\n%s\n%s\n%s' % (sep, msg, sep)
    --> 653     raise LookupError(resource_not_found)
        654
        655


    LookupError:
    **********************************************************************
      Resource 'corpora/stopwords' not found.  Please use the NLTK
      Downloader to obtain the resource:  >>> nltk.download()
      Searched in:
        - '/Users/chrislee/nltk_data'
        - '/usr/share/nltk_data'
        - '/usr/local/share/nltk_data'
        - '/usr/lib/nltk_data'
        - '/usr/local/lib/nltk_data'
    **********************************************************************



```python
nltk.download()
```

    showing info https://raw.githubusercontent.com/nltk/nltk_data/gh-pages/index.xml



    ---------------------------------------------------------------------------

    UnicodeDecodeError                        Traceback (most recent call last)

    <ipython-input-70-a1a554e5d735> in <module>()
    ----> 1 nltk.download()


    /anaconda/lib/python3.6/site-packages/nltk/downloader.py in download(self, info_or_id, download_dir, quiet, force, prefix, halt_on_error, raise_on_error)
        659             # function should make a new copy of self to use?
        660             if download_dir is not None: self._download_dir = download_dir
    --> 661             self._interactive_download()
        662             return True
        663


    /anaconda/lib/python3.6/site-packages/nltk/downloader.py in _interactive_download(self)
        980         if TKINTER:
        981             try:
    --> 982                 DownloaderGUI(self).mainloop()
        983             except TclError:
        984                 DownloaderShell(self).run()


    /anaconda/lib/python3.6/site-packages/nltk/downloader.py in mainloop(self, *args, **kwargs)
       1715
       1716     def mainloop(self, *args, **kwargs):
    -> 1717         self.top.mainloop(*args, **kwargs)
       1718
       1719     #/////////////////////////////////////////////////////////////////


    /anaconda/lib/python3.6/tkinter/__init__.py in mainloop(self, n)
       1275     def mainloop(self, n=0):
       1276         """Call the mainloop of Tk."""
    -> 1277         self.tk.mainloop(n)
       1278     def quit(self):
       1279         """Quit the Tcl interpreter. All widgets will be destroyed."""


    UnicodeDecodeError: 'utf-8' codec can't decode byte 0xff in position 0: invalid start byte



```python
import nltk
```


# This is as far as I've been able to get so far. Everything below here is junk... Feel free to disregard...


```python
df_users=pd.read_csv("users.csv")
```


```python
users_list=list(df_users.users.values)
```


```python
tweets2={}
```


```python
df_tweets=pd.DataFrame()
```


```python
df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>contributors</th>
      <th>coordinates</th>
      <th>created_at</th>
      <th>entities</th>
      <th>extended_entities</th>
      <th>favorite_count</th>
      <th>favorited</th>
      <th>geo</th>
      <th>id</th>
      <th>id_str</th>
      <th>...</th>
      <th>quoted_status_id_str</th>
      <th>retweet_count</th>
      <th>retweeted</th>
      <th>retweeted_status</th>
      <th>source</th>
      <th>text</th>
      <th>truncated</th>
      <th>user</th>
      <th>withheld_in_countries</th>
      <th>screen_name</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>rbtess2_0</th>
      <td>None</td>
      <td>None</td>
      <td>Sun Jul 22 07:21:29 +0000 2018</td>
      <td>{'hashtags': [], 'symbols': [], 'urls': [], 'u...</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1020931823176757248</td>
      <td>1020931823176757248</td>
      <td>...</td>
      <td>NaN</td>
      <td>284</td>
      <td>False</td>
      <td>{'contributors': None, 'coordinates': None, 'c...</td>
      <td>&lt;a href="http://twitter.com" rel="nofollow"&gt;Tw...</td>
      <td>RT @Chicago1Ray: I Remember when there were tw...</td>
      <td>False</td>
      <td>{'contributors_enabled': False, 'created_at': ...</td>
      <td>NaN</td>
      <td>rbtess2_0</td>
    </tr>
    <tr>
      <th>rbtess2_1</th>
      <td>None</td>
      <td>None</td>
      <td>Sun Jul 22 07:20:48 +0000 2018</td>
      <td>{'hashtags': [{'indices': [77, 85], 'text': 'F...</td>
      <td>{'media': [{'display_url': 'pic.twitter.com/xz...</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1020931650023194631</td>
      <td>1020931650023194631</td>
      <td>...</td>
      <td>NaN</td>
      <td>727</td>
      <td>False</td>
      <td>{'contributors': None, 'coordinates': None, 'c...</td>
      <td>&lt;a href="http://twitter.com" rel="nofollow"&gt;Tw...</td>
      <td>RT @USATrump45: RETWEET if you Demand ABC FIRE...</td>
      <td>False</td>
      <td>{'contributors_enabled': False, 'created_at': ...</td>
      <td>NaN</td>
      <td>rbtess2_1</td>
    </tr>
    <tr>
      <th>rbtess2_2</th>
      <td>None</td>
      <td>None</td>
      <td>Sat Jul 21 23:13:17 +0000 2018</td>
      <td>{'hashtags': [], 'symbols': [], 'urls': [], 'u...</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1020808965024813056</td>
      <td>1020808965024813056</td>
      <td>...</td>
      <td>NaN</td>
      <td>819</td>
      <td>False</td>
      <td>{'contributors': None, 'coordinates': None, 'c...</td>
      <td>&lt;a href="http://twitter.com" rel="nofollow"&gt;Tw...</td>
      <td>RT @RealErinCruz: .@potus &amp;amp; @ACLJ we need ...</td>
      <td>False</td>
      <td>{'contributors_enabled': False, 'created_at': ...</td>
      <td>NaN</td>
      <td>rbtess2_2</td>
    </tr>
    <tr>
      <th>rbtess2_3</th>
      <td>None</td>
      <td>None</td>
      <td>Sat Jul 21 23:08:45 +0000 2018</td>
      <td>{'hashtags': [], 'symbols': [], 'urls': [], 'u...</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1020807822248996866</td>
      <td>1020807822248996866</td>
      <td>...</td>
      <td>NaN</td>
      <td>4365</td>
      <td>False</td>
      <td>{'contributors': None, 'coordinates': None, 'c...</td>
      <td>&lt;a href="http://twitter.com" rel="nofollow"&gt;Tw...</td>
      <td>RT @ryteouswretch: First @Ocasio2018 wants to ...</td>
      <td>False</td>
      <td>{'contributors_enabled': False, 'created_at': ...</td>
      <td>NaN</td>
      <td>rbtess2_3</td>
    </tr>
    <tr>
      <th>rbtess2_4</th>
      <td>None</td>
      <td>None</td>
      <td>Sat Jul 21 21:57:31 +0000 2018</td>
      <td>{'hashtags': [], 'media': [{'display_url': 'pi...</td>
      <td>{'media': [{'display_url': 'pic.twitter.com/Ex...</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1020789898230616064</td>
      <td>1020789898230616064</td>
      <td>...</td>
      <td>NaN</td>
      <td>3</td>
      <td>False</td>
      <td>{'contributors': None, 'coordinates': None, 'c...</td>
      <td>&lt;a href="http://twitter.com" rel="nofollow"&gt;Tw...</td>
      <td>RT @darsavmo: https://t.co/ExbrlWsp9p</td>
      <td>False</td>
      <td>{'contributors_enabled': False, 'created_at': ...</td>
      <td>NaN</td>
      <td>rbtess2_4</td>
    </tr>
    <tr>
      <th>rbtess2_5</th>
      <td>None</td>
      <td>None</td>
      <td>Sat Jul 21 21:56:40 +0000 2018</td>
      <td>{'hashtags': [], 'media': [{'display_url': 'pi...</td>
      <td>{'media': [{'display_url': 'pic.twitter.com/XQ...</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1020789681858936832</td>
      <td>1020789681858936832</td>
      <td>...</td>
      <td>NaN</td>
      <td>881</td>
      <td>False</td>
      <td>{'contributors': None, 'coordinates': None, 'c...</td>
      <td>&lt;a href="http://twitter.com" rel="nofollow"&gt;Tw...</td>
      <td>RT @ArizonaKayte: Valid point. https://t.co/XQ...</td>
      <td>False</td>
      <td>{'contributors_enabled': False, 'created_at': ...</td>
      <td>NaN</td>
      <td>rbtess2_5</td>
    </tr>
    <tr>
      <th>rbtess2_6</th>
      <td>None</td>
      <td>None</td>
      <td>Sat Jul 21 21:56:13 +0000 2018</td>
      <td>{'hashtags': [], 'symbols': [], 'urls': [], 'u...</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1020789569661358081</td>
      <td>1020789569661358081</td>
      <td>...</td>
      <td>NaN</td>
      <td>1630</td>
      <td>False</td>
      <td>{'contributors': None, 'coordinates': None, 'c...</td>
      <td>&lt;a href="http://twitter.com" rel="nofollow"&gt;Tw...</td>
      <td>RT @RedNationRising: This will blow your mind\...</td>
      <td>False</td>
      <td>{'contributors_enabled': False, 'created_at': ...</td>
      <td>NaN</td>
      <td>rbtess2_6</td>
    </tr>
    <tr>
      <th>rbtess2_7</th>
      <td>None</td>
      <td>None</td>
      <td>Sat Jul 21 21:55:42 +0000 2018</td>
      <td>{'hashtags': [{'indices': [74, 83], 'text': 'W...</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1020789439088480257</td>
      <td>1020789439088480257</td>
      <td>...</td>
      <td>NaN</td>
      <td>397</td>
      <td>False</td>
      <td>{'contributors': None, 'coordinates': None, 'c...</td>
      <td>&lt;a href="http://twitter.com" rel="nofollow"&gt;Tw...</td>
      <td>RT @KMGGaryde: If you like their Agenda you ca...</td>
      <td>False</td>
      <td>{'contributors_enabled': False, 'created_at': ...</td>
      <td>NaN</td>
      <td>rbtess2_7</td>
    </tr>
    <tr>
      <th>rbtess2_8</th>
      <td>None</td>
      <td>None</td>
      <td>Sat Jul 21 21:54:22 +0000 2018</td>
      <td>{'hashtags': [{'indices': [90, 99], 'text': 'W...</td>
      <td>{'media': [{'display_url': 'pic.twitter.com/CS...</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1020789106366967809</td>
      <td>1020789106366967809</td>
      <td>...</td>
      <td>NaN</td>
      <td>666</td>
      <td>False</td>
      <td>{'contributors': None, 'coordinates': None, 'c...</td>
      <td>&lt;a href="http://twitter.com" rel="nofollow"&gt;Tw...</td>
      <td>RT @Trump454545: Download this image to keep a...</td>
      <td>False</td>
      <td>{'contributors_enabled': False, 'created_at': ...</td>
      <td>NaN</td>
      <td>rbtess2_8</td>
    </tr>
    <tr>
      <th>rbtess2_9</th>
      <td>None</td>
      <td>None</td>
      <td>Sat Jul 21 21:53:59 +0000 2018</td>
      <td>{'hashtags': [], 'symbols': [], 'urls': [], 'u...</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1020789009340125184</td>
      <td>1020789009340125184</td>
      <td>...</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>NaN</td>
      <td>&lt;a href="http://twitter.com" rel="nofollow"&gt;Tw...</td>
      <td>@NFL stop playing ,,,, boycott</td>
      <td>False</td>
      <td>{'contributors_enabled': False, 'created_at': ...</td>
      <td>NaN</td>
      <td>rbtess2_9</td>
    </tr>
    <tr>
      <th>rbtess2_10</th>
      <td>None</td>
      <td>None</td>
      <td>Sat Jul 21 16:58:57 +0000 2018</td>
      <td>{'hashtags': [], 'media': [{'display_url': 'pi...</td>
      <td>{'media': [{'display_url': 'pic.twitter.com/ug...</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1020714761565220864</td>
      <td>1020714761565220864</td>
      <td>...</td>
      <td>NaN</td>
      <td>278</td>
      <td>False</td>
      <td>{'contributors': None, 'coordinates': None, 'c...</td>
      <td>&lt;a href="http://twitter.com" rel="nofollow"&gt;Tw...</td>
      <td>RT @RichardTBurnett: Whoopi’s Friends....👍🏻😎 h...</td>
      <td>False</td>
      <td>{'contributors_enabled': False, 'created_at': ...</td>
      <td>NaN</td>
      <td>rbtess2_10</td>
    </tr>
    <tr>
      <th>rbtess2_11</th>
      <td>None</td>
      <td>None</td>
      <td>Sat Jul 21 16:58:33 +0000 2018</td>
      <td>{'hashtags': [], 'symbols': [], 'urls': [], 'u...</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1020714660046286849</td>
      <td>1020714660046286849</td>
      <td>...</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>NaN</td>
      <td>&lt;a href="http://twitter.com" rel="nofollow"&gt;Tw...</td>
      <td>@NFL boycott</td>
      <td>False</td>
      <td>{'contributors_enabled': False, 'created_at': ...</td>
      <td>NaN</td>
      <td>rbtess2_11</td>
    </tr>
    <tr>
      <th>rbtess2_12</th>
      <td>None</td>
      <td>None</td>
      <td>Sat Jul 21 16:57:55 +0000 2018</td>
      <td>{'hashtags': [{'indices': [88, 97], 'text': 'W...</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1020714499173748736</td>
      <td>1020714499173748736</td>
      <td>...</td>
      <td>NaN</td>
      <td>1827</td>
      <td>False</td>
      <td>{'contributors': None, 'coordinates': None, 'c...</td>
      <td>&lt;a href="http://twitter.com" rel="nofollow"&gt;Tw...</td>
      <td>RT @AMErikaNGIRLBOT: Well, you heard her !!! “...</td>
      <td>False</td>
      <td>{'contributors_enabled': False, 'created_at': ...</td>
      <td>NaN</td>
      <td>rbtess2_12</td>
    </tr>
    <tr>
      <th>rbtess2_13</th>
      <td>None</td>
      <td>None</td>
      <td>Sat Jul 21 16:57:01 +0000 2018</td>
      <td>{'hashtags': [], 'symbols': [], 'urls': [], 'u...</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1020714274430349314</td>
      <td>1020714274430349314</td>
      <td>...</td>
      <td>NaN</td>
      <td>2494</td>
      <td>False</td>
      <td>{'contributors': None, 'coordinates': None, 'c...</td>
      <td>&lt;a href="http://twitter.com" rel="nofollow"&gt;Tw...</td>
      <td>RT @smartiekat123: This hate filled, anti-Amer...</td>
      <td>False</td>
      <td>{'contributors_enabled': False, 'created_at': ...</td>
      <td>NaN</td>
      <td>rbtess2_13</td>
    </tr>
    <tr>
      <th>rbtess2_14</th>
      <td>None</td>
      <td>None</td>
      <td>Sat Jul 21 16:56:48 +0000 2018</td>
      <td>{'hashtags': [], 'media': [{'display_url': 'pi...</td>
      <td>{'media': [{'display_url': 'pic.twitter.com/jq...</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1020714221024268288</td>
      <td>1020714221024268288</td>
      <td>...</td>
      <td>NaN</td>
      <td>339</td>
      <td>False</td>
      <td>{'contributors': None, 'coordinates': None, 'c...</td>
      <td>&lt;a href="http://twitter.com" rel="nofollow"&gt;Tw...</td>
      <td>RT @RedNationRising: See the connection? https...</td>
      <td>False</td>
      <td>{'contributors_enabled': False, 'created_at': ...</td>
      <td>NaN</td>
      <td>rbtess2_14</td>
    </tr>
    <tr>
      <th>rbtess2_15</th>
      <td>None</td>
      <td>None</td>
      <td>Sat Jul 21 04:52:30 +0000 2018</td>
      <td>{'hashtags': [{'indices': [18, 25], 'text': 'S...</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1020531944663199744</td>
      <td>1020531944663199744</td>
      <td>...</td>
      <td>NaN</td>
      <td>261</td>
      <td>False</td>
      <td>{'contributors': None, 'coordinates': None, 'c...</td>
      <td>&lt;a href="http://twitter.com" rel="nofollow"&gt;Tw...</td>
      <td>RT @SusanStormXO: #SIGNED ✔️\n@TheView \n\nWe ...</td>
      <td>False</td>
      <td>{'contributors_enabled': False, 'created_at': ...</td>
      <td>NaN</td>
      <td>rbtess2_15</td>
    </tr>
    <tr>
      <th>rbtess2_16</th>
      <td>None</td>
      <td>None</td>
      <td>Sat Jul 21 04:35:46 +0000 2018</td>
      <td>{'hashtags': [{'indices': [15, 30], 'text': 'B...</td>
      <td>{'media': [{'display_url': 'pic.twitter.com/dt...</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1020527730251661313</td>
      <td>1020527730251661313</td>
      <td>...</td>
      <td>NaN</td>
      <td>833</td>
      <td>False</td>
      <td>{'contributors': None, 'coordinates': None, 'c...</td>
      <td>&lt;a href="http://twitter.com" rel="nofollow"&gt;Tw...</td>
      <td>RT @poconomtn: #BoycottTheView\n#FireWhoopie h...</td>
      <td>False</td>
      <td>{'contributors_enabled': False, 'created_at': ...</td>
      <td>NaN</td>
      <td>rbtess2_16</td>
    </tr>
    <tr>
      <th>rbtess2_17</th>
      <td>None</td>
      <td>None</td>
      <td>Sat Jul 21 04:35:33 +0000 2018</td>
      <td>{'hashtags': [], 'media': [{'display_url': 'pi...</td>
      <td>{'media': [{'display_url': 'pic.twitter.com/G7...</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1020527676862365696</td>
      <td>1020527676862365696</td>
      <td>...</td>
      <td>NaN</td>
      <td>19</td>
      <td>False</td>
      <td>{'contributors': None, 'coordinates': None, 'c...</td>
      <td>&lt;a href="http://twitter.com" rel="nofollow"&gt;Tw...</td>
      <td>RT @PaulWestonEden: @Golfinggary5221  https://...</td>
      <td>False</td>
      <td>{'contributors_enabled': False, 'created_at': ...</td>
      <td>NaN</td>
      <td>rbtess2_17</td>
    </tr>
    <tr>
      <th>rbtess2_18</th>
      <td>None</td>
      <td>None</td>
      <td>Sat Jul 21 04:34:43 +0000 2018</td>
      <td>{'hashtags': [], 'symbols': [], 'urls': [], 'u...</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1020527469550555136</td>
      <td>1020527469550555136</td>
      <td>...</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>NaN</td>
      <td>&lt;a href="http://twitter.com" rel="nofollow"&gt;Tw...</td>
      <td>@TheNewArena BOYCOTT</td>
      <td>False</td>
      <td>{'contributors_enabled': False, 'created_at': ...</td>
      <td>NaN</td>
      <td>rbtess2_18</td>
    </tr>
    <tr>
      <th>rbtess2_19</th>
      <td>None</td>
      <td>None</td>
      <td>Sat Jul 21 04:34:09 +0000 2018</td>
      <td>{'hashtags': [], 'symbols': [], 'urls': [], 'u...</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1020527324222164992</td>
      <td>1020527324222164992</td>
      <td>...</td>
      <td>NaN</td>
      <td>3321</td>
      <td>False</td>
      <td>{'contributors': None, 'coordinates': None, 'c...</td>
      <td>&lt;a href="http://twitter.com" rel="nofollow"&gt;Tw...</td>
      <td>RT @_NewMediaSal: THE VIEW ADVERTISERS\nClorox...</td>
      <td>False</td>
      <td>{'contributors_enabled': False, 'created_at': ...</td>
      <td>NaN</td>
      <td>rbtess2_19</td>
    </tr>
    <tr>
      <th>rbtess2_20</th>
      <td>None</td>
      <td>None</td>
      <td>Sat Jul 21 04:33:21 +0000 2018</td>
      <td>{'hashtags': [{'indices': [68, 76], 'text': 'T...</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1020527122971013120</td>
      <td>1020527122971013120</td>
      <td>...</td>
      <td>NaN</td>
      <td>5099</td>
      <td>False</td>
      <td>{'contributors': None, 'coordinates': None, 'c...</td>
      <td>&lt;a href="http://twitter.com" rel="nofollow"&gt;Tw...</td>
      <td>RT @RealMAGASteve: According to sources at ABC...</td>
      <td>False</td>
      <td>{'contributors_enabled': False, 'created_at': ...</td>
      <td>NaN</td>
      <td>rbtess2_20</td>
    </tr>
    <tr>
      <th>rbtess2_21</th>
      <td>None</td>
      <td>None</td>
      <td>Sat Jul 21 04:33:08 +0000 2018</td>
      <td>{'hashtags': [], 'symbols': [], 'urls': [{'dis...</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1020527068654833665</td>
      <td>1020527068654833665</td>
      <td>...</td>
      <td>NaN</td>
      <td>3227</td>
      <td>False</td>
      <td>{'contributors': None, 'coordinates': None, 'c...</td>
      <td>&lt;a href="http://twitter.com" rel="nofollow"&gt;Tw...</td>
      <td>RT @bbusa617: BREAKING: Petition to fire Whoop...</td>
      <td>False</td>
      <td>{'contributors_enabled': False, 'created_at': ...</td>
      <td>NaN</td>
      <td>rbtess2_21</td>
    </tr>
    <tr>
      <th>rbtess2_22</th>
      <td>None</td>
      <td>None</td>
      <td>Sat Jul 21 04:32:17 +0000 2018</td>
      <td>{'hashtags': [], 'symbols': [], 'urls': [], 'u...</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1020526856955727872</td>
      <td>1020526856955727872</td>
      <td>...</td>
      <td>NaN</td>
      <td>1543</td>
      <td>False</td>
      <td>{'contributors': None, 'coordinates': None, 'c...</td>
      <td>&lt;a href="http://twitter.com" rel="nofollow"&gt;Tw...</td>
      <td>RT @RedNationRising: Former CIA Officer and wh...</td>
      <td>False</td>
      <td>{'contributors_enabled': False, 'created_at': ...</td>
      <td>NaN</td>
      <td>rbtess2_22</td>
    </tr>
    <tr>
      <th>rbtess2_23</th>
      <td>None</td>
      <td>None</td>
      <td>Sat Jul 21 04:31:46 +0000 2018</td>
      <td>{'hashtags': [], 'symbols': [], 'urls': [], 'u...</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1020526725724327936</td>
      <td>1020526725724327936</td>
      <td>...</td>
      <td>NaN</td>
      <td>457</td>
      <td>False</td>
      <td>{'contributors': None, 'coordinates': None, 'c...</td>
      <td>&lt;a href="http://twitter.com" rel="nofollow"&gt;Tw...</td>
      <td>RT @John_KissMyBot: Trump To The NFL 👉 I ‘Can’...</td>
      <td>False</td>
      <td>{'contributors_enabled': False, 'created_at': ...</td>
      <td>NaN</td>
      <td>rbtess2_23</td>
    </tr>
    <tr>
      <th>rbtess2_24</th>
      <td>None</td>
      <td>None</td>
      <td>Sat Jul 21 01:57:55 +0000 2018</td>
      <td>{'hashtags': [], 'symbols': [], 'urls': [], 'u...</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1020488009383280640</td>
      <td>1020488009383280640</td>
      <td>...</td>
      <td>NaN</td>
      <td>1629</td>
      <td>False</td>
      <td>{'contributors': None, 'coordinates': None, 'c...</td>
      <td>&lt;a href="http://twitter.com" rel="nofollow"&gt;Tw...</td>
      <td>RT @GaetaSusan: The Election was Rigged for Hi...</td>
      <td>False</td>
      <td>{'contributors_enabled': False, 'created_at': ...</td>
      <td>NaN</td>
      <td>rbtess2_24</td>
    </tr>
    <tr>
      <th>rbtess2_25</th>
      <td>None</td>
      <td>None</td>
      <td>Sat Jul 21 01:57:00 +0000 2018</td>
      <td>{'hashtags': [], 'symbols': [], 'urls': [], 'u...</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1020487775370522624</td>
      <td>1020487775370522624</td>
      <td>...</td>
      <td>NaN</td>
      <td>736</td>
      <td>False</td>
      <td>{'contributors': None, 'coordinates': None, 'c...</td>
      <td>&lt;a href="http://twitter.com" rel="nofollow"&gt;Tw...</td>
      <td>RT @ArizonaKayte: I wonder what laws they'll i...</td>
      <td>False</td>
      <td>{'contributors_enabled': False, 'created_at': ...</td>
      <td>NaN</td>
      <td>rbtess2_25</td>
    </tr>
    <tr>
      <th>rbtess2_26</th>
      <td>None</td>
      <td>None</td>
      <td>Sat Jul 21 01:56:21 +0000 2018</td>
      <td>{'hashtags': [{'indices': [35, 48], 'text': 'L...</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1020487611989876736</td>
      <td>1020487611989876736</td>
      <td>...</td>
      <td>NaN</td>
      <td>64</td>
      <td>False</td>
      <td>{'contributors': None, 'coordinates': None, 'c...</td>
      <td>&lt;a href="http://twitter.com" rel="nofollow"&gt;Tw...</td>
      <td>RT @WeSupport45: We need your help #LivePDNati...</td>
      <td>False</td>
      <td>{'contributors_enabled': False, 'created_at': ...</td>
      <td>NaN</td>
      <td>rbtess2_26</td>
    </tr>
    <tr>
      <th>rbtess2_27</th>
      <td>None</td>
      <td>None</td>
      <td>Fri Jul 20 20:45:50 +0000 2018</td>
      <td>{'hashtags': [{'indices': [31, 59], 'text': 'L...</td>
      <td>{'media': [{'additional_media_info': {'monetiz...</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1020409469824585728</td>
      <td>1020409469824585728</td>
      <td>...</td>
      <td>NaN</td>
      <td>759</td>
      <td>False</td>
      <td>{'contributors': None, 'coordinates': None, 'c...</td>
      <td>&lt;a href="http://twitter.com" rel="nofollow"&gt;Tw...</td>
      <td>RT @RealMAGASteve: Not only is #LiberalismIsAM...</td>
      <td>False</td>
      <td>{'contributors_enabled': False, 'created_at': ...</td>
      <td>NaN</td>
      <td>rbtess2_27</td>
    </tr>
    <tr>
      <th>rbtess2_28</th>
      <td>None</td>
      <td>None</td>
      <td>Fri Jul 20 20:27:40 +0000 2018</td>
      <td>{'hashtags': [{'indices': [21, 29], 'text': 'R...</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1020404896481402880</td>
      <td>1020404896481402880</td>
      <td>...</td>
      <td>NaN</td>
      <td>354</td>
      <td>False</td>
      <td>{'contributors': None, 'coordinates': None, 'c...</td>
      <td>&lt;a href="http://twitter.com" rel="nofollow"&gt;Tw...</td>
      <td>RT @Matthewcogdeill: #RETWEET TO TELL @Twitter...</td>
      <td>False</td>
      <td>{'contributors_enabled': False, 'created_at': ...</td>
      <td>NaN</td>
      <td>rbtess2_28</td>
    </tr>
    <tr>
      <th>rbtess2_29</th>
      <td>None</td>
      <td>None</td>
      <td>Fri Jul 20 20:24:58 +0000 2018</td>
      <td>{'hashtags': [{'indices': [59, 67], 'text': 'L...</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1020404219612975105</td>
      <td>1020404219612975105</td>
      <td>...</td>
      <td>NaN</td>
      <td>4350</td>
      <td>False</td>
      <td>{'contributors': None, 'coordinates': None, 'c...</td>
      <td>&lt;a href="http://twitter.com" rel="nofollow"&gt;Tw...</td>
      <td>RT @KTHopkins: Knife attack on a bus in the Ge...</td>
      <td>False</td>
      <td>{'contributors_enabled': False, 'created_at': ...</td>
      <td>NaN</td>
      <td>rbtess2_29</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>yaaaq1_0</th>
      <td>None</td>
      <td>None</td>
      <td>Sat Jul 21 19:15:39 +0000 2018</td>
      <td>{'hashtags': [], 'symbols': [], 'urls': [], 'u...</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1020749162818736129</td>
      <td>1020749162818736129</td>
      <td>...</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>NaN</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>@ufmradio هذا مو كنه مات</td>
      <td>False</td>
      <td>{'contributors_enabled': False, 'created_at': ...</td>
      <td>NaN</td>
      <td>yaaaq1_0</td>
    </tr>
    <tr>
      <th>yaaaq1_1</th>
      <td>None</td>
      <td>None</td>
      <td>Sat Jul 21 12:22:13 +0000 2018</td>
      <td>{'hashtags': [], 'symbols': [], 'urls': [], 'u...</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1020645116258680832</td>
      <td>1020645116258680832</td>
      <td>...</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>NaN</td>
      <td>&lt;a href="http://twitter.com/download/android" ...</td>
      <td>@nayef965 اتفق جملة وتفصيلا</td>
      <td>False</td>
      <td>{'contributors_enabled': False, 'created_at': ...</td>
      <td>NaN</td>
      <td>yaaaq1_1</td>
    </tr>
    <tr>
      <th>yaaaq1_2</th>
      <td>None</td>
      <td>None</td>
      <td>Fri Jul 20 16:57:26 +0000 2018</td>
      <td>{'hashtags': [], 'symbols': [], 'urls': [], 'u...</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1020351992051453953</td>
      <td>1020351992051453953</td>
      <td>...</td>
      <td>NaN</td>
      <td>8756</td>
      <td>False</td>
      <td>{'contributors': None, 'coordinates': None, 'c...</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>RT @al7umaide: السلام عليكم لو تكرمتو ممكن نوص...</td>
      <td>False</td>
      <td>{'contributors_enabled': False, 'created_at': ...</td>
      <td>NaN</td>
      <td>yaaaq1_2</td>
    </tr>
    <tr>
      <th>yaaaq1_3</th>
      <td>None</td>
      <td>None</td>
      <td>Mon Jul 16 15:26:48 +0000 2018</td>
      <td>{'hashtags': [], 'symbols': [], 'urls': [], 'u...</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1018879630873374721</td>
      <td>1018879630873374721</td>
      <td>...</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>NaN</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>@T8VxPG6PQ4kiMkm @PhoneAmh1 @RAH_M12 @wanaassa...</td>
      <td>False</td>
      <td>{'contributors_enabled': False, 'created_at': ...</td>
      <td>NaN</td>
      <td>yaaaq1_3</td>
    </tr>
    <tr>
      <th>yaaaq1_4</th>
      <td>None</td>
      <td>None</td>
      <td>Mon Jul 16 15:07:32 +0000 2018</td>
      <td>{'hashtags': [], 'symbols': [], 'urls': [], 'u...</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1018874780186038272</td>
      <td>1018874780186038272</td>
      <td>...</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>NaN</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>@PhoneAmh1 @RAH_M12 @wanaassah @sasmari473 @bi...</td>
      <td>False</td>
      <td>{'contributors_enabled': False, 'created_at': ...</td>
      <td>NaN</td>
      <td>yaaaq1_4</td>
    </tr>
    <tr>
      <th>yaaaq1_5</th>
      <td>None</td>
      <td>None</td>
      <td>Mon Jul 16 14:18:33 +0000 2018</td>
      <td>{'hashtags': [{'indices': [0, 25], 'text': 'مس...</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1018862455001362434</td>
      <td>1018862455001362434</td>
      <td>...</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>NaN</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>#مستمرين_ومقاطعين_المراعي\n اكثر المشاركين تغر...</td>
      <td>True</td>
      <td>{'contributors_enabled': False, 'created_at': ...</td>
      <td>NaN</td>
      <td>yaaaq1_5</td>
    </tr>
    <tr>
      <th>yaaaq1_6</th>
      <td>None</td>
      <td>None</td>
      <td>Mon Jul 16 14:15:15 +0000 2018</td>
      <td>{'hashtags': [], 'symbols': [], 'urls': [], 'u...</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1018861625166782466</td>
      <td>1018861625166782466</td>
      <td>...</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>NaN</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>@PhoneAmh1 @RAH_M12 @wanaassah @sasmari473 @bi...</td>
      <td>False</td>
      <td>{'contributors_enabled': False, 'created_at': ...</td>
      <td>NaN</td>
      <td>yaaaq1_6</td>
    </tr>
    <tr>
      <th>yaaaq1_7</th>
      <td>None</td>
      <td>None</td>
      <td>Sun Jul 15 05:38:08 +0000 2018</td>
      <td>{'hashtags': [], 'symbols': [], 'urls': [], 'u...</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1018369102073737216</td>
      <td>1018369102073737216</td>
      <td>...</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>NaN</td>
      <td>&lt;a href="http://twitter.com/download/android" ...</td>
      <td>@MSDAR_NEWS تكلفة المعيشه ممكن لكن الي يجي سيا...</td>
      <td>False</td>
      <td>{'contributors_enabled': False, 'created_at': ...</td>
      <td>NaN</td>
      <td>yaaaq1_7</td>
    </tr>
    <tr>
      <th>yaaaq1_8</th>
      <td>None</td>
      <td>None</td>
      <td>Fri Jul 13 18:19:29 +0000 2018</td>
      <td>{'hashtags': [], 'symbols': [], 'urls': [], 'u...</td>
      <td>NaN</td>
      <td>4</td>
      <td>False</td>
      <td>None</td>
      <td>1017835922476421122</td>
      <td>1017835922476421122</td>
      <td>...</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>NaN</td>
      <td>&lt;a href="http://twitter.com/download/android" ...</td>
      <td>@UberFacts No one feae Fridays we only fear mo...</td>
      <td>False</td>
      <td>{'contributors_enabled': False, 'created_at': ...</td>
      <td>NaN</td>
      <td>yaaaq1_8</td>
    </tr>
    <tr>
      <th>empress_farah_0</th>
      <td>None</td>
      <td>None</td>
      <td>Sat Jul 21 11:04:12 +0000 2018</td>
      <td>{'hashtags': [], 'symbols': [], 'urls': [], 'u...</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1020625483615080448</td>
      <td>1020625483615080448</td>
      <td>...</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>NaN</td>
      <td>&lt;a href="http://twitter.com/download/android" ...</td>
      <td>@sadeghZibakalam معلومه که مردم به خواسته خودش...</td>
      <td>False</td>
      <td>{'contributors_enabled': False, 'created_at': ...</td>
      <td>NaN</td>
      <td>empress_farah_0</td>
    </tr>
    <tr>
      <th>empress_farah_1</th>
      <td>None</td>
      <td>None</td>
      <td>Sat Jul 21 11:01:38 +0000 2018</td>
      <td>{'hashtags': [], 'symbols': [], 'urls': [{'dis...</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1020624840129241088</td>
      <td>1020624840129241088</td>
      <td>...</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>NaN</td>
      <td>&lt;a href="http://twitter.com/download/android" ...</td>
      <td>@sadeghZibakalam معلومه که بعد از براندازی جام...</td>
      <td>True</td>
      <td>{'contributors_enabled': False, 'created_at': ...</td>
      <td>NaN</td>
      <td>empress_farah_1</td>
    </tr>
    <tr>
      <th>empress_farah_2</th>
      <td>None</td>
      <td>None</td>
      <td>Thu Jul 19 12:46:54 +0000 2018</td>
      <td>{'hashtags': [{'indices': [0, 14], 'text': 'Ne...</td>
      <td>{'media': [{'display_url': 'pic.twitter.com/dC...</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1019926555286540289</td>
      <td>1019926555286540289</td>
      <td>...</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>NaN</td>
      <td>&lt;a href="http://twitter.com/download/android" ...</td>
      <td>#NewProfilePic https://t.co/dCCfJrLuKB</td>
      <td>False</td>
      <td>{'contributors_enabled': False, 'created_at': ...</td>
      <td>NaN</td>
      <td>empress_farah_2</td>
    </tr>
    <tr>
      <th>Gombe1Isah_0</th>
      <td>None</td>
      <td>None</td>
      <td>Sat Jul 21 11:49:17 +0000 2018</td>
      <td>{'hashtags': [], 'symbols': [], 'urls': [], 'u...</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1020636830079901697</td>
      <td>1020636830079901697</td>
      <td>...</td>
      <td>NaN</td>
      <td>925</td>
      <td>False</td>
      <td>{'contributors': None, 'coordinates': None, 'c...</td>
      <td>&lt;a href="http://twitter.com/download/android" ...</td>
      <td>RT @arewashams: Every woman deserves a man who...</td>
      <td>False</td>
      <td>{'contributors_enabled': False, 'created_at': ...</td>
      <td>NaN</td>
      <td>Gombe1Isah_0</td>
    </tr>
    <tr>
      <th>Gombe1Isah_1</th>
      <td>None</td>
      <td>None</td>
      <td>Sat Jul 21 00:07:07 +0000 2018</td>
      <td>{'hashtags': [], 'symbols': [], 'urls': [], 'u...</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1020460125289689088</td>
      <td>1020460125289689088</td>
      <td>...</td>
      <td>NaN</td>
      <td>36</td>
      <td>False</td>
      <td>{'contributors': None, 'coordinates': None, 'c...</td>
      <td>&lt;a href="http://twitter.com/download/android" ...</td>
      <td>RT @Tweets2Motivate: Today is a great day to s...</td>
      <td>False</td>
      <td>{'contributors_enabled': False, 'created_at': ...</td>
      <td>NaN</td>
      <td>Gombe1Isah_1</td>
    </tr>
    <tr>
      <th>Gombe1Isah_2</th>
      <td>None</td>
      <td>None</td>
      <td>Sat Jul 21 00:05:48 +0000 2018</td>
      <td>{'hashtags': [], 'media': [{'display_url': 'pi...</td>
      <td>{'media': [{'display_url': 'pic.twitter.com/Hc...</td>
      <td>1</td>
      <td>False</td>
      <td>None</td>
      <td>1020459792849166338</td>
      <td>1020459792849166338</td>
      <td>...</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>NaN</td>
      <td>&lt;a href="http://twitter.com/download/android" ...</td>
      <td>@Binat_Aji  https://t.co/HckHH1DFVR</td>
      <td>False</td>
      <td>{'contributors_enabled': False, 'created_at': ...</td>
      <td>NaN</td>
      <td>Gombe1Isah_2</td>
    </tr>
    <tr>
      <th>Gombe1Isah_3</th>
      <td>None</td>
      <td>None</td>
      <td>Fri Jul 20 14:03:38 +0000 2018</td>
      <td>{'hashtags': [], 'symbols': [], 'urls': [], 'u...</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1020308251030827008</td>
      <td>1020308251030827008</td>
      <td>...</td>
      <td>NaN</td>
      <td>71</td>
      <td>False</td>
      <td>{'contributors': None, 'coordinates': None, 'c...</td>
      <td>&lt;a href="http://twitter.com/download/android" ...</td>
      <td>RT @HQNigerianArmy: The (COAS) Lt Gen TY Burat...</td>
      <td>False</td>
      <td>{'contributors_enabled': False, 'created_at': ...</td>
      <td>NaN</td>
      <td>Gombe1Isah_3</td>
    </tr>
    <tr>
      <th>Gombe1Isah_4</th>
      <td>None</td>
      <td>None</td>
      <td>Fri Jul 20 14:01:09 +0000 2018</td>
      <td>{'hashtags': [], 'symbols': [], 'urls': [], 'u...</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1020307628256264194</td>
      <td>1020307628256264194</td>
      <td>...</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>NaN</td>
      <td>&lt;a href="http://twitter.com/download/android" ...</td>
      <td>@HEDankwambo @BusinessDayNg Congratulations Yo...</td>
      <td>False</td>
      <td>{'contributors_enabled': False, 'created_at': ...</td>
      <td>NaN</td>
      <td>Gombe1Isah_4</td>
    </tr>
    <tr>
      <th>Gombe1Isah_5</th>
      <td>None</td>
      <td>None</td>
      <td>Fri Jul 20 13:57:47 +0000 2018</td>
      <td>{'hashtags': [], 'symbols': [], 'urls': [], 'u...</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1020306779098492928</td>
      <td>1020306779098492928</td>
      <td>...</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>NaN</td>
      <td>&lt;a href="http://twitter.com/download/android" ...</td>
      <td>@ayeesh_chuchu kut lanlakam.</td>
      <td>False</td>
      <td>{'contributors_enabled': False, 'created_at': ...</td>
      <td>NaN</td>
      <td>Gombe1Isah_5</td>
    </tr>
    <tr>
      <th>Gombe1Isah_6</th>
      <td>None</td>
      <td>None</td>
      <td>Fri Jul 20 10:31:27 +0000 2018</td>
      <td>{'hashtags': [], 'symbols': [], 'urls': [], 'u...</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1020254856026755072</td>
      <td>1020254856026755072</td>
      <td>...</td>
      <td>NaN</td>
      <td>34</td>
      <td>False</td>
      <td>{'contributors': None, 'coordinates': None, 'c...</td>
      <td>&lt;a href="http://twitter.com/download/android" ...</td>
      <td>RT @umanafs: Yakai Dan Uwa!!\n\nKasani Sallah ...</td>
      <td>False</td>
      <td>{'contributors_enabled': False, 'created_at': ...</td>
      <td>NaN</td>
      <td>Gombe1Isah_6</td>
    </tr>
    <tr>
      <th>Gombe1Isah_7</th>
      <td>None</td>
      <td>None</td>
      <td>Fri Jul 20 10:21:30 +0000 2018</td>
      <td>{'hashtags': [], 'symbols': [], 'urls': [], 'u...</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1020252350492168192</td>
      <td>1020252350492168192</td>
      <td>...</td>
      <td>NaN</td>
      <td>13</td>
      <td>False</td>
      <td>{'contributors': None, 'coordinates': None, 'c...</td>
      <td>&lt;a href="http://twitter.com/download/android" ...</td>
      <td>RT @CleverQuotez: Never argue with an idiot th...</td>
      <td>False</td>
      <td>{'contributors_enabled': False, 'created_at': ...</td>
      <td>NaN</td>
      <td>Gombe1Isah_7</td>
    </tr>
    <tr>
      <th>Gombe1Isah_8</th>
      <td>None</td>
      <td>None</td>
      <td>Wed Jul 18 10:28:34 +0000 2018</td>
      <td>{'hashtags': [], 'symbols': [], 'urls': [], 'u...</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1019529354441822208</td>
      <td>1019529354441822208</td>
      <td>...</td>
      <td>1019277438675816448</td>
      <td>174</td>
      <td>False</td>
      <td>{'contributors': None, 'coordinates': None, 'c...</td>
      <td>&lt;a href="http://twitter.com/download/android" ...</td>
      <td>RT @TrackHateSpeech: FAKE NEWS!\n\n6 people su...</td>
      <td>False</td>
      <td>{'contributors_enabled': False, 'created_at': ...</td>
      <td>NaN</td>
      <td>Gombe1Isah_8</td>
    </tr>
    <tr>
      <th>Gombe1Isah_9</th>
      <td>None</td>
      <td>None</td>
      <td>Tue Jul 17 18:55:47 +0000 2018</td>
      <td>{'hashtags': [], 'symbols': [], 'urls': [], 'u...</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1019294613050417152</td>
      <td>1019294613050417152</td>
      <td>...</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>NaN</td>
      <td>&lt;a href="http://twitter.com/download/android" ...</td>
      <td>@Madame_Flowy Why do You ask all this questions?</td>
      <td>False</td>
      <td>{'contributors_enabled': False, 'created_at': ...</td>
      <td>NaN</td>
      <td>Gombe1Isah_9</td>
    </tr>
    <tr>
      <th>Gombe1Isah_10</th>
      <td>None</td>
      <td>None</td>
      <td>Tue Jul 17 18:50:30 +0000 2018</td>
      <td>{'hashtags': [], 'symbols': [], 'urls': [], 'u...</td>
      <td>NaN</td>
      <td>1</td>
      <td>False</td>
      <td>None</td>
      <td>1019293281782558720</td>
      <td>1019293281782558720</td>
      <td>...</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>NaN</td>
      <td>&lt;a href="http://twitter.com/download/android" ...</td>
      <td>@itswarenbuffett Kind regard Sir</td>
      <td>False</td>
      <td>{'contributors_enabled': False, 'created_at': ...</td>
      <td>NaN</td>
      <td>Gombe1Isah_10</td>
    </tr>
    <tr>
      <th>Gombe1Isah_11</th>
      <td>None</td>
      <td>None</td>
      <td>Tue Jul 17 18:49:00 +0000 2018</td>
      <td>{'hashtags': [], 'symbols': [], 'urls': [], 'u...</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1019292902810415106</td>
      <td>1019292902810415106</td>
      <td>...</td>
      <td>NaN</td>
      <td>1</td>
      <td>False</td>
      <td>NaN</td>
      <td>&lt;a href="http://twitter.com/download/android" ...</td>
      <td>@umanafs Amin ya Allah</td>
      <td>False</td>
      <td>{'contributors_enabled': False, 'created_at': ...</td>
      <td>NaN</td>
      <td>Gombe1Isah_11</td>
    </tr>
    <tr>
      <th>Gombe1Isah_12</th>
      <td>None</td>
      <td>None</td>
      <td>Tue Jul 17 09:58:16 +0000 2018</td>
      <td>{'hashtags': [], 'symbols': [], 'urls': [], 'u...</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1019159338437566464</td>
      <td>1019159338437566464</td>
      <td>...</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>NaN</td>
      <td>&lt;a href="http://twitter.com/download/android" ...</td>
      <td>@th_saleem Hallaka kobo at gombe</td>
      <td>False</td>
      <td>{'contributors_enabled': False, 'created_at': ...</td>
      <td>NaN</td>
      <td>Gombe1Isah_12</td>
    </tr>
    <tr>
      <th>Gombe1Isah_13</th>
      <td>None</td>
      <td>None</td>
      <td>Fri Jul 13 07:13:41 +0000 2018</td>
      <td>{'hashtags': [], 'symbols': [], 'urls': [{'dis...</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1017668368608964608</td>
      <td>1017668368608964608</td>
      <td>...</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>NaN</td>
      <td>&lt;a href="http://instagram.com" rel="nofollow"&gt;...</td>
      <td>Just posted a photo https://t.co/NZryHlilWG</td>
      <td>False</td>
      <td>{'contributors_enabled': False, 'created_at': ...</td>
      <td>NaN</td>
      <td>Gombe1Isah_13</td>
    </tr>
    <tr>
      <th>Gombe1Isah_14</th>
      <td>None</td>
      <td>None</td>
      <td>Thu Jul 12 23:56:54 +0000 2018</td>
      <td>{'hashtags': [], 'symbols': [], 'urls': [], 'u...</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1017558451764031492</td>
      <td>1017558451764031492</td>
      <td>...</td>
      <td>NaN</td>
      <td>7247</td>
      <td>False</td>
      <td>{'contributors': None, 'coordinates': None, 'c...</td>
      <td>&lt;a href="http://twitter.com/download/android" ...</td>
      <td>RT @Imamofpeace: A massacre of Christians is t...</td>
      <td>False</td>
      <td>{'contributors_enabled': False, 'created_at': ...</td>
      <td>NaN</td>
      <td>Gombe1Isah_14</td>
    </tr>
    <tr>
      <th>Gombe1Isah_15</th>
      <td>None</td>
      <td>None</td>
      <td>Thu Jul 12 18:35:58 +0000 2018</td>
      <td>{'hashtags': [], 'symbols': [], 'urls': [], 'u...</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1017477685646413826</td>
      <td>1017477685646413826</td>
      <td>...</td>
      <td>NaN</td>
      <td>4118</td>
      <td>False</td>
      <td>{'contributors': None, 'coordinates': None, 'c...</td>
      <td>&lt;a href="http://twitter.com/download/android" ...</td>
      <td>RT @itswarenbuffett: Have confidence in your o...</td>
      <td>False</td>
      <td>{'contributors_enabled': False, 'created_at': ...</td>
      <td>NaN</td>
      <td>Gombe1Isah_15</td>
    </tr>
    <tr>
      <th>Gombe1Isah_16</th>
      <td>None</td>
      <td>None</td>
      <td>Thu Jul 12 18:34:49 +0000 2018</td>
      <td>{'hashtags': [], 'symbols': [], 'urls': [], 'u...</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1017477393454391297</td>
      <td>1017477393454391297</td>
      <td>...</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>NaN</td>
      <td>&lt;a href="http://twitter.com/download/android" ...</td>
      <td>@HEDankwambo indeed Sir,\npolitical hygiene is...</td>
      <td>False</td>
      <td>{'contributors_enabled': False, 'created_at': ...</td>
      <td>NaN</td>
      <td>Gombe1Isah_16</td>
    </tr>
    <tr>
      <th>Jitender_shakya_0</th>
      <td>None</td>
      <td>None</td>
      <td>Sat Jul 21 09:50:09 +0000 2018</td>
      <td>{'hashtags': [], 'symbols': [], 'urls': [], 'u...</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1020606850201419777</td>
      <td>1020606850201419777</td>
      <td>...</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>NaN</td>
      <td>&lt;a href="http://twitter.com/download/android" ...</td>
      <td>@rssurjewala Nice</td>
      <td>False</td>
      <td>{'contributors_enabled': False, 'created_at': ...</td>
      <td>NaN</td>
      <td>Jitender_shakya_0</td>
    </tr>
  </tbody>
</table>
<p>28336 rows × 32 columns</p>
</div>




```python
df.screen_name=df.screen_name.apply(lambda x:x[:x.rfind("_")])
```


```python
df.screen_name
```




    rbtess2_0                    rbtess2
    rbtess2_1                    rbtess2
    rbtess2_2                    rbtess2
    rbtess2_3                    rbtess2
    rbtess2_4                    rbtess2
    rbtess2_5                    rbtess2
    rbtess2_6                    rbtess2
    rbtess2_7                    rbtess2
    rbtess2_8                    rbtess2
    rbtess2_9                    rbtess2
    rbtess2_10                   rbtess2
    rbtess2_11                   rbtess2
    rbtess2_12                   rbtess2
    rbtess2_13                   rbtess2
    rbtess2_14                   rbtess2
    rbtess2_15                   rbtess2
    rbtess2_16                   rbtess2
    rbtess2_17                   rbtess2
    rbtess2_18                   rbtess2
    rbtess2_19                   rbtess2
    rbtess2_20                   rbtess2
    rbtess2_21                   rbtess2
    rbtess2_22                   rbtess2
    rbtess2_23                   rbtess2
    rbtess2_24                   rbtess2
    rbtess2_25                   rbtess2
    rbtess2_26                   rbtess2
    rbtess2_27                   rbtess2
    rbtess2_28                   rbtess2
    rbtess2_29                   rbtess2
                              ...       
    yaaaq1_0                      yaaaq1
    yaaaq1_1                      yaaaq1
    yaaaq1_2                      yaaaq1
    yaaaq1_3                      yaaaq1
    yaaaq1_4                      yaaaq1
    yaaaq1_5                      yaaaq1
    yaaaq1_6                      yaaaq1
    yaaaq1_7                      yaaaq1
    yaaaq1_8                      yaaaq1
    empress_farah_0        empress_farah
    empress_farah_1        empress_farah
    empress_farah_2        empress_farah
    Gombe1Isah_0              Gombe1Isah
    Gombe1Isah_1              Gombe1Isah
    Gombe1Isah_2              Gombe1Isah
    Gombe1Isah_3              Gombe1Isah
    Gombe1Isah_4              Gombe1Isah
    Gombe1Isah_5              Gombe1Isah
    Gombe1Isah_6              Gombe1Isah
    Gombe1Isah_7              Gombe1Isah
    Gombe1Isah_8              Gombe1Isah
    Gombe1Isah_9              Gombe1Isah
    Gombe1Isah_10             Gombe1Isah
    Gombe1Isah_11             Gombe1Isah
    Gombe1Isah_12             Gombe1Isah
    Gombe1Isah_13             Gombe1Isah
    Gombe1Isah_14             Gombe1Isah
    Gombe1Isah_15             Gombe1Isah
    Gombe1Isah_16             Gombe1Isah
    Jitender_shakya_0    Jitender_shakya
    Name: screen_name, Length: 28336, dtype: object




```python
df.to_csv("twitterdata.csv")
```


```python
df_tweets
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
    </tr>
  </thead>
  <tbody>
  </tbody>
</table>
</div>




```python

df=pd.DataFrame(tweets2).T
```


```python
df.index
```




    Index(['rbtess2_0', 'rbtess2_1', 'rbtess2_2', 'rbtess2_3', 'rbtess2_4',
           'rbtess2_5', 'rbtess2_6', 'rbtess2_7', 'rbtess2_8', 'rbtess2_9',
           ...
           'Gombe1Isah_8', 'Gombe1Isah_9', 'Gombe1Isah_10', 'Gombe1Isah_11',
           'Gombe1Isah_12', 'Gombe1Isah_13', 'Gombe1Isah_14', 'Gombe1Isah_15',
           'Gombe1Isah_16', 'Jitender_shakya_0'],
          dtype='object', length=28336)




```python
df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>contributors</th>
      <th>coordinates</th>
      <th>created_at</th>
      <th>entities</th>
      <th>extended_entities</th>
      <th>favorite_count</th>
      <th>favorited</th>
      <th>geo</th>
      <th>id</th>
      <th>id_str</th>
      <th>...</th>
      <th>quoted_status_id_str</th>
      <th>retweet_count</th>
      <th>retweeted</th>
      <th>retweeted_status</th>
      <th>source</th>
      <th>text</th>
      <th>truncated</th>
      <th>user</th>
      <th>withheld_in_countries</th>
      <th>screen_name</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>rbtess2_0</th>
      <td>None</td>
      <td>None</td>
      <td>Sun Jul 22 07:21:29 +0000 2018</td>
      <td>{'hashtags': [], 'symbols': [], 'urls': [], 'u...</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1020931823176757248</td>
      <td>1020931823176757248</td>
      <td>...</td>
      <td>NaN</td>
      <td>284</td>
      <td>False</td>
      <td>{'contributors': None, 'coordinates': None, 'c...</td>
      <td>&lt;a href="http://twitter.com" rel="nofollow"&gt;Tw...</td>
      <td>RT @Chicago1Ray: I Remember when there were tw...</td>
      <td>False</td>
      <td>{'contributors_enabled': False, 'created_at': ...</td>
      <td>NaN</td>
      <td>Index(['rbtess2</td>
    </tr>
    <tr>
      <th>rbtess2_1</th>
      <td>None</td>
      <td>None</td>
      <td>Sun Jul 22 07:20:48 +0000 2018</td>
      <td>{'hashtags': [{'indices': [77, 85], 'text': 'F...</td>
      <td>{'media': [{'display_url': 'pic.twitter.com/xz...</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1020931650023194631</td>
      <td>1020931650023194631</td>
      <td>...</td>
      <td>NaN</td>
      <td>727</td>
      <td>False</td>
      <td>{'contributors': None, 'coordinates': None, 'c...</td>
      <td>&lt;a href="http://twitter.com" rel="nofollow"&gt;Tw...</td>
      <td>RT @USATrump45: RETWEET if you Demand ABC FIRE...</td>
      <td>False</td>
      <td>{'contributors_enabled': False, 'created_at': ...</td>
      <td>NaN</td>
      <td>Index(['rbtess2</td>
    </tr>
    <tr>
      <th>rbtess2_2</th>
      <td>None</td>
      <td>None</td>
      <td>Sat Jul 21 23:13:17 +0000 2018</td>
      <td>{'hashtags': [], 'symbols': [], 'urls': [], 'u...</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1020808965024813056</td>
      <td>1020808965024813056</td>
      <td>...</td>
      <td>NaN</td>
      <td>819</td>
      <td>False</td>
      <td>{'contributors': None, 'coordinates': None, 'c...</td>
      <td>&lt;a href="http://twitter.com" rel="nofollow"&gt;Tw...</td>
      <td>RT @RealErinCruz: .@potus &amp;amp; @ACLJ we need ...</td>
      <td>False</td>
      <td>{'contributors_enabled': False, 'created_at': ...</td>
      <td>NaN</td>
      <td>Index(['rbtess2</td>
    </tr>
    <tr>
      <th>rbtess2_3</th>
      <td>None</td>
      <td>None</td>
      <td>Sat Jul 21 23:08:45 +0000 2018</td>
      <td>{'hashtags': [], 'symbols': [], 'urls': [], 'u...</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1020807822248996866</td>
      <td>1020807822248996866</td>
      <td>...</td>
      <td>NaN</td>
      <td>4365</td>
      <td>False</td>
      <td>{'contributors': None, 'coordinates': None, 'c...</td>
      <td>&lt;a href="http://twitter.com" rel="nofollow"&gt;Tw...</td>
      <td>RT @ryteouswretch: First @Ocasio2018 wants to ...</td>
      <td>False</td>
      <td>{'contributors_enabled': False, 'created_at': ...</td>
      <td>NaN</td>
      <td>Index(['rbtess2</td>
    </tr>
    <tr>
      <th>rbtess2_4</th>
      <td>None</td>
      <td>None</td>
      <td>Sat Jul 21 21:57:31 +0000 2018</td>
      <td>{'hashtags': [], 'media': [{'display_url': 'pi...</td>
      <td>{'media': [{'display_url': 'pic.twitter.com/Ex...</td>
      <td>0</td>
      <td>False</td>
      <td>None</td>
      <td>1020789898230616064</td>
      <td>1020789898230616064</td>
      <td>...</td>
      <td>NaN</td>
      <td>3</td>
      <td>False</td>
      <td>{'contributors': None, 'coordinates': None, 'c...</td>
      <td>&lt;a href="http://twitter.com" rel="nofollow"&gt;Tw...</td>
      <td>RT @darsavmo: https://t.co/ExbrlWsp9p</td>
      <td>False</td>
      <td>{'contributors_enabled': False, 'created_at': ...</td>
      <td>NaN</td>
      <td>Index(['rbtess2</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 32 columns</p>
</div>




```python
df['screen_name']=df.index
```


```python
df.screen_name
```




    rbtess2_0                    rbtess2_0
    rbtess2_1                    rbtess2_1
    rbtess2_2                    rbtess2_2
    rbtess2_3                    rbtess2_3
    rbtess2_4                    rbtess2_4
    rbtess2_5                    rbtess2_5
    rbtess2_6                    rbtess2_6
    rbtess2_7                    rbtess2_7
    rbtess2_8                    rbtess2_8
    rbtess2_9                    rbtess2_9
    rbtess2_10                  rbtess2_10
    rbtess2_11                  rbtess2_11
    rbtess2_12                  rbtess2_12
    rbtess2_13                  rbtess2_13
    rbtess2_14                  rbtess2_14
    rbtess2_15                  rbtess2_15
    rbtess2_16                  rbtess2_16
    rbtess2_17                  rbtess2_17
    rbtess2_18                  rbtess2_18
    rbtess2_19                  rbtess2_19
    rbtess2_20                  rbtess2_20
    rbtess2_21                  rbtess2_21
    rbtess2_22                  rbtess2_22
    rbtess2_23                  rbtess2_23
    rbtess2_24                  rbtess2_24
    rbtess2_25                  rbtess2_25
    rbtess2_26                  rbtess2_26
    rbtess2_27                  rbtess2_27
    rbtess2_28                  rbtess2_28
    rbtess2_29                  rbtess2_29
                               ...        
    yaaaq1_0                      yaaaq1_0
    yaaaq1_1                      yaaaq1_1
    yaaaq1_2                      yaaaq1_2
    yaaaq1_3                      yaaaq1_3
    yaaaq1_4                      yaaaq1_4
    yaaaq1_5                      yaaaq1_5
    yaaaq1_6                      yaaaq1_6
    yaaaq1_7                      yaaaq1_7
    yaaaq1_8                      yaaaq1_8
    empress_farah_0        empress_farah_0
    empress_farah_1        empress_farah_1
    empress_farah_2        empress_farah_2
    Gombe1Isah_0              Gombe1Isah_0
    Gombe1Isah_1              Gombe1Isah_1
    Gombe1Isah_2              Gombe1Isah_2
    Gombe1Isah_3              Gombe1Isah_3
    Gombe1Isah_4              Gombe1Isah_4
    Gombe1Isah_5              Gombe1Isah_5
    Gombe1Isah_6              Gombe1Isah_6
    Gombe1Isah_7              Gombe1Isah_7
    Gombe1Isah_8              Gombe1Isah_8
    Gombe1Isah_9              Gombe1Isah_9
    Gombe1Isah_10            Gombe1Isah_10
    Gombe1Isah_11            Gombe1Isah_11
    Gombe1Isah_12            Gombe1Isah_12
    Gombe1Isah_13            Gombe1Isah_13
    Gombe1Isah_14            Gombe1Isah_14
    Gombe1Isah_15            Gombe1Isah_15
    Gombe1Isah_16            Gombe1Isah_16
    Jitender_shakya_0    Jitender_shakya_0
    Name: screen_name, Length: 28336, dtype: object




```python
len(df)
```




    541




```python
df['follower_screenname']=df.index
```


```python

```


    ---------------------------------------------------------------------------

    ValueError                                Traceback (most recent call last)

    <ipython-input-143-2a4fb521f97a> in <module>()
    ----> 1 df.text.at[0]


    /anaconda/lib/python3.6/site-packages/pandas/core/indexing.py in __getitem__(self, key)
       2139                 raise ValueError('Invalid call for scalar access (getting)!')
       2140
    -> 2141         key = self._convert_key(key)
       2142         return self.obj._get_value(*key, takeable=self._takeable)
       2143


    /anaconda/lib/python3.6/site-packages/pandas/core/indexing.py in _convert_key(self, key, is_setter)
       2225             else:
       2226                 if is_integer(i):
    -> 2227                     raise ValueError("At based indexing on an non-integer "
       2228                                      "index can only have non-integer "
       2229                                      "indexers")


    ValueError: At based indexing on an non-integer index can only have non-integer indexers



```python
fol_df[fol_df=='rbtess2']
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>AmbJohnBolton</th>
      <th>AmbJohnBolton_follower_scores</th>
      <th>HillaryClinton</th>
      <th>HillaryClinton_follower_scores</th>
      <th>SecPompeo</th>
      <th>SecPompeo_follower_scores</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>rbtess2</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>5</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>6</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>7</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>8</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>9</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>10</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>11</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>12</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>13</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>14</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>15</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>16</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>17</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>18</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>19</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>20</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>21</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>22</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>23</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>24</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>25</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>26</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>27</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>28</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>29</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>170</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>171</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>172</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>173</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>174</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>175</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>176</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>177</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>178</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>179</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>180</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>181</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>182</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>183</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>184</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>185</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>186</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>187</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>188</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>189</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>190</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>191</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>192</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>193</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>194</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>195</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>196</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>197</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>198</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>199</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
<p>200 rows × 6 columns</p>
</div>




```python
df.to_csv("tweets_followers.csv")
```


```python
tweets2 = []
with open('tweets.json', 'r') as f:
    for line in f:
        tweets2.append(json.loads(line))

df=pd.DataFrame(tweets2)
```


```python
fol_df.sort_index(axis=1, inplace=True)
```


```python
get_bm_score('zempodzem')
```




    0.32230602190830704




```python
fol_df.to_csv("followers.csv")
```


```python
for
    fol_df.at[i,score_columns]
```




    'rbtess2'




# Access the Twitter API using tweepy (see the provided "Twitter Dev Account Setup Instructions" for details on how to set up a free twitter dev account)


```python
with open("nikki_followers.txt",'r') as f:
    data = f
```


```python


adata=pd.read_csv("nikki_followers.csv")
```


```python
for i in range(600):
    try:
        adata['bm_score'][i]=get_bm_score(adata['screen_name'][i])
    except:
        adata['bm_score'][i]=np.NAN
```


```python
adata.head(20)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>screen_name</th>
      <th>bm_score</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>steelefive</td>
      <td>0.100309</td>
    </tr>
    <tr>
      <th>1</th>
      <td>giantsfan10107</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2</th>
      <td>lives4juicy</td>
      <td>0.286212</td>
    </tr>
    <tr>
      <th>3</th>
      <td>MIANSAL14903091</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4</th>
      <td>danielm54795415</td>
      <td>0.966190</td>
    </tr>
    <tr>
      <th>5</th>
      <td>RG70203818</td>
      <td>0.849586</td>
    </tr>
    <tr>
      <th>6</th>
      <td>MargieArnett</td>
      <td>0.030069</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Smithkarensmit1</td>
      <td>0.944833</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Ixoyecarag1</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Hountainer</td>
      <td>0.935232</td>
    </tr>
    <tr>
      <th>10</th>
      <td>BBCJamesCook</td>
      <td>0.038503</td>
    </tr>
    <tr>
      <th>11</th>
      <td>AChardelle</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>12</th>
      <td>johnbra56438373</td>
      <td>0.896414</td>
    </tr>
    <tr>
      <th>13</th>
      <td>Steven221507</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>14</th>
      <td>gKEQli2QRE7ke8k</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>15</th>
      <td>pbstoinoff</td>
      <td>0.800598</td>
    </tr>
    <tr>
      <th>16</th>
      <td>PKhongpetchsak</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>17</th>
      <td>BrendaDevine</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>18</th>
      <td>akid_</td>
      <td>0.038503</td>
    </tr>
    <tr>
      <th>19</th>
      <td>ruby79775766</td>
      <td>0.904074</td>
    </tr>
  </tbody>
</table>
</div>




```python
adata['bm_score'][1]=1
```

    /anaconda/lib/python3.6/site-packages/ipykernel_launcher.py:1: SettingWithCopyWarning:
    A value is trying to be set on a copy of a slice from a DataFrame

    See the caveats in the documentation: http://pandas.pydata.org/pandas-docs/stable/indexing.html#indexing-view-versus-copy
      """Entry point for launching an IPython kernel.



```python

def get_bm_score(screen_name):  
    score = bom.check_account('@'+screen_name)['scores']['english']
    return score

```


```python
a=get_bm_score('Smithkarensmit1')
```


```python
a
```




    0.9448325758058531




```python
mashape_key = "3Jb4MhTkKnmshUOMY639XsisVlVGp1SQgbFjsnj42uy0GsYZ1T"
twitter_app_auth = {
    'consumer_key': '9Gpcxva2RwolHrDLdhRhYlVln',
    'consumer_secret': 'ylLbTVYjTz6Px2AGV6W662QDKjYDdCuAO3aa6ybStlrCOguK0b',
    'access_token': '867610424690188289-sA0uIW3KLVMro7I7JJmygJXRDhJMWj6',
    'access_token_secret': 'eoBTKR3dGstSziJ6AXr8sdUWdYSOaJXCsbPPKr3adMBE0',
  }

bom = botometer.Botometer(wait_on_ratelimit=True,
                              mashape_key=mashape_key,
                              **twitter_app_auth)

    # Check a single account by screen name
def get_bm_score(screen_name):    
    result = bom.check_account('@ITCrowdsource')
    return
    # Check a single account by id
    #result = bom.check_account(1017839053)

    # Check a sequence of accounts
    # accounts = ['@clayadavis', '@onurvarol', '@jabawack']
    # for screen_name, result in bom.check_accounts_in(accounts):
        # Do stuff with `screen_name` and `result`
```


```python
# import requests
```


```python
user = api.get_user('nikkihaley')
follower_cursor=tweepy.Cursor(api.followers, id="nikkihaley")
```


```python
user = api.get_user('nikkihaley')
follower_cursor=tweepy.Cursor(api.user_timeline, id="nikkihaley")
```


```python
api.user_timeline(id=user)
```


    ---------------------------------------------------------------------------

    TweepError                                Traceback (most recent call last)

    <ipython-input-66-9ee9478e6936> in <module>()
    ----> 1 api.user_timeline(id=user)


    /anaconda/lib/python3.6/site-packages/tweepy/binder.py in _call(*args, **kwargs)
        248             return method
        249         else:
    --> 250             return method.execute()
        251
        252     # Set pagination mode


    /anaconda/lib/python3.6/site-packages/tweepy/binder.py in execute(self)
        232                     raise RateLimitError(error_msg, resp)
        233                 else:
    --> 234                     raise TweepError(error_msg, resp, api_code=api_error_code)
        235
        236             # Parse the response payload


    TweepError: Twitter error response: status code = 431



```python
# http://www.tweepy.org/
import tweepy

# Replace the API_KEY and API_SECRET with your application's key and secret.
auth = tweepy.AppAuthHandler("9Gpcxva2RwolHrDLdhRhYlVln", "ylLbTVYjTz6Px2AGV6W662QDKjYDdCuAO3aa6ybStlrCOguK0b")

api = tweepy.API(auth, wait_on_rate_limit=True, wait_on_rate_limit_notify=True)

if (not api):
    print ("Can't Authenticate")
    sys.exit(-1)
```

# Run the tweepy script to save tweets that include the desired search query to a json file on disk.


```python
follower_cursor.pages()
```


    ---------------------------------------------------------------------------

    TypeError                                 Traceback (most recent call last)

    <ipython-input-20-0e55b7fba131> in <module>()
    ----> 1 len(follower_cursor.pages())


    TypeError: object of type 'CursorIterator' has no len()



```python
print(user.screen_name)
print(user.followers_count)

nikkihaley_followers = user.followers()
for follower in user.followers():
   print(follower.screen_name)
```

    nikkihaley
    1616790
    gKEQli2QRE7ke8k
    pbstoinoff
    PKhongpetchsak
    BrendaDevine
    akid_
    ruby79775766
    Qpatriot55
    BillRunk
    Jibarito4848
    MenForAmerica1
    KhalisaMaria
    dpoythress95
    NGaddesh
    HUANGSINAING
    Stevenbarrack2
    ChrisEnerIdeas
    Alexjvsilva
    alexcampbell0
    JohnSerraoJr
    farzana_waseem



```python
tweepy.Cursor(api.user_timeline).pages(3)
```


```python
len(nikkihaley_followers)
```




    20




```python
import sys
import jsonpickle
import os

searchQuery = 'From:nikkihaley'  # this is what we're searching for
maxTweets = 10 # some arbitrary large number
tweetsPerQry = 10  # this is the max the API permits
fName = 'tweets.json' # we'll store the tweets in a text file.

# If results from a specific ID onwards are reqd, set since_id to that ID.
# else default to no lower limit, go as far back as API allows
sinceId = None

# If results only below a specific ID are, set max_id to that ID.
# else default to no upper limit, start from the most recent tweet matching the search query.
max_id = -1

tweetCount = 0
print("Downloading max {0} tweets".format(maxTweets))
with open(fName, 'w') as f:
    while tweetCount < maxTweets:
        try:
            if (max_id <= 0):
                if (not sinceId):
                    new_tweets = api.search(q=searchQuery, count=tweetsPerQry)
                else:
                    new_tweets = api.search(q=searchQuery, count=tweetsPerQry,
                                            since_id=sinceId)
            else:
                if (not sinceId):
                    new_tweets = api.search(q=searchQuery, count=tweetsPerQry,
                                            max_id=str(max_id - 1))
                else:
                    new_tweets = api.search(q=searchQuery, count=tweetsPerQry,
                                            max_id=str(max_id - 1),
                                            since_id=sinceId)
            if not new_tweets:
                print("No more tweets found")
                break
            for tweet in new_tweets:
                f.write(jsonpickle.encode(tweet._json, unpicklable=False) +
                        '\n')
            tweetCount += len(new_tweets)
            print("Downloaded {0} tweets".format(tweetCount))
            max_id = new_tweets[-1].id
        except tweepy.TweepError as e:
            # Just exit if any error
            print("some error : " + str(e))
            break

print ("Downloaded {0} tweets, Saved to {1}".format(tweetCount, fName))
```

    Downloading max 10 tweets
    Downloaded 10 tweets
    Downloaded 10 tweets, Saved to tweets.json



```python
tweets2 = []
with open('tweets.json', 'r') as f:
    for line in f:
        tweets2.append(json.loads(line))

df=pd.DataFrame(tweets2)
```


```python
df.text
```




    0    RT @CNNOpinion: For Gaza peace, tell the truth...
    1    RT @FoxNews: .@nikkihaley: "We can't do one th...
    2    RT @USUN: Thank you @SecPompeo for spending ti...
    3    RT @USUN: “We can’t do one thing until we see ...
    4    RT @USUN: This morning we joined @SecPompeo in...
    5    RT @USUN: A special thank you to @KayColesJame...
    6    RT @UNWatch: "The @UNHumanRights Council has p...
    7    @Heritage Thank you for having me @KayColesJames!
    8    RT @USUN: Our thoughts on the UN Human Rights ...
    9    Wise words from a wise man. May we live by the...
    Name: text, dtype: object




```python
df=='nikkihaley'
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>contributors</th>
      <th>coordinates</th>
      <th>created_at</th>
      <th>entities</th>
      <th>favorite_count</th>
      <th>favorited</th>
      <th>geo</th>
      <th>id</th>
      <th>id_str</th>
      <th>in_reply_to_screen_name</th>
      <th>...</th>
      <th>lang</th>
      <th>metadata</th>
      <th>place</th>
      <th>retweet_count</th>
      <th>retweeted</th>
      <th>retweeted_status</th>
      <th>source</th>
      <th>text</th>
      <th>truncated</th>
      <th>user</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>...</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
    </tr>
  </tbody>
</table>
<p>1 rows × 25 columns</p>
</div>




```python
user
```




    User(_api=<tweepy.api.API object at 0x109c649e8>, _json={'id': 37666984, 'id_str': '37666984', 'name': 'Nikki Haley', 'screen_name': 'nikkihaley', 'location': 'New York, NY', 'profile_location': None, 'description': '', 'url': 'https://t.co/D1eeLGKP4e', 'entities': {'url': {'urls': [{'url': 'https://t.co/D1eeLGKP4e', 'expanded_url': 'http://Instagr.am/nikkihaley', 'display_url': 'Instagr.am/nikkihaley', 'indices': [0, 23]}]}, 'description': {'urls': []}}, 'protected': False, 'followers_count': 1616860, 'friends_count': 290, 'listed_count': 5371, 'created_at': 'Mon May 04 14:10:21 +0000 2009', 'favourites_count': 3056, 'utc_offset': None, 'time_zone': None, 'geo_enabled': True, 'verified': True, 'statuses_count': 5221, 'lang': 'en', 'status': {'created_at': 'Sat Jul 21 21:30:20 +0000 2018', 'id': 1020783054774824962, 'id_str': '1020783054774824962', 'text': 'RT @CNNOpinion: For Gaza peace, tell the truth about Hamas, write @nikkihaley, @jaredkushner, @USAmbIsrael and @jdgreenblatt45.   https://t…', 'truncated': False, 'entities': {'hashtags': [], 'symbols': [], 'user_mentions': [{'screen_name': 'CNNOpinion', 'name': 'CNN Opinion', 'id': 259074538, 'id_str': '259074538', 'indices': [3, 14]}, {'screen_name': 'nikkihaley', 'name': 'Nikki Haley', 'id': 37666984, 'id_str': '37666984', 'indices': [66, 77]}, {'screen_name': 'jaredkushner', 'name': 'Jared Kushner', 'id': 29547260, 'id_str': '29547260', 'indices': [79, 92]}, {'screen_name': 'USAmbIsrael', 'name': 'David M. Friedman', 'id': 343958195, 'id_str': '343958195', 'indices': [94, 106]}, {'screen_name': 'jdgreenblatt45', 'name': 'Jason D. Greenblatt', 'id': 839864106140192770, 'id_str': '839864106140192770', 'indices': [111, 126]}], 'urls': []}, 'source': '<a href="http://twitter.com/download/iphone" rel="nofollow">Twitter for iPhone</a>', 'in_reply_to_status_id': None, 'in_reply_to_status_id_str': None, 'in_reply_to_user_id': None, 'in_reply_to_user_id_str': None, 'in_reply_to_screen_name': None, 'geo': None, 'coordinates': None, 'place': None, 'contributors': None, 'retweeted_status': {'created_at': 'Sat Jul 21 17:21:03 +0000 2018', 'id': 1020720320364318726, 'id_str': '1020720320364318726', 'text': 'For Gaza peace, tell the truth about Hamas, write @nikkihaley, @jaredkushner, @USAmbIsrael and @jdgreenblatt45.   https://t.co/DsJN6szDZq', 'truncated': False, 'entities': {'hashtags': [], 'symbols': [], 'user_mentions': [{'screen_name': 'nikkihaley', 'name': 'Nikki Haley', 'id': 37666984, 'id_str': '37666984', 'indices': [50, 61]}, {'screen_name': 'jaredkushner', 'name': 'Jared Kushner', 'id': 29547260, 'id_str': '29547260', 'indices': [63, 76]}, {'screen_name': 'USAmbIsrael', 'name': 'David M. Friedman', 'id': 343958195, 'id_str': '343958195', 'indices': [78, 90]}, {'screen_name': 'jdgreenblatt45', 'name': 'Jason D. Greenblatt', 'id': 839864106140192770, 'id_str': '839864106140192770', 'indices': [95, 110]}], 'urls': [{'url': 'https://t.co/DsJN6szDZq', 'expanded_url': 'https://cnn.it/2LtLpN3', 'display_url': 'cnn.it/2LtLpN3', 'indices': [114, 137]}]}, 'source': '<a href="http://twitter.com" rel="nofollow">Twitter Web Client</a>', 'in_reply_to_status_id': None, 'in_reply_to_status_id_str': None, 'in_reply_to_user_id': None, 'in_reply_to_user_id_str': None, 'in_reply_to_screen_name': None, 'geo': None, 'coordinates': None, 'place': None, 'contributors': None, 'is_quote_status': False, 'retweet_count': 237, 'favorite_count': 895, 'favorited': False, 'retweeted': False, 'possibly_sensitive': False, 'lang': 'en'}, 'is_quote_status': False, 'retweet_count': 237, 'favorite_count': 0, 'favorited': False, 'retweeted': False, 'lang': 'en'}, 'contributors_enabled': False, 'is_translator': False, 'is_translation_enabled': False, 'profile_background_color': '333333', 'profile_background_image_url': 'http://abs.twimg.com/images/themes/theme1/bg.png', 'profile_background_image_url_https': 'https://abs.twimg.com/images/themes/theme1/bg.png', 'profile_background_tile': True, 'profile_image_url': 'http://pbs.twimg.com/profile_images/935292090703138816/cvC8C2gR_normal.jpg', 'profile_image_url_https': 'https://pbs.twimg.com/profile_images/935292090703138816/cvC8C2gR_normal.jpg', 'profile_banner_url': 'https://pbs.twimg.com/profile_banners/37666984/1511891672', 'profile_link_color': 'CC0033', 'profile_sidebar_border_color': 'FFFFFF', 'profile_sidebar_fill_color': 'CCCCCC', 'profile_text_color': '000000', 'profile_use_background_image': False, 'has_extended_profile': False, 'default_profile': False, 'default_profile_image': False, 'following': None, 'follow_request_sent': None, 'notifications': None, 'translator_type': 'none'}, id=37666984, id_str='37666984', name='Nikki Haley', screen_name='nikkihaley', location='New York, NY', profile_location=None, description='', url='https://t.co/D1eeLGKP4e', entities={'url': {'urls': [{'url': 'https://t.co/D1eeLGKP4e', 'expanded_url': 'http://Instagr.am/nikkihaley', 'display_url': 'Instagr.am/nikkihaley', 'indices': [0, 23]}]}, 'description': {'urls': []}}, protected=False, followers_count=1616860, friends_count=290, listed_count=5371, created_at=datetime.datetime(2009, 5, 4, 14, 10, 21), favourites_count=3056, utc_offset=None, time_zone=None, geo_enabled=True, verified=True, statuses_count=5221, lang='en', status=Status(_api=<tweepy.api.API object at 0x109c649e8>, _json={'created_at': 'Sat Jul 21 21:30:20 +0000 2018', 'id': 1020783054774824962, 'id_str': '1020783054774824962', 'text': 'RT @CNNOpinion: For Gaza peace, tell the truth about Hamas, write @nikkihaley, @jaredkushner, @USAmbIsrael and @jdgreenblatt45.   https://t…', 'truncated': False, 'entities': {'hashtags': [], 'symbols': [], 'user_mentions': [{'screen_name': 'CNNOpinion', 'name': 'CNN Opinion', 'id': 259074538, 'id_str': '259074538', 'indices': [3, 14]}, {'screen_name': 'nikkihaley', 'name': 'Nikki Haley', 'id': 37666984, 'id_str': '37666984', 'indices': [66, 77]}, {'screen_name': 'jaredkushner', 'name': 'Jared Kushner', 'id': 29547260, 'id_str': '29547260', 'indices': [79, 92]}, {'screen_name': 'USAmbIsrael', 'name': 'David M. Friedman', 'id': 343958195, 'id_str': '343958195', 'indices': [94, 106]}, {'screen_name': 'jdgreenblatt45', 'name': 'Jason D. Greenblatt', 'id': 839864106140192770, 'id_str': '839864106140192770', 'indices': [111, 126]}], 'urls': []}, 'source': '<a href="http://twitter.com/download/iphone" rel="nofollow">Twitter for iPhone</a>', 'in_reply_to_status_id': None, 'in_reply_to_status_id_str': None, 'in_reply_to_user_id': None, 'in_reply_to_user_id_str': None, 'in_reply_to_screen_name': None, 'geo': None, 'coordinates': None, 'place': None, 'contributors': None, 'retweeted_status': {'created_at': 'Sat Jul 21 17:21:03 +0000 2018', 'id': 1020720320364318726, 'id_str': '1020720320364318726', 'text': 'For Gaza peace, tell the truth about Hamas, write @nikkihaley, @jaredkushner, @USAmbIsrael and @jdgreenblatt45.   https://t.co/DsJN6szDZq', 'truncated': False, 'entities': {'hashtags': [], 'symbols': [], 'user_mentions': [{'screen_name': 'nikkihaley', 'name': 'Nikki Haley', 'id': 37666984, 'id_str': '37666984', 'indices': [50, 61]}, {'screen_name': 'jaredkushner', 'name': 'Jared Kushner', 'id': 29547260, 'id_str': '29547260', 'indices': [63, 76]}, {'screen_name': 'USAmbIsrael', 'name': 'David M. Friedman', 'id': 343958195, 'id_str': '343958195', 'indices': [78, 90]}, {'screen_name': 'jdgreenblatt45', 'name': 'Jason D. Greenblatt', 'id': 839864106140192770, 'id_str': '839864106140192770', 'indices': [95, 110]}], 'urls': [{'url': 'https://t.co/DsJN6szDZq', 'expanded_url': 'https://cnn.it/2LtLpN3', 'display_url': 'cnn.it/2LtLpN3', 'indices': [114, 137]}]}, 'source': '<a href="http://twitter.com" rel="nofollow">Twitter Web Client</a>', 'in_reply_to_status_id': None, 'in_reply_to_status_id_str': None, 'in_reply_to_user_id': None, 'in_reply_to_user_id_str': None, 'in_reply_to_screen_name': None, 'geo': None, 'coordinates': None, 'place': None, 'contributors': None, 'is_quote_status': False, 'retweet_count': 237, 'favorite_count': 895, 'favorited': False, 'retweeted': False, 'possibly_sensitive': False, 'lang': 'en'}, 'is_quote_status': False, 'retweet_count': 237, 'favorite_count': 0, 'favorited': False, 'retweeted': False, 'lang': 'en'}, created_at=datetime.datetime(2018, 7, 21, 21, 30, 20), id=1020783054774824962, id_str='1020783054774824962', text='RT @CNNOpinion: For Gaza peace, tell the truth about Hamas, write @nikkihaley, @jaredkushner, @USAmbIsrael and @jdgreenblatt45.   https://t…', truncated=False, entities={'hashtags': [], 'symbols': [], 'user_mentions': [{'screen_name': 'CNNOpinion', 'name': 'CNN Opinion', 'id': 259074538, 'id_str': '259074538', 'indices': [3, 14]}, {'screen_name': 'nikkihaley', 'name': 'Nikki Haley', 'id': 37666984, 'id_str': '37666984', 'indices': [66, 77]}, {'screen_name': 'jaredkushner', 'name': 'Jared Kushner', 'id': 29547260, 'id_str': '29547260', 'indices': [79, 92]}, {'screen_name': 'USAmbIsrael', 'name': 'David M. Friedman', 'id': 343958195, 'id_str': '343958195', 'indices': [94, 106]}, {'screen_name': 'jdgreenblatt45', 'name': 'Jason D. Greenblatt', 'id': 839864106140192770, 'id_str': '839864106140192770', 'indices': [111, 126]}], 'urls': []}, source='Twitter for iPhone', source_url='http://twitter.com/download/iphone', in_reply_to_status_id=None, in_reply_to_status_id_str=None, in_reply_to_user_id=None, in_reply_to_user_id_str=None, in_reply_to_screen_name=None, geo=None, coordinates=None, place=None, contributors=None, retweeted_status=Status(_api=<tweepy.api.API object at 0x109c649e8>, _json={'created_at': 'Sat Jul 21 17:21:03 +0000 2018', 'id': 1020720320364318726, 'id_str': '1020720320364318726', 'text': 'For Gaza peace, tell the truth about Hamas, write @nikkihaley, @jaredkushner, @USAmbIsrael and @jdgreenblatt45.   https://t.co/DsJN6szDZq', 'truncated': False, 'entities': {'hashtags': [], 'symbols': [], 'user_mentions': [{'screen_name': 'nikkihaley', 'name': 'Nikki Haley', 'id': 37666984, 'id_str': '37666984', 'indices': [50, 61]}, {'screen_name': 'jaredkushner', 'name': 'Jared Kushner', 'id': 29547260, 'id_str': '29547260', 'indices': [63, 76]}, {'screen_name': 'USAmbIsrael', 'name': 'David M. Friedman', 'id': 343958195, 'id_str': '343958195', 'indices': [78, 90]}, {'screen_name': 'jdgreenblatt45', 'name': 'Jason D. Greenblatt', 'id': 839864106140192770, 'id_str': '839864106140192770', 'indices': [95, 110]}], 'urls': [{'url': 'https://t.co/DsJN6szDZq', 'expanded_url': 'https://cnn.it/2LtLpN3', 'display_url': 'cnn.it/2LtLpN3', 'indices': [114, 137]}]}, 'source': '<a href="http://twitter.com" rel="nofollow">Twitter Web Client</a>', 'in_reply_to_status_id': None, 'in_reply_to_status_id_str': None, 'in_reply_to_user_id': None, 'in_reply_to_user_id_str': None, 'in_reply_to_screen_name': None, 'geo': None, 'coordinates': None, 'place': None, 'contributors': None, 'is_quote_status': False, 'retweet_count': 237, 'favorite_count': 895, 'favorited': False, 'retweeted': False, 'possibly_sensitive': False, 'lang': 'en'}, created_at=datetime.datetime(2018, 7, 21, 17, 21, 3), id=1020720320364318726, id_str='1020720320364318726', text='For Gaza peace, tell the truth about Hamas, write @nikkihaley, @jaredkushner, @USAmbIsrael and @jdgreenblatt45.   https://t.co/DsJN6szDZq', truncated=False, entities={'hashtags': [], 'symbols': [], 'user_mentions': [{'screen_name': 'nikkihaley', 'name': 'Nikki Haley', 'id': 37666984, 'id_str': '37666984', 'indices': [50, 61]}, {'screen_name': 'jaredkushner', 'name': 'Jared Kushner', 'id': 29547260, 'id_str': '29547260', 'indices': [63, 76]}, {'screen_name': 'USAmbIsrael', 'name': 'David M. Friedman', 'id': 343958195, 'id_str': '343958195', 'indices': [78, 90]}, {'screen_name': 'jdgreenblatt45', 'name': 'Jason D. Greenblatt', 'id': 839864106140192770, 'id_str': '839864106140192770', 'indices': [95, 110]}], 'urls': [{'url': 'https://t.co/DsJN6szDZq', 'expanded_url': 'https://cnn.it/2LtLpN3', 'display_url': 'cnn.it/2LtLpN3', 'indices': [114, 137]}]}, source='Twitter Web Client', source_url='http://twitter.com', in_reply_to_status_id=None, in_reply_to_status_id_str=None, in_reply_to_user_id=None, in_reply_to_user_id_str=None, in_reply_to_screen_name=None, geo=None, coordinates=None, place=None, contributors=None, is_quote_status=False, retweet_count=237, favorite_count=895, favorited=False, retweeted=False, possibly_sensitive=False, lang='en'), is_quote_status=False, retweet_count=237, favorite_count=0, favorited=False, retweeted=False, lang='en'), contributors_enabled=False, is_translator=False, is_translation_enabled=False, profile_background_color='333333', profile_background_image_url='http://abs.twimg.com/images/themes/theme1/bg.png', profile_background_image_url_https='https://abs.twimg.com/images/themes/theme1/bg.png', profile_background_tile=True, profile_image_url='http://pbs.twimg.com/profile_images/935292090703138816/cvC8C2gR_normal.jpg', profile_image_url_https='https://pbs.twimg.com/profile_images/935292090703138816/cvC8C2gR_normal.jpg', profile_banner_url='https://pbs.twimg.com/profile_banners/37666984/1511891672', profile_link_color='CC0033', profile_sidebar_border_color='FFFFFF', profile_sidebar_fill_color='CCCCCC', profile_text_color='000000', profile_use_background_image=False, has_extended_profile=False, default_profile=False, default_profile_image=False, following=False, follow_request_sent=None, notifications=None, translator_type='none')




```python
tweets2 = []
with open('tweets.json', 'r') as f:
    for line in f:
        tweets2.append(json.loads(line))
```


```python
df=pd.DataFrame(tweets2)
```


```python
df.columns
```




    Index(['contributors', 'coordinates', 'created_at', 'entities',
           'favorite_count', 'favorited', 'geo', 'id', 'id_str',
           'in_reply_to_screen_name', 'in_reply_to_status_id',
           'in_reply_to_status_id_str', 'in_reply_to_user_id',
           'in_reply_to_user_id_str', 'is_quote_status', 'lang', 'metadata',
           'place', 'retweet_count', 'retweeted', 'retweeted_status', 'source',
           'text', 'truncated', 'user'],
          dtype='object')




```python
df2=pd.DataFrame(df.user.values)
```


```python
df2
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>0</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>{'contributors_enabled': False, 'created_at': ...</td>
    </tr>
  </tbody>
</table>
</div>




```python
df=pd.DataFrame(data.)
```


      File "<ipython-input-86-7034aec9e491>", line 1
        df=pd.DataFrame(data.)
                             ^
    SyntaxError: invalid syntax




```python
df2=pd.DataFrame(df.user)
```


```python
df.id_str

```




    0      1017839053662801920
    1      1017837401278824448
    2      1017819475104628738
    3      1017816931737456641
    4      1017789848156098560
    5      1017777128971137026
    6      1017760282045710337
    7      1017731161668423680
    8      1017729952203014145
    9      1017721884291592192
    10     1017700676158328832
    11     1017699176795328512
    12     1017684352816308224
    13     1017667431601459200
    14     1017640491482013696
    15     1017608254812311552
    16     1017575710423240706
    17     1017570121169145863
    18     1017557640896016384
    19     1017550941879439360
    20     1017540787503075329
    21     1017540531721854978
    22     1017525725463830528
    23     1017523036109582337
    24     1017522027144630272
    25     1017516520702906368
    26     1017513789854568448
    27     1017505946388647936
    28     1017498262507524097
    29     1017496592033746945
                  ...         
    420    1017164230674481154
    421    1017164226639683584
    422    1017164225876320256
    423    1017164214748762112
    424    1017164199275872256
    425    1017164154598281216
    426    1017164139414732800
    427    1017164123367464961
    428    1017164112906915842
    429    1017164110692220931
    430    1017164098860146689
    431    1017164098310508544
    432    1017164086906245120
    433    1017164083001356288
    434    1017164081965535239
    435    1017164077658001408
    436    1017164063644807168
    437    1017164058812928006
    438    1017164054941450240
    439    1017164046708043776
    440    1017164025161895936
    441    1017164009408221187
    442    1017164002236026880
    443    1017164000327593985
    444    1017163981306388481
    445    1017163967276437504
    446    1017163960477503488
    447    1017163951237365773
    448    1017163947277979649
    449    1017163894580830208
    Name: id_str, Length: 450, dtype: object




```python
mashape_key = "3Jb4MhTkKnmshUOMY639XsisVlVGp1SQgbFjsnj42uy0GsYZ1T"
twitter_app_auth = {
    'consumer_key': '9Gpcxva2RwolHrDLdhRhYlVln',
    'consumer_secret': 'ylLbTVYjTz6Px2AGV6W662QDKjYDdCuAO3aa6ybStlrCOguK0b',
    'access_token': '867610424690188289-sA0uIW3KLVMro7I7JJmygJXRDhJMWj6',
    'access_token_secret': 'eoBTKR3dGstSziJ6AXr8sdUWdYSOaJXCsbPPKr3adMBE0',
  }
bom = botometer.Botometer(wait_on_ratelimit=True,
                          mashape_key=mashape_key,
                          **twitter_app_auth)

# Check a single account by screen name
result = bom.check_account('@ITCrowdsource')

# Check a single account by id
#result = bom.check_account(1017839053)

# Check a sequence of accounts
# accounts = ['@clayadavis', '@onurvarol', '@jabawack']
# for screen_name, result in bom.check_accounts_in(accounts):
    # Do stuff with `screen_name` and `result`
```


```python
result
```




    {'cap': {'english': 0.9615147589542459, 'universal': 0.8731487918871739},
     'categories': {'content': 0.9386073163011215,
      'friend': 0.8896394155645955,
      'network': 0.95020703614119,
      'sentiment': 0.9411933966292786,
      'temporal': 0.8535286114881314,
      'user': 0.9597053361519627},
     'display_scores': {'content': 4.7,
      'english': 4.9,
      'friend': 4.4,
      'network': 4.8,
      'sentiment': 4.7,
      'temporal': 4.3,
      'universal': 4.7,
      'user': 4.8},
     'scores': {'english': 0.9757295961173048, 'universal': 0.9459854875837471},
     'user': {'id_str': '4894792367', 'screen_name': 'ITCrowdsource'}}



# Known Bot Accounts Botometer Score


```python
import matplotlib.pyplot as plt
%matplotlib inline
```


```python
botlist= ['IndustryAi','nautabotnews','alejandronw','ITCrowdsource','spwebtraffic','DaveRaven72','NewWorkshop','threatintelbot','slomogoldfish']
botscores=[]
for bot in botlist:
    score = bom.check_account('@'+bot)['scores']['english']
    botscores.append([bot,score])

```


```python
botscores
```




    [['IndustryAi', 0.8601677371481485],
     ['nautabotnews', 0.8383531326036774],
     ['alejandronw', 0.8138717339737572],
     ['ITCrowdsource', 0.9757295961173048],
     ['spwebtraffic', 0.7238002465135849],
     ['DaveRaven72', 0.5487020083865324],
     ['NewWorkshop', 0.7565854729457974],
     ['threatintelbot', 0.9240939964164224],
     ['slomogoldfish', 0.6310548366819692]]




```python
bot_df=pd.DataFrame(botscores, columns=["username","score"])
bot_df
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>username</th>
      <th>score</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>IndustryAi</td>
      <td>0.860168</td>
    </tr>
    <tr>
      <th>1</th>
      <td>nautabotnews</td>
      <td>0.838353</td>
    </tr>
    <tr>
      <th>2</th>
      <td>alejandronw</td>
      <td>0.813872</td>
    </tr>
    <tr>
      <th>3</th>
      <td>ITCrowdsource</td>
      <td>0.975730</td>
    </tr>
    <tr>
      <th>4</th>
      <td>spwebtraffic</td>
      <td>0.723800</td>
    </tr>
    <tr>
      <th>5</th>
      <td>DaveRaven72</td>
      <td>0.548702</td>
    </tr>
    <tr>
      <th>6</th>
      <td>NewWorkshop</td>
      <td>0.756585</td>
    </tr>
    <tr>
      <th>7</th>
      <td>threatintelbot</td>
      <td>0.924094</td>
    </tr>
    <tr>
      <th>8</th>
      <td>slomogoldfish</td>
      <td>0.631055</td>
    </tr>
  </tbody>
</table>
</div>



# Known Human Account Bot Score


```python
humanlist= ['realDonalTrump','BarackObama','SenSchumer','BillGates','elonmusk','marcusdusautoy','justinbieber','selenagomez','taylorswift13']
humanscores=[]
for human in humanlist:
    score = bom.check_account('@'+human)['scores']['english']
    humanscores.append([human,score])

```


```python
humanscores
```




    [['realDonalTrump', 0.5274968237537414],
     ['BarackObama', 0.12588928944948308],
     ['SenSchumer', 0.05780718298413593],
     ['BillGates', 0.03546593341354506],
     ['elonmusk', 0.01983296694956421],
     ['marcusdusautoy', 0.13558257714003935],
     ['justinbieber', 0.04178983547133819],
     ['selenagomez', 0.023437100509770627],
     ['taylorswift13', 0.06263387811083082]]




```python
human_df=pd.DataFrame(humanscores, columns=["username","score"])
human_df
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>username</th>
      <th>score</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>realDonalTrump</td>
      <td>0.527497</td>
    </tr>
    <tr>
      <th>1</th>
      <td>BarackObama</td>
      <td>0.125889</td>
    </tr>
    <tr>
      <th>2</th>
      <td>SenSchumer</td>
      <td>0.057807</td>
    </tr>
    <tr>
      <th>3</th>
      <td>BillGates</td>
      <td>0.035466</td>
    </tr>
    <tr>
      <th>4</th>
      <td>elonmusk</td>
      <td>0.019833</td>
    </tr>
    <tr>
      <th>5</th>
      <td>marcusdusautoy</td>
      <td>0.135583</td>
    </tr>
    <tr>
      <th>6</th>
      <td>justinbieber</td>
      <td>0.041790</td>
    </tr>
    <tr>
      <th>7</th>
      <td>selenagomez</td>
      <td>0.023437</td>
    </tr>
    <tr>
      <th>8</th>
      <td>taylorswift13</td>
      <td>0.062634</td>
    </tr>
  </tbody>
</table>
</div>



# Plotting the Botometer Score of Bots and Humans


```python
fig, ax = plt.subplots(2, figsize=(8,12))
fig.subplots_adjust(hspace=0.7, wspace=0.3)

ax[0].barh(bot_df.username, bot_df.score)
ax[0].set_title("Twitter Bots on the Botometer", fontsize = 24)
ax[0].set_xlabel("Botometer Score", fontsize=18)
ax[0].set_ylabel("Username of Bots", fontsize=18)
ax[0].set_xlim(0,1)
ax[1].barh(human_df.username, human_df.score)
ax[1].set_title("Humans on the Botometer", fontsize = 24)
ax[1].set_xlabel("Botometer Score", fontsize=18)
ax[1].set_xlim(0,1)
ax[1].set_ylabel("Username of Humans", fontsize=18)

```




    Text(0,0.5,'Username of Humans')




![png](output_165_1.png)



```python
result['scores']['english']
```




    0.9757295961173048




```python
df3=pd.DataFrame(result)
```


```python
df3.loc["english","scores"]
```




    0.9757295961173048
