# Elon2DogePUBLIC
Contains basic information about the Elon2Doge project. If you would like access to the code base, DM me on twitter @Elon2Doge.

## Overview
With my current job I am not allowed to automatically buy/sell stocks, however I was interested in looking at trading algorithms. This brought me to crypto. At first, I was looking at trading based off market data (volume, price etc..) but I deemed it was impossible to compete with hedge funds who have access to millisecond-by-millisecond data, so I started looking for my own market indicators. I settled on Dogecoin because, since it is such a ridiculous "Meme Coin", many hedge funds and crypto whales stay clear of it, meaning it is free from the market manipulation that normally occurs. After quick tests on the price of DOGE minutes after an Elon Musk tweet, I decided to build this system.

## System Process
This system is constantly watching @ElonMusk for any new tweets. When a new tweet occurs, a new thread is created to handle it so the main thread can continue polling his twitter. This thread will recursively traverse the entire twitter thread collecting the tweet messages and URLs to any images attached. It then checks to see if any form of DOGE (DOOOGE, DDDOGE, DOGEEEEE all work) occurs in the messages. If it finds a tweet containing DOGE, it will check the input parameter USE_SENTIMENT_ANALYSIS to see if it should preform sentiment analysis. If the sentiment is positive it buys, if negative it skips. 

If the word doge does not appear in the messages, it will scrape all text from each image URL (see explanation below). If an image contains the DOGE keyword, it buys.

Once a buy is triggered, a market order is sent to Binance. The system then sleeps for 15 minutes before selling all the doge purchased.

After selling the Doge, the system creates a graph with the minute-by-minute data which it tweets out along with the percent change. The system will also update the twitter bio with the new running total since account creation.

## Twitter
The python Tweepy library was used to access Elon's account. When receiving live information from api's, the common (and arguably correct) approach is to set up a listener to stream the api. When doing this, I noticed that tweets were received consistently within 5-6 seconds. Since every second matters in this use case, I opted for a different approach. Twitter allows 2,000,000 api calls every month, which allowed me to poll Elon's account every 3 seconds (2,678,400 api calls/month) with room left over for calls to be used for testing. This reduced the time it takes to receive a tweet to be between .5 and 3.5 seconds.

## Binance
Binance is used as the trading platform for cryptos. Since the volume and price of DOGE is ridiculously volatile at the time of tweet, I opted to use a market order for purchase out of fear of missing out. For the sale however, my system creates a limit order at the 14-minute mark, then if this limit is not filled by the 15-minute mark the system will just use another market order as an exit. Binance was used because of their low maker/taker fees of .1%. In the future, I will add a system which ensures .3% of my wallet is in BNB to be used to pay the maker/taker fees. This will reduce the fees to .075% maker/taker.

## Scraping Text from Images
While the most common reason for triggering a purchase is when the thread contains the keyword DOGE, I also added in a function to scrape all image URLs from the thread. I then use Google's Tesseract library to scrape any text that appears in the image. This is extremely useful when people post pictures of tweets or news articles as it easily reads black text on a white background.
For example. When passed this meme,

https://pbs.twimg.com/media/E7HWhnsXoAAkvaL?format=jpg&name=small

the system scraped: 
"What are you trying to tell me, that I can make a lot of money with Dogecoin?"
"No, Neo. I'm trying to tell you that Dogecoin is money."

## Sentiment Analysis
This system has the option to use Sentiment analysis on tweets before purchasing. To do this, I used the Vader library from nltk. This is a Naive Baynes spin off which mostly treats sentences with a bag of words approach but can recognize simple phrases. For example, "not cool" would be deemed negative, even though cool has a positive connotation. It also adds more weight to sentences ending with “!” among other things. I do not have Elon2Doge using the sentiment analysis since I have yet to see Elon tweet anything negative about Doge and do not want to miss out. My other account CZ2Doge is using sentiment analysis to determine if it should purchase because he occasionally tweets about outages or fees around Doge which lower the price.

## Image Recognition
While this portion is not involved in my production model, I thought it was kind of fun :). I created a Convoluted Neural Network which can spot if an image contains either the dogecoin logo or the doge meme dog with 90% percision. Even though this is decent performance, it is not added into production because it produces some false positives. Things like the glow from rocket exhaust and a glowing golden sun are commonly confused for the doge symbol. These false positives make it not worth adding to production because the maker-taker fees add up.

## Graph Creation
Matplotlib is used to create the graph that is tweeted. The Binance API only allows minute by minute historical data, so the graph is not as precise as I would like, as it only has 15 data points. Also, since a market order is sometimes used to sell, I will occasionally sell for less than the current stock price so the real sell price can be lower than what is displayed on the graph.


