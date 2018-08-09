---
layout: page
title: EDA2
permalink: /eda2

---


EDA Visualizations and Remarks


```python
# Basic tools
import warnings
warnings.filterwarnings('ignore')
import numpy as np
import pandas as pd
import matplotlib
import matplotlib.pyplot as plt
import seaborn as sns
from pandas.core import datetools
from pandas.plotting import scatter_matrix
from sklearn.utils import resample
import seaborn as sns

%matplotlib inline
```


```python
# Load csv file into Dataframe and split into features and labels
def split_X_y(filename):
    # Read csv files in to pandas dataframe
    df=pd.read_csv(filename,index_col=0)

    # return DataFrame object with labels dropped, and a Series object of labels
    return df.drop('bot_or_not',axis=1), df.bot_or_not



# Plot histogram comparison of Bots vs Humans
def hist_bot_vs_hum(predictor, title=None, bins=25, normed=True,
                    xmin=None,xmax=None, ymin=None,ymax=None):
    fig, ax = plt.subplots(figsize=(14,8))
    if title:
        ax.set_title(title, fontsize=24)
    else:
        ax.set_title("Normed Histogram of '"+predictor+"'\nHumans vs. Bots", fontsize=24)

    if (xmin!=None) & (xmax!=None):
        ax.set_xlim(xmin,xmax)

    if (ymin!=None) & (ymax!=None):
        ax.set_ylim(ymin,ymax)

    if normed:
        ax.set_ylabel("Frequency", fontsize=18)
    else:
        ax.set_ylabel("Counts", fontsize = 18)

    ax.hist(hum_X_train[predictor], bins=bins, normed=normed,
            label="Humans", color='r', alpha = 0.2)

    ax.hist(bot_X_train[predictor], bins=bins, normed=normed,
            label="Bots", color='b', alpha = 0.2)

    ax.set_xlabel(predictor, fontsize=18)

    ax.grid(True, '--',alpha=0.2)
    ax.legend(fontsize=18, loc='best')



# Create four scatterplots based on col1 and col2
# Top-Left       : All Humans Data
# Top-Right      : All Bots Data
# Bottom-Left    : All Humans and All Bots
# Bottom-Right   : All Bots and Equal Number of Humans (randomly sampled)
def visual_bot_vs_hum(col1,col2, xmin=None, xmax=None,ymin=None,ymax=None):
    fig,ax=plt.subplots(2,2,figsize=(16,16))
    fax=ax.ravel()

    fig.suptitle("'"+col1+"' vs. '"+col2+"'", fontsize=24)
    prop_sample=resample(hum_X_train, replace=False, n_samples=len(bot_X_train))


    fax[0].scatter(bot_X_train[col1],bot_X_train[col2],
               marker='x', alpha=0.1, color='r', label='Bot')
    fax[0].set_title("Just Bots", fontsize=18)


    fax[1].scatter(hum_X_train[col1],hum_X_train[col2],
               marker='+', alpha=0.1, color='b', label="Human")
    fax[1].set_title("Humans Only", fontsize=18)


    fax[2].scatter(prop_sample[col1],prop_sample[col2],
               marker='+', alpha=0.1, color='b', label="Human")
    fax[2].set_title("Humans Only", fontsize=18)

#     fax[2].scatter(prop_sample[col1],prop_sample[col2],
#                marker='+', alpha=0.1, color='b', label="Human")
#     fax[2].scatter(bot_X_train[col1],bot_X_train[col2],
#                marker='x', alpha=0.1, color='r', label='Bot')
#     fax[2].set_title("Equal Number of Humans and Bots", fontsize=18)



    fax[3].scatter(hum_X_train[col1],hum_X_train[col2],
               marker='+', alpha=0.1, color='b', label="Human")
    fax[3].scatter(bot_X_train[col1],bot_X_train[col2],
               marker='x', alpha=0.1, color='r', label='Bot')
    fax[3].set_title("All Bots and All Humans", fontsize=18)


    for i in range(4):
        fax[i].legend(fontsize=14,loc=(1,1))
        fax[i].grid(True, alpha=0.2, ls='--')
        fax[i].set_xlabel(col1, fontsize=18)
        fax[i].set_ylabel(col2, fontsize=18)
        if (xmin==None) & (xmax==None):
            fax[i].set_xlim(np.percentile(hum_X_train[col1], 5), np.percentile(hum_X_train[col1],95))
        else:
            fax[i].set_xlim(min(0,xmin),xmax)
        if (ymin==None) & (ymax==None):
            fax[i].set_ylim(np.percentile(hum_X_train[col2], 5), np.percentile(hum_X_train[col2],95))        
        else:
            fax[i].set_ylim(min(0,ymin),ymax)




```


```python
X_train, y_train = split_X_y("BASELINE_train.csv")
```


```python
    sns.distplot(hum_X_train[predictor], norm_hist=True, ax=ax)
    sns.distplot(bot_X_train[predictor], norm_hist=True, ax=ax)


```


```python
hum_X_train = X_train.loc[y_train[y_train==0].index]
hum_y_train = y_train.loc[y_train[y_train==0].index]

bot_X_train = X_train.loc[y_train[y_train==1].index]
bot_y_train = y_train.loc[y_train[y_train==1].index]


columns= hum_X_train.columns
len(columns)
```




    18



# Histograms


```python
sns.distplot?
```


```python
hist_bot_vs_hum('user_listed_count',
                bins=25,normed=True, xmin=0, xmax=250)
```


![png](output_8_0.png)


Figure 1. The plot above shows a stark difference between bots and humans in their list joining behavior. Bots seem to either join no lists or join a bunch with few inbetween.

----


```python
hist_bot_vs_hum('char_count',normed=True)
```


![png](output_10_0.png)


Figure 24. Given that tweets have a 140-character limit, bots might be taking better care to ensure that their tweets stay within the allowed character limits. Perhaps a bit better than humans do.

----


```python
hist_bot_vs_hum(columns[1],normed=True)
```


![png](output_12_0.png)



```python
hist_bot_vs_hum(columns[2],normed=True)
```


![png](output_13_0.png)


The plot above 'user_verified' is an indicator variable. This plot interestingly shows that bots never verify their accounts.

----


```python
hist_bot_vs_hum(columns[3],normed=True)
```


![png](output_15_0.png)



```python
hist_bot_vs_hum(columns[4],normed=True)
```


![png](output_16_0.png)



```python
hist_bot_vs_hum(columns[5],normed=True)
```


![png](output_17_0.png)



```python
hist_bot_vs_hum(columns[6],normed=True)
```


![png](output_18_0.png)



```python
hist_bot_vs_hum(columns[7],normed=True)
```


![png](output_19_0.png)



```python
hist_bot_vs_hum(columns[8],normed=True)
```


![png](output_20_0.png)



```python
hist_bot_vs_hum(columns[9],normed=True)
```


![png](output_21_0.png)



```python
hist_bot_vs_hum(columns[10],normed=True)
```


![png](output_22_0.png)



```python
hist_bot_vs_hum(columns[11],normed=True)
```


![png](output_23_0.png)


Figure 25. The distribution of average word length looks to be similar between bots and humans. This suggests that bots are capable of creating sentences that closely resemble human created sentences.

----


```python
hist_bot_vs_hum(columns[12],normed=True)
```


![png](output_25_0.png)


Figure 26. Bots seem to be better at keeping the number of stopwords at a minimum. This may be because bots are mindful of the 140 character space.

----


```python
hist_bot_vs_hum(columns[13],normed=True)
```


![png](output_27_0.png)



```python
hist_bot_vs_hum(columns[14],normed=True)
```


![png](output_28_0.png)



```python
hist_bot_vs_hum(columns[15],normed=True, xmin=-1,xmax=15)
```


![png](output_29_0.png)


Figure 27. Bots show a trend of using at least one upper-case word in their tweets. Humans, on the other hand usually do not use any upper case words in their tweets.

----


```python
hist_bot_vs_hum(columns[16],normed=True)
```


![png](output_31_0.png)


Figure 28. This plot shows how similar the distributions of our sentiment predictors are for humans and bots. Figure 30 also shows a strikingly similar distribution.

----


```python
hist_bot_vs_hum(columns[17],normed=True)
```


![png](output_33_0.png)


Figure 29. That the distributions are so similar may suggest that sentiment polarity and sentiment subjectivity may not be well suited for discerning bots from humans.

----


# Scatter Plots

### Sentiment Polarity vs. Retweet Count


```python
visual_bot_vs_hum('sentiment_polarity','retweet_count', xmin=-1.1, xmax=1.1, ymin=0,ymax=100000)
```


![png](output_37_0.png)


Figure 30. This plot and the plot in Figure 24 were picked to point out that humans have tweets of sentiment polarity equal or close to 1 that have been retweeted more than 60,000 times, which may be considered having gone viral.

Figure 31. Bots, on the other hand show the same region as being empty. This may suggest that bots are unable or perhaps unwilling to generate a tweet of positive polarity, thus resulting in no viral tweets in the same region in the plot.

----

----

Stopwords vs Averge Number of Words


```python
visual_bot_vs_hum('avg_word','stopwords', xmin=-0.1, xmax=20.1, ymin=-0.1,ymax=20)
```


![png](output_41_0.png)


Figure 32. The number of stopwords in relation to average word length seems to be more tightly distributed for bots than for humans. This may be indicative of how machines calculate their tweets.

----


```python
columns[5]
```




    'user_listed_count'



### Retweet Count vs. User Statuses Count


```python
visual_bot_vs_hum(columns[0],columns[7])
```


![png](output_45_0.png)


There seems to be discrete levels to the number of tweets (statuses) bots create. Comparing the left plots (top and bottom), it can be seen that the number of tweets created by bots take on discrete levels (see top left y-axis in red), whereas human created tweets are in numbers that densely populate the entire range. This may be due to coordinated activity by bots controlled by a single group of actors. It might be worthwhile looking into whether each discrete level corresponds to the same group of Twitter users, which, if verified, would strongly indicate that a single user controls them all.

----


```python
visual_bot_vs_hum(columns[2],columns[3])
```


![png](output_47_0.png)


There seems to be discrete levels in bots for the y-axis, whereas humans show a continuous distribution. The implications of the number of followers a user has should be looked into further for coordinated efforts made by Twitter accounts controlled by a single group or user.

----


```python
visual_bot_vs_hum(columns[5],columns[16])
```


![png](output_49_0.png)


Notice the vertical bands shown in the top-left plot. There is an especially distinct vertical band at approximately 70 for user_listed_count. This band seems to be unique to bots as humans do not show much of a vertical pattern. It may be worthwhile looking into.


```python
visual_bot_vs_hum(columns[5],columns[17])
```


![png](output_51_0.png)


Notice the same vertical bands shown here again in the top-left plot. There is an especially distinct vertical band at approximately 70 for user_listed_count. This band seems to be unique to bots as humans do not show much of a vertical pattern.


```python
scatter_matrix(hum_X_train, figsize=(30,30))
```




    array([[<matplotlib.axes._subplots.AxesSubplot object at 0x121d262e8>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x1274ec048>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x1217ee9b0>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x121811470>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x125104c50>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x125104c18>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x11aa8cc18>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x11aac0278>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x1227cc1d0>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x128c059b0>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x122e0bb70>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x124073518>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x12402a5f8>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x11af036d8>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x11af96a90>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x127458748>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x118671da0>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x122369908>],
           [<matplotlib.axes._subplots.AxesSubplot object at 0x1164abe10>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x1241a9d30>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x119f9f208>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x11b8f0dd8>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x11b8f94e0>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x11391c860>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x123c01780>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x12266b550>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x1232aa6d8>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x12504e4e0>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x11dbac710>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x122ca3d30>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x124929208>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x12244b748>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x128b8cf98>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x122ce3518>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x12661f128>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x1250a3828>],
           [<matplotlib.axes._subplots.AxesSubplot object at 0x1240fdf98>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x11af6ceb8>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x12539f470>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x1267528d0>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x127466eb8>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x119f78860>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x12535bd68>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x127dd3080>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x11aa793c8>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x1266e6160>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x126e3bbe0>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x126f8f240>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x121d46860>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x1248534e0>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x1224c7f60>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x1250575c0>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x12182abe0>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x1248a9cc0>],
           [<matplotlib.axes._subplots.AxesSubplot object at 0x1247af390>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x12671b8d0>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x124168ef0>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x12418b518>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x12541e7f0>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x121888e10>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x125353470>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x12547ce80>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x1250ebb70>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x11af690f0>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x1254bb630>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x12419bc50>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x1250ca588>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x11afdc048>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x11d795780>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x125fc6cc0>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x1223c4320>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x11af82470>],
           [<matplotlib.axes._subplots.AxesSubplot object at 0x11b89cef0>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x11bbbf550>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x126779b00>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x12674ed30>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x123838400>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x12239d940>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x118e1ef60>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x122375b38>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x12502d860>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x12414ae80>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x1256ed4e0>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x124049f60>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x126f3cbe0>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x11aade8d0>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x1240707f0>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x124fd3320>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x118653e10>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x1240c2390>],
           [<matplotlib.axes._subplots.AxesSubplot object at 0x1247e39b0>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x11aea0278>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x11aea4908>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x126ecb0b8>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x11dd25e10>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x12511f940>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x11dcbae80>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x12601fa90>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x11863b080>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x1221e16a0>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x126c78be0>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x1221d16a0>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x126e0b470>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x122f15ef0>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x124f13550>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x11dc7fb00>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x11bb5c198>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x117519240>],
           [<matplotlib.axes._subplots.AxesSubplot object at 0x126d18860>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x11d7f3e80>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x11bc28f60>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x122f58630>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x11a99dc50>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x118e662b0>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x118e5e6a0>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x127e7ba90>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x122e800f0>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x127eb7710>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x123fea1d0>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x123ff1e10>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x126d14390>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x1247419b0>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x127dc3550>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x121ddc080>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x1240e55c0>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x11dc6bbe0>],
           [<matplotlib.axes._subplots.AxesSubplot object at 0x121d6b710>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x121d734a8>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x126635a20>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x128c33080>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x128bffb70>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x12569c160>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x1186c2cc0>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x127debba8>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x125fe78d0>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x126018e10>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x127dfc208>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x1274086a0>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x1220cc160>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x11b878780>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x116847da0>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x1238039b0>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x1167adef0>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x11686d550>],
           [<matplotlib.axes._subplots.AxesSubplot object at 0x1168a0fd0>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x1168f3080>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x125317f98>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x12477aeb8>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x1253c8b38>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x11dfee390>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x116916048>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x124e89748>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x124760d68>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x116939358>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x1169685f8>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x11699ac88>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x1173172e8>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x117349908>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x117329e10>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x1173b7128>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x1173ec748>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x11741dcf8>],
           [<matplotlib.axes._subplots.AxesSubplot object at 0x11745f898>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x117492438>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x1174c6a58>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x11816df98>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x1181afb38>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x1181e2668>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x118217ba8>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x118257208>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x11828bcf8>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x118295eb8>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x1182fa048>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x11832e668>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x11836e198>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x1183a4748>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x1183d62e8>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x11840a898>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x11ab065f8>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x11ab5f438>],
           [<matplotlib.axes._subplots.AxesSubplot object at 0x11aaf2438>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x11abc2c88>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x12ab98748>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x11abf8d68>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x11ac363c8>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x11ac67048>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x11ac9bac8>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x11acdb128>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x11ad12748>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x11ad41828>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x11ad75eb8>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x11adb5438>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x11adeca58>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x11adc02e8>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x11ae5a358>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x11df60978>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x11df90f98>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x11e013a58>],
           [<matplotlib.axes._subplots.AxesSubplot object at 0x1218b16d8>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x121aa9cf8>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x121ae9278>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x121b1bd68>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x121b50908>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x121b85e48>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x121bc44a8>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x121bf6f98>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x121c03ba8>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x121deb2e8>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x121e1e908>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x121e5d438>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x121f159e8>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x121f48588>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x121f7bb38>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x121fbb198>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x121ff46d8>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x121f8e080>],
           [<matplotlib.axes._subplots.AxesSubplot object at 0x122056f28>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x1221989e8>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x123495048>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x1234cb668>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x1234f72e8>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x12352fd68>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x12356f3c8>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x1235a4978>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x1235d5ba8>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x123612278>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x1236497b8>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x12367add8>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x123666320>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x1236ed6d8>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x123721cf8>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x123761358>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x123795dd8>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x123882a58>],
           [<matplotlib.axes._subplots.AxesSubplot object at 0x1238c20b8>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x1238f85f8>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x123935198>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x123958c88>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x12399c208>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x1239d0828>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x123a11358>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x123a19fd0>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x123a76668>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x123aabc88>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x123ae9748>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x123b1ecf8>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x123b52908>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x123b85f28>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x123c7c9e8>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x123cadf98>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x123ce41d0>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x123d20828>],
           [<matplotlib.axes._subplots.AxesSubplot object at 0x123d52e48>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x123d94978>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x123d9d828>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x123df9ba8>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x123e38208>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x123e6dcf8>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x123eae2e8>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x123ed0e48>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x123f143c8>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x123f30630>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x1241f9f28>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x123f1d860>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x124265d68>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x1242a63c8>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x1242dbe48>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x12431b4a8>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x12433fc88>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x124382358>],
           [<matplotlib.axes._subplots.AxesSubplot object at 0x1243b7898>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x1243ebeb8>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x124429208>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x12445a898>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x124492eb8>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x1244d1518>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x1244a48d0>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x124534cf8>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x124575358>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x1245ac898>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x1245e93c8>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x12461b048>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x124651668>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x124686c88>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x1246c5748>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x1246f8278>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x12495b7b8>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x12498cdd8>],
           [<matplotlib.axes._subplots.AxesSubplot object at 0x1249cd978>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x1249d6ac8>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x124a33c18>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x124a72278>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x124aa4d68>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x124ae8358>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x124b0aeb8>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x124b4d4a8>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x124b82b38>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x124bc40b8>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x124b56588>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x124c27898>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x124c69358>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x124c9c978>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x124ccef98>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x124d00c18>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x124d416d8>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x124d76cf8>],
           [<matplotlib.axes._subplots.AxesSubplot object at 0x124db6278>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x124de7518>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x124e1aba8>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x124f59208>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x125142828>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x124f6ed30>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x1251b1048>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x1251e7668>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x12521bc88>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x12525a748>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x12528b3c8>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x1254c89e8>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x125509048>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x12553cac8>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x12556e5f8>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x1255a2b38>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x1255e3198>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x125617c18>]],
          dtype=object)




![png](output_53_1.png)
