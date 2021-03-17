# Project 3: Reddit Classification

Brandon Bergeron | DSI-113020 | 01.22.2021

## Problem Statement

The objective of this project was to successfully scrape two subreddits using the [Pushshift](https://github.com/pushshift/api) API, and build a model that could successfully predict which subreddit an individual post came from. I chose the subreddits [r/TalesFromYourServer](https://www.reddit.com/r/TalesFromYourServer/) and [r/TalesFromRetail](https://www.reddit.com/r/TalesFromRetail/)


## Project Directory
```
project-3
|__ code
|   |__ 1_Data_Collection.ipynb
|   |__ 2_Cleaning_EDA.ipynb
|   |__ 3_Modeling.ipynb
|__ data
|   |__ all_retail.csv
|   |__ all_server.csv
|   |__ retail_text.csv
|   |__ server_text.csv
|   |__ clean_posts.csv
|   |__ pre_covid.csv
|   |__ covid.csv
|__ README.md
|__ slide-deck.pdf
```

## Executive Summary

This project consisted of three major phases: Data Collection, Data Cleaning and EDA, and finally Modeling. 

I collected the data by scraping the two subreddits, getting rid of deleted posts, and set out to learn more about the data before cleaning. I did some preprocessing using a custom regex filter wrapped in a WordNetLemmatizer on the posts collected to make them more easily searchable for patterns and possible noise to remove. It slowly became apparent that this may be an easier classification task than originally thought due to industry specific keywords, so I sought to remove as many unfair advantages as I could and set out to build as successful a model as possible. 

I then tried a wide range of models, fine-tuning the parameters on each, choosing 3 final models to use GridSearchCV on for better performance: LogisticRegression, AdaBoostClassifier, and Support-Vector-Classifier. After this initial stage of modeling, I decided to look deeper into the use frequency of the word 'mask' and get a bigger picture look of how the current pandemic could be affecting my model. After splitting the data at March 1st using the UniversalTimeCode attached to each post, I was able to look at model performance on each set, as well as how a model trained in a pre-covid world would perform today.

## Data Collection

The use of the Pushshift Reddit API was fairly straightforward. The only issue I encountered was a limit of 100 posts per pull rather than the 500-post limit listed in the docs, as well as a high volume of deleted posts from r/TalesFromRetail. I ended up splitting my pulls for each subreddit into 2 files- One with only the posts that weren't deleted, and one with each pull in case I decided to go back and search for a pattern in the deleted posts. I didn't ever return to the deleted posts, but a complete csv of each subreddit's pulls is in the repo as 'all_retail.csv', and 'all_server.csv'

## Data Cleaning and EDA

There were two stages to my Data Cleaning and EDA. 

First I looked at the first and last time-code to get an idea of the timeframe the posts covered, searched through the columns to find any useful patterns or columns for cleaning, and dropped duplicates. Most other columns were either empty, or mostly missing so I dropped everything except the time code, author, title, and text in the post.

After ensuring that each post was unique, I did some basic modeling to see what kind of difficulties I would encounter, and was skeptical of the success of these first models. I initialy had near 100% accuracy on my train data, and 90% accuracy on my test data with no parameter tuning with all of my important features having to do with exclusively restaurants/serving. After realizing this, I decided to take a step back and look deeper into the contents of each subreddit to understand what was driving the high performance. 

To do this, I built a custom regex filter wrapped in a stock WordNetLemmatizer inspired by [this](https://towardsdatascience.com/building-an-etl-pipeline-in-python-f96845089635) article on data pipelines. This function created a new column of lemmatized posts for easier inspection/searching. I was able to then fit a CountVectorizer to this column to identify the highest occuring words in each subreddit and identify patterns. The most glaring was the high occurence of "TLDR;" (too-long-dont-read) in the r/TalesFromYourServer subreddit, and high occurence of a special blankspace character "&x200b" which converted to "amp x200b" in r/TalesFromRetail. I decided to remove both of these from all posts to limit the hidden advantage given to my model. Once this was complete I was able to get a great sense for the makeup of these two subreddits and proceed with modeling.


## Modeling

I tried out a wide variety of models, first with just a stock CountVectorizer and StandardScaler as the preprocessing components. Almost all models showed slight overfitting, but good performance. I then bumped down the max features for the CountVectorizer down to 500, and fit all models with both a Cvect and TFIDF vectorizer at that setting to see if one would work better in certain models. Across the board, the TfidfVectorizer slightly outperformed the Count Vectorizer. I then used my prior experience with each model to tune the hyper-parameters and see if I could get any more out of them, before choosing my final three models to GridSearch on. These were AdaBoost, SVC, and a basic LogisticRegression. The Support-Vector-Classifier took hours to fit, so I decided to search a few parameters manually but they did not improve the model. AdaBoost and LogisticRegression both saw marginal increases in performance, but nothing substantial. In the end, turning back to RandomForest and just doubling the amount of estimators to 200 got me to 95.7% accuracy, which was my best performing model. 

## Future Work

After realizing that more model adjustments were producing diminishing returns I decided to look at the effect of the pandemic on both of these subreddits and my models performance by separating the posts before/after March 1st, 2020. There was a large spike in the usage of the word 'mask' in r/TalesFromYourServer, and I was curious to see if that would throw off a model trained on posts that were created before the pandemic. I saw a very slight decline in performance, but not enough to definitively say that the pandemic had an adverse effect on my models performance. I would like to dig a little deeper into the type of errors my model was prone to in these different scenarios. I wasn't particularly concerned with type I or II errors when predicting the subreddits, but I would have maybe been more interested in which subreddit was thrown off by the addition of a new landscape. 

I also experimented with using a Neural Network, but was not successful. I will likely return to this project and implement one once I become more familiar with how to use one with text. 


## Final Conclusions and Summary

I was able to successfully classify my ~20K reddit posts with 95.7% accuracy which I could not improve upon, and after considering possible effects of the pandemic on my model, I was not able to conclude that it had any meaningful effect.  