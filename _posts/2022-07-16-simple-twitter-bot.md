---
layout: post
title: "Simple Twitter Bot"
date: 2022-07-30 22:24:00 -0700
categories: jekyll update
---

### Introduction

In my [last post](https://schnaible.github.io/jekyll/update/2022/07/12/learning-web-scraping.html) I created a simple web-scraper using BeautifulSoup4 and requests in Python. I mentioned that I was trying to get elevated access for my twitter account and luckily I was able to obtain that after several emails. With elevated access we are granted the ability to create tweets.

For this project I used the web-scraper to pull random quotes and [Tweepy](https://docs.tweepy.org/en/stable/) to create tweets using Twitter API. Using these we will automate the process of gathering a random quote and using it to produce a tweet every couple hours.

**Note:** If you're following along you will most likely have to change your Twitter project's user authentication settings. I found this [stakoverflow post](https://stackoverflow.com/a/70958807/19328917) explaining how to change these settings. Be sure to re-generate and save your API Tokens after updating permissions.

### Setup

To begin we must import the Tweepy library and set up our authentication tokens. The Access Token and Secret are the keys that we just regenerated after changing our app permissions.

    import tweepy
    import datetime
    import random
    import requests

    CONSUMER_KEY = 'API Key'
    CONSUMER_SECRET = 'API Key Secret'
    ACCESS_KEY = 'Access Token'
    ACCESS_SECRET = 'Access Token Secret'
    BEARER_TOKEN = 'Your Bearer Token'

After we initialize our keys we can setup our API connection.

    auth = tweepy.OAuthHandler(CONSUMER_KEY,CONSUMER_SECRET)
    auth.set_access_token(ACCESS_KEY,ACCESS_SECRET)
    api = tweepy.API(auth)

#### Testing Access

Once you've replaced the strings with your personal tokens we can verify that our connection to the Twitter API Using the codeblock below. This simply makes a request to the API to verify the credentials. If you see "Authentication OK" Then you can move on to the next step. Otherwise double check that your tokens and keys match the correct fields.

    try:
        api.verify_credentials()
        print("Authentication OK")
    except:
        print("Error during authentication")

### Producing Tweets

#### Craeting a Single Tweet

We can create a single tweet using the API method 'update_status()'. This method takes a string as a parameter, which will end up being the text of the tweet itself. The following line:

    api.update_status("Hello Twitter")

Will produce the tweet shown here:

![A Single Tweet](/assets/images/singleTweet.png "Creating Tweet")

Since we want to be able to create tweets with more than just one message we adapt this line to take a string variable, then throw it in a function that takes that string as a parameter.

    def makeTweet(message):
        api.update_status(message)

#### Pulling a Quote

As mentioned in the Introduction of this post, the goal is to use the web-scraper to get a random quote and use it to produce a tweet automatically. The first thing we will do is put the web-scraper into its own function, with some small changes.

    def pullQuote():
        quoteLength = 999
        tweetText = ''

        while (quoteLength > 280):
            # The page we want to scrape. Tests connection
            pageNum = random.randint(1, 10)
            page = requests.get("https://quotes.toscrape.com/page/" + str(pageNum))

            # Get the HTML content and find all instances where class is quote
            soup = BeautifulSoup(page.content, 'html.parser')
            quotes = soup.find_all(class_="quote")

            # Each page has 10 quotes, generate a random value in this range and pull the quote
            randInt = random.randint(0, 9)

            quoteText = quotes[randInt].find_all(class_="text")[0].text     # Get the Quote from the inner class
            quoteAuthor = quotes[randInt].find_all(class_="author")[0].text # Get the Author from the inner class

            tweetText = (quoteText + '\n' + quoteAuthor)

            quoteLength = len(tweetText)

        return(tweetText)

The most notable change from my original post where we created the web-scraper is the inclusion of checking the length of our tweet. The maximum length of a tweet is 280 characters so we have to make sure our message, the quote itself along with the author, is not more than 280 characters. Setting a variable, `quoteLength` to a temp value of 999 will let us loop our web-scraping code at least once to get a quote.

To check the length of our message we format our tweet and set it to the variable `tweetText`. With this variable we can set `quoteLength` with `quoteLength = len(tweetText)`. After this while block loops once it will check the new `quoteLength` to ensure that the pulled quote and author is not over 280 characters.

#### Automating Tweets

We now have everything we need to pull a random quote and produce a tweet. Now we need to automate the process. Start off by creating a new function `def autoTweet()`. This function will be an endless loop that pulls a quote with `def pullQuote` and sends it with `makeTweet`. Using `time.sleep()` we can wait a given amount of time before producing another tweet.

    def autoTweet():
        while True:
            message = pullQuote()
            makeTweet(message)
            time.sleep (3600)        # wait 1 hours before creating another tweet

Finally, throwing that into a main and running it:

    def main():
        autoTweet()

    main()

Congrats! You should now have one tweet created and more on their way.

Below is a complete program to tweet a random quote periodically. Also available on my [GitHub](https://github.com/schnaible/TwitterBot).

    import time
    import datetime
    import random
    import requests

    import tweepy
    from bs4 import BeautifulSoup


    CONSUMER_KEY = 'API Key'
    CONSUMER_SECRET = 'API Key Secret'
    ACCESS_KEY = 'Access Token'
    ACCESS_SECRET = 'Access Token Secret'
    BEARER_TOKEN = 'Your Bearer Token'

    auth = tweepy.OAuthHandler(CONSUMER_KEY,CONSUMER_SECRET)
    auth.set_access_token(ACCESS_KEY,ACCESS_SECRET)
    api = tweepy.API(auth)

    try:
        api.verify_credentials()
        print("Authentication OK")
    except:
        print("Error during authentication")

    def pullQuote():
        quoteLength = 999
        tweetText = ''

        while (quoteLength > 280):
            # The page we want to scrape. Tests connection
            pageNum = random.randint(1, 10)
            page = requests.get("https://quotes.toscrape.com/page/" + str(pageNum))

            # Get the HTML content and find all instances where class is quote
            soup = BeautifulSoup(page.content, 'html.parser')
            quotes = soup.find_all(class_="quote")

            # Each page has 10 quotes, generate a random value in this range and pull the quote
            randInt = random.randint(0, 9)

            quoteText = quotes[randInt].find_all(class_="text")[0].text     # Get the Quote from the inner class
            quoteAuthor = quotes[randInt].find_all(class_="author")[0].text # Get the Author from the inner class

            tweetText = (quoteText + '\n' + quoteAuthor)

            quoteLength = len(tweetText)

        return(tweetText)

    def autoTweet():
        while True:
            message = pullQuote()
            makeTweet(message)
            print('Tweet Created: ' + str(datetime.datetime.now())) # Print the time tweet was created
            time.sleep (3600)        # wait 1 hours before creating another tweet


    def main():
        autoTweet()

    main()
