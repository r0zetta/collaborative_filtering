# collaborative_filtering
This repository contains a report (this README) and accompanying code and data that detail a set of experiments designed to simulate theoretical poisoning attacks against social network recomendation mechanisms.



# Introduction
It is highly likely that social networks utilize collaborative filtering mechanisms to recommend actions and content to users. A variety of recommendation mechanisms exist on these platforms, each tailored for their own specific purposes. Examples may include recommendations for other users to follow (Twitter), or groups to join (Facebook), construction of curated timelines or search results (Twitter), and recommendations for content to view next (YouTube).

Collaborative filtering techniques consume user preference data in order to build models that can be used to recommend items to users in a system. Recommendations are typically based on one of two criteria:
- user-based recommendations recommend items that were most popular with similar users
- item-based recommendations recommend items that are most similar to other items the user has interacted with

While we don't know exactly how public social network recommendation mechanisms are implemented, we can 



## AIM
Simulate social network recommendation logic and attempt to manipulate it using poisoning attacks.

## GOAL
Use a Sybil attack to boost an account such that it is recommended to users it otherwise wouldn’t have been recommended to.



# Experimental detail






The attacks detailed in this report are designed to boost a specifically chosen target user such that it is recommended to users who interacted with a separate, specific high-profile user. Each attack is performed by instructing a number of accounts to retweet both the target account (to be boosted) and the separate high-profile user. This experiment demonstrates the effectiveness of this attack using different numbers of attacker accounts and retweet counts.

## Experimental process

This experimental procedure follows these steps:
1. Load a dataset of anonymized retweet interactions collected from actual Twitter data.
2. Train a collaborative filtering model on the loaded data.
3. Select a target account to be "amplified" such that it is recommended to a set of users who have interacted with a separate, high-profile user also in the dataset. We select 20 such users as a "control" set.
4. Implement simple recommendation logic based on cosine similarity of the vector representations of the trained model, and observe recommendations for the control set.
5. Select a set of "amplifiers" that have not interacted with either the target account or the high-profile account.
6. For various numbers of amplifier accounts and retweet counts, create a new dataset containing additional interactions between each selected amplifier account and both the target account and the high-profile account. In practise, this process involved appending two new rows per amplifier account - one adding a retweet count for the target account and another adding retweet count for the high-profile account.
7. Train a new model on the poisoned dataset.
8. Run both target-based and source-based recommendations for each member of the control group and record the number of times the target appeared in the top-n (3) recommendations.
9. Present results as a graph.

Each step is clearly labeled in the notebook.

## Libraries used

- Numpy (https://numpy.org/)
- Pandas (https://pandas.pydata.org/)
- NetworkX (https://networkx.org/)
- Louvian community detection (https://github.com/taynaud/python-louvain)
- Collaborative filtering model: fastai collab_learner (https://docs.fast.ai/collab)
- Cosine similarity matrix: sklearn cosine_similarity (https://scikit-learn.org/stable/modules/generated/sklearn.metrics.pairwise.cosine_similarity.html)
- Visualizsation: matplotlib (https://matplotlib.org/) and seaborn (https://seaborn.pydata.org/)


## Recommendation algorithm implementations

The fastai collaborative filtering model consists of a set of learned weights for both sources and targets. A cosine similarity matrix can be built from each of these sets of vectors, thus allowing one to query the similarity between any given pair of sources or targets. Using these cosine similarity matrices, one can implement the following simple recommendation algorithms that can be used to reccommend accounts to a user based on who the user has retweeted.

1. Target-based similarity

Using the target-based cosine similarity matrix, one can generate recommendations for source based on who they've retweeted. For a given source, obtain a list of all target accounts retweeted by the source, and the number of times the source retweeted the target account. For each of the target accounts in the list, obtain a list of _t_max_matches_ most similar target accounts from the target-based cosine similarity matrix. For each of these, multiply the similarity value (between the source account and the target account) with the number of times the source retweeted the target account, and add that value to a running total score for each target retweeted account.

In pseudocode:
```
for target, num_retweets in get_source_retweets(source):
    for similar, similarity in get_most_similar(target):
        recommended[similar] += num_retweets * similarity
```

2. Source-based similarity

Using the source-based cosine similarity matrix, one can generate a ranked list of recommendations for a given source account as follows. First obtain a list of _s_max_matches_ sources most similar to the target account. For each of these source accounts, obtain a list of target accounts they retweeted, and the number of times they retweeted. For each target-retweeted_count, multiply retweeted_count by the similarity value between the initially queried source and this source account. Add that value to a running total score for each target retweeted account.

In pseudocode:
```
for similar_source, similarity in get_most_similar(source):
    for target, num_retweets in get_source_retweets(similar_source):
        recommended[target] += similarity * num_retweets
```

Both of the above mechanisms will generate a ranked list of target accounts to recommend to the source - a list of targets and score values where higher scores are more highly recommended. By comparing this ranked list to a list of accounts the source has already interacted with, a list of recommendations of targets the user hasn't yet interacted with can be generated.

Note: we can measure the effectiveness of both of these simple methods by comparing the ranked list of recommended accounts against accounts the user has already retweeted. The closer the lists match, the more accurate the recommendations are. Based on experimental results it is clear that the source-based recommendation approach is much more accurate.

The algorithms demonstrated in this notebook were implemented by me and are intentionally simple. Social network recommendation methodology is likely based on similar principles as demonstrated here (i.e. collaborative filtering, plus some additional logic), but may utilize other available information such as which accounts the user is following or being followed by, metrics related to content a user viewed and how long they viewed that content for, hashtags the user has included in their posts, accounts the user has tagged in their posts, accounts the user has replied to, likes (which are not available via the Twitter API), the user's geographical region, the user's language settings, and so on.

Note that it would be extremely difficult to determine how close the algorithms implemented in this experiment match the underlying mechanisms in real social networks such as Twitter. The data we're working with is fixed - it's a snapshot collected over a short period of time. The data also only includes interactions between a limited number of accounts (those that interacted with the small number of users queried during the data collection process). One may view this experiment as a series of "what-if" experiments, which demonstrate how recommendations in the system would have changed if retweet activity had been different to the originally recorded behaviour.


## About the dataset

The Twitter dataset was collected using Twitter's streaming API (https://developer.twitter.com/en/docs/twitter-api/v1/tweets/filter-realtime/api-reference/post-statuses-filter). The filter API was instructed to capture Tweet objects from a small number (approx. 100) of accounts participating in US political discussion during October 2020. Twitter's filter API returns Tweet objects when any of the queried accounts publish a tweet, and when tweets mentioning (replies, retweets, mentions) any of the queried accounts are published. Once the data had been collected, retweet interactions between accounts were processed out of the raw Tweet objects in the form account_retweeting - account_being_retweeted - number_of_retweets_observed. Account names were then converted into a set of integers in order to anonymize the data. This data was then written to disk as a csv in the form Source,Target,Weight - allowing it to be directly imported into gephi (https://gephi.org) for graph visualization purposes. The retweet data can be found in this repository (twitter_ratings/ratings.csv).

The dataset contains 52920 rows representing interactions between 25137 retweeters and 8405 retweeted across a total of 95893 retweet interactions. Some additional statistics about the dataset are generated in the code below.
