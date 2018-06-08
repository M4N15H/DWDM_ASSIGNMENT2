# # Assignment 2
### The Following Packages Were Installed For This Assignment
* Tweepy - Twitter API for python.
* elasticsearch - To push the data into Elastic Search DB.
* vaderSentiment -  A lexicon and rule-based sentiment analysis library.
* csvkit - To View CSV Files.
* numpy - Python Library.
* pandas - Python Library.

```sh
$ pip3 install tweepy
$ pip3 install elasticsearch
$ pip3 install vaderSentiment
$ pip3 install csvkit
$ pip3 install numpy
$ pip3 install pandas
```
### Twitter Tweet Extraction:
```sh
$ import tweepy
import time
import json
import csv
import re

#Twitter API credentials
consumer_key = "hCG5oBpZyW7lOgyEFkJE3qvG4"
consumer_secret = "AlgkEIr1fCAXBe7aUfWNPfK7AYhKfeAycN2J6fS8hCwMzcfE8O"
access_key = "1001224378602917893-ZsCZ4l8n7ADhPToJZHd9fhQAWzyq12"
access_secret = "cV8EUO6AAkOu7KNFtpv2Waulx1msXiOdy6FWTDhZdqu89"

#Create connection with OAuth
auth = tweepy.OAuthHandler(consumer_key, consumer_secret)
auth.set_access_token(access_key, access_secret)
api = tweepy.API(auth)

#Extracting data from the twitter account
# In Reference to https://dev.twitter.com/rest/reference/get/users/show describes get_user
def get_profile(screen_name):
    tweepyapi = tweepy.API(auth)
    try:
        user_profile = api.get_user(screen_name)
    except tweepy.error.TweepError as e:
        user_profile = json.loads(e.response.text)

    return user_profile


def get_tweets(query):
    api = tweepy.API(auth)
    try:
        trendintweets = api.search(query)
    except tweepy.error.TweepError as e:
            trendintweets = [json.loads(e.reposnse.text)]
    return trendintweets

#saving csv
queries = ["#syria -filter:retweets lang:en", "#food -filter:retweets lang:en","#red -filter:retweets lang:en""@Windows","#art -filter:retweets lang:en","#india -filter:retweets lang:en","#happy -filter:retweets lang:en","#positivevibes -filer:retweets lang:en","#happybirthday -filter:retweets lang:en"]
with open ('data.csv','w') as outfile:
            writer = csv.writer(outfile)
            writer.writerow(['id', 'user', 'created_at', 'text'])
            for query in queries:
                    t = get_tweets(query)
                    for tweet in t:
                        tweet.text=' '.join(re.sub("(@[A-Za-z0-9]+)|([^0-9A-Za-z \t])|(\w+:\/\/\S+)"," ",tweet.text).split())# Regex taken from https://stackoverflow.com/questions/8376691/how-to-remove-hashtag-user-link-of-a-tweet-using-regular-expression
                        writer.writerow([tweet.id_str,tweet.user.screen_name,tweet.created_at,tweet.text])
```
### Sentiment Analysis:
```sh
$ import csv
from vaderSentiment.vaderSentiment import SentimentIntensityAnalyzer
import numpy as np
import pandas as pd
df = pd.read_csv("data.csv")
saved_column = df.text
#print(saved_column)

# In Reference to https://github.com/cjhutto/vaderSentiment

f= open('data.csv')
csv_f= csv.reader(f)

analyzer = SentimentIntensityAnalyzer()
df = pd.DataFrame(columns=['tweet','sentiment','sentiment_score'])

info = []
info_a = []
text_tweet = []
for idx,row in enumerate(csv_f):
    vs = analyzer.polarity_scores(row[3])
   # print("{:-<65} {}".format(row[3], str(vs)))
    pos = vs['pos']
    neg = vs['neg']
    neu = vs['neu']
    arr = np.array([pos,neg,neu])
    idx = np.argmax(arr)
    if idx==0:
       info_a.append('positive')
    elif idx==1:
       info_a.append('negative')
    elif idx==2:
       info_a.append('neutral')
    info.append(np.max(arr))
    text_tweet.append(row[3])

df['tweet'] = text_tweet
df['sentiment'] = info_a
df['sentiment_score'] = info
df.to_csv('sentiment.csv',index=False)
```
### Loading Data Into ElasticSearch DB:
```sh
$ from datetime import datetime
from elasticsearch import Elasticsearch
from elasticsearch import helpers
from elasticsearch.helpers import bulk
import csv

# In Reference to http://elasticsearch-py.readthedocs.io/en/master/helpers.html

es = Elasticsearch('http://104.211.58.86:9200/')

with open('sentiment.csv') as f:
       reader = csv.DictReader(f)
       helpers.bulk(es, reader, index='data', doc_type='elasticdata')
```
### ETL As a Batch Process (ShellScript) :
```sh
$ echo "Analyzing...."

python3 tweetscap.py
python3 vadersenti.py
python3 elasticdata.py

echo "FINISHED"
```
----



