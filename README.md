# Simulating poisoning attacks against collaborative filtering-based recommendation mechanisms
This repository details experiments designed to simulate attacks against social network recomendation mechanisms, and evaluate their effectiveness.

# Introduction
Social networks employ a variety of mechanisms designed to recommend content and actions to users. Examples of these mechanisms include suggesting accounts to follow (Twitter), suggesting groups to join (Facebook), constructing curated timelines and search results (Twitter), and recommending videos to watch (YouTube). It is highly likely that some or many of these mechanisms utilize collaborative filtering techniques. Collaborative filtering is a machine learning technique that calculates similarities between users and items based on user preference data. Similarity vectors obtained from a trained collaborative filtering model can then be used to output recommendations for any user in the system. Recommendations are typically based on one of two criteria:
- user-based recommendations recommend items that were popular with similar users
- item-based recommendations recommend items that are similar to other items the user has interacted with

Recommendations can also be calculated from a combination of both user-based and item-based techniques.

Knowledge of how these recommendation mechanisms work can be used to craft attacks against these systems. A number of motives exist for such attacks:
- promotion attacks: 
  - cause a piece of content, item, or user to be ranked similar to another piece of content, item, or user in the system.
  - cause a piece of content, item, or user to appear at a higher position in a user's timeline or in search results.
- demotion attacks:
  - cause a piece of content, item, or user to be ranked less similar to another piece of content, item, or user in the system.
  - cause a piece of content, item, or user to appear at a lower position (or not at all) in a user's timeline or in search results.
- social engineering: if an adversary already has knowledge on how a specific user has interacted with items in the system, an attack can be crafted to target that user with a recommendation.

The most widely used attacks against recommender mechanisms are Sybil attacks – an adversary creates several fake users or accounts, and has them engage with items in patterns designed to change how that item is recommended to other users. Here, the term ‘engage’ is dependent on the system being attacked, and could include rating an item, reviewing a product, browsing a number of items, following a user, adding items to a shopping basket or wishlist, or liking a post. Attackers may probe a system using ‘throw-away’ accounts in order to understand how the underlying recommendation mechanisms work, and to test whether any detection capabilities exist in the system. Skilled attackers carefully automate their fake users to behave like normal users in order to avoid Sybil attack detection techniques. Such approaches are facilitated by a plethora of inexpensive services available on the Internet. These services allow attackers to purchase views, likes, retweets, followers, reviews, and ratings on all of the big-name social networks, crowdsourced review sites, app stores, and ecommerce sites on the Internet. For more analysis on this phenomenon, take a look at this article https://blog.f-secure.com/how-ai-is-already-being-poisoned-against-you/.

A common attack approach used on social networks involves a large number of users collectively agreeing to perform a specific action in order to achieve a goal (promote a piece of content, promote a user, cause a phrase, keyword, or hashtag to trend, etc.) Coordinated attacks are extremely common on Twitter. Here's how they often work. Users form large groups using "followback" mechanisms:
- many accounts post tweets that contain lists of other accounts to follow
- those tweets are retweeted by hundreds or thousands of other participating accounts
- when a user follows one of the mentioned accounts, that account will follow them back

![followback_train](images/followback_coordination.png)

The mechanism results in large groups of accounts that follow each other. It is not uncommon to find accounts on Twitter that follow and are followed by tens of thousands of other accounts (with a high overlap of following-followed). These followback rings, or retweet rooms as they are also called, then collaborate to amplify content, keywords, phrases, or hashtags.

![retweet coordination](images/retweet_collaboration.png)
![let's make this trend](images/lets_make_this_trend.png)

At the time of writing, coordinated amplification mechanisms identical to those detailed above were being used to spread dozens of intentionally misleading voter fraud stories associated with the 2020 US presidential elections.

Twitter coordination is also used to boost brand new Twitter accounts. This tactic is often used when a member of a collective has their account suspended or banned, and needs to quickly "level up" a newly created account. Anecdotal evidence of this process is documented here https://blog.f-secure.com/discovering-hidden-twitter-amplification/.

Due to the prevalence of coordinated Sybil attacks on social networking sites such as Twitter, a study of how these attacks shape underlying collaborative filtering models and recommendation mechanisms is of interest. The research documented here focuses on the study of those attacks. The experiments documented in this report attempt to manipulate recommendation algorithms by simulating Sybil attacks designed to boost an account such that it is recommended to users it otherwise wouldn’t have been recommended to. Experimental details follow.

# Experimental details

All experiments documented here follow the procedure outlined below:
1. Load a dataset of anonymized retweet interactions collected from actual Twitter data.
2. Train a collaborative filtering model on the loaded data.
3. Select a target account to be "amplified" such that it is recommended to a set of users who have interacted with a separate, high-profile user also in the dataset. We select 20 such users as a "control" set.
4. Implement recommendation logic based on cosine similarity of the vector representations of the trained model, and observe recommendations for the control set.
5. Select a set of "amplifier accounts" that have not interacted with either the target account or the high-profile account, and are not members of the control set.
6. For a number of different proposed sets of amplifier accounts and parameter choices, create a new dataset containing additional interactions between each selected amplifier account and both the target account and the high-profile account. In practise, this process involves appending two new rows per amplifier account - one adding a retweet count for the target account and another adding retweet count for the high-profile account.
7. Train a new model on the modified dataset.
8. Run both target-based and source-based recommendations for each member of the control group and record the number of times the target appeared in the top-n (3) recommendations.
9. Present and discuss the results.

In all cases, experiments were run for 10 iterations in order to obtain an average value and corresponding error margins.

For each dataset, the following experiments were run:
1. Amplifier accounts were selected at random from the entire dataset. Different numbers of amplifiers and retweet counts were tried. This experiment was run to determine lower bounds for effective algorithmic manipulation, and to study of the effects of large-scale manipulation at higher values.
2. Amplifiers accounts were chosen from communities derived by applying the Louvain community detection algorithm across a node-edge graph of the original dataset. For each community with at least 200 members, the experiment was run with a fixed 200 amplifiers, each performing 20 retweets of both target account and high-profile account. This experiment was run to determine whether community membership contributes to the effectiveness of algorithmic manipulation.
3. A set of amplifiers with high cosine similarity to the members of the control set were selected. As in experiment 1, varying numbers of amplifiers and retweets were tested. This experiment was designed to determine whether similarity to the control set contributes to the effectiveness of algorithmic manipulation.

If you're interested in the details of these implementations, take a look at the accompanying jupyter notebooks (or even try running them yourself).

## About the datasets
This repository contains two datasets that represent anonymized retweet interactions between Twitter accounts. All data was captured directly from Twitter. The reason why retweet interactions were chosen for this experiment is because those interactions can be extracted directly from Tweet objects obtained via the API. At this moment it is not possible to obtain Like interactions using that API.

The two datasets are described in more detail below.

### US2020 - US2020/anonymized_interactions.csv
This dataset was collected using the filter function of Twitter's streaming API (https://developer.twitter.com/en/docs/twitter-api/v1/tweets/filter-realtime/api-reference/post-statuses-filter). The filter API was instructed to capture Tweet objects from approximately 100 accounts participating in US political discussion during mid-October 2020. Twitter's filter API returns Tweet objects when any of the queried accounts publish a tweet, and when tweets mentioning (replies, retweets, mentions) any of the queried accounts are published.

This dataset contains 52920 rows representing interactions between 25137 retweeters and 8405 retweeted across a total of 95893 retweet interactions. Some additional statistics about the dataset can be viewed by running the accompanying notebook.

- **user_025303** was chosen as the high-profile account. This verified Twitter account belongs to a US politician and receives a great deal of engagement on the platform.
- **user_004286** was chosen as the target account. This non-verified Twitter account actively participates in US political conversation, and often shares disinformation.

The following image is a graph visualization of this dataset.
![US2020 base graphviz](images/fastai_US2020_baseline_unmarked.png)


### UK2019 - UK2019/anonymized_interactions.csv
This dataset was collected using the filter function of Twitter's streaming API (https://developer.twitter.com/en/docs/twitter-api/v1/tweets/filter-realtime/api-reference/post-statuses-filter). The filter API was instructed to capture Tweet objects matching a small number of hashtags related to the December 12th 2019 UK general election. Hashtags included #GE2019, #GE19, #generalelection2019 and #generalelection19. The raw retweet interactions recorded by this process contained approximately 100,000 unique source nodes and 50,000 target nodes. This node-edge graph was too unwieldy to run experiments with, so it was trimmed as follows:
- targets that had received less than 5 retweets were dropped
- all but a random sample of 10,000 accounts that had exactly one outgoing retweet edge were dropped

The resulting dataset contains 112963 rows representing interactions between 35007 retweeters and 3057 retweeted across a total of 123890 retweet interactions. Some additional statistics about the dataset are generated in the accompanying notebook.

- **user_035060** was chosen as the high-profile account. This verified Twitter account belongs to a popular activist that receives engagement from left-wing users on the platform.
- **user_035067** was chosen as the target account. This non-verified Twitter account is a noteable proponent of Brexit, an avid supporter of the UK Conservative party, and often shares biased content and disinformation.

The following image is a graph visualization of this dataset.
![UK2019 base graphviz](images/fastai_UK2019_base_unmarked.png)

Retweet interactions between accounts were processed from collected raw Tweet objects in the form:

**account_retweeting** - **account_being_retweeted** - **number_of_retweets_observed**. 

Account names were anonymized by replacing the Twitter user's screen_name with an anonymized name in the form **user_XXXXXX**. This data was then written to disk as a csv in the form Source,Target,Weight - allowing it to be read into Pandas dataframes and directly imported into gephi (https://gephi.org) for graph visualization purposes.

## Recommendation algorithm implementations
The fastai collaborative filtering model consists of a set of learned weight vectors for both sources and targets. A cosine similarity matrix can be built from each of these sets of vectors, thus allowing one to query the similarity between any given pair of sources or targets. Using these cosine similarity matrices, user-based and item-based recommendation logic can be implemented.

### Target-based (item-based) similarity
Using the target-based cosine similarity matrix, recommendations can be calculated for a source based on who they've retweeted. For a given source, obtain a list of all target accounts they've retweeted and the number of times the source retweeted that account. For each of the target accounts identified, obtain a list of _t_max_matches_ most similar accounts from the (previously calculated) target-based cosine similarity matrix. For each of these, multiply the similarity value (between the source account and the target account) with the number of times the source retweeted the target account, and add that value to a running total score for each target retweeted account.

In pseudocode:
```
for target, num_retweets in get_source_retweets(source):
    for similar, similarity in get_most_similar(target):
        recommended[similar] += num_retweets * similarity
```

### Source-based (user-based) similarity
Using the source-based cosine similarity matrix, a ranked list of recommendations can be generated for a given source account as follows. First obtain a list of _s_max_matches_ accounts most similar to the source account. For each of these accounts, obtain a list of target accounts they retweeted, and the number of times they retweeted. For each target-retweeted_count pair, multiply retweeted_count by the similarity value between the source and this account. Add that value to a running total score for each target retweeted account.

In pseudocode:
```
for similar_source, similarity in get_most_similar(source):
    for target, num_retweets in get_source_retweets(similar_source):
        recommended[target] += similarity * num_retweets
```

Both of the above mechanisms will generate a ranked list of target accounts to recommend to a given source - a list of targets and score values where higher scores are more highly recommended. By comparing this ranked list to a list of accounts the source has already interacted with, a list of recommendations of targets the user hasn't yet interacted with can be generated.

The effectiveness of these methods can be determined by comparing the ranked list of recommended accounts against accounts the user has already retweeted. The more matches, the more accurate the recommendations are.

Here is a sample output for the target-based recommendation algorithm. Only two of the calculated recommendations matched accounts the source had already interacted with.

![target-based recommendations](images/US2020_target_recommendations.png)

Here is a sample output for the source-based recommendation algorithm, based on the same user shown in the previous example. Nine of the calculated recommendations matched accounts the source had already interacted with.

![source-based recommendations](images/US2020_source_recommendations.png)

Based on experimental results it is apparent that source-based recommendations were much more accurate.

The recommendation algorithms implemented in these experiments are intentionally simple. Social network recommendation mechanisms are likely based on similar principles (i.e. collaborative filtering, plus some additional logic), but may utilize other available information such as:
- accounts the user is following or being followed by
- metrics related to content a user viewed and how long they viewed that content for
- hashtags the user has included in their posts
- accounts the user has tagged in their posts
- accounts the user has replied to
- likes (which are not currently available via the Twitter API)
- the user's geographical region
- the user's language settings
- and so on.

It would be difficult to determine how close the algorithms implemented in this experiment match the underlying mechanisms in real social networks such as Twitter. The data we're working with is fixed - a snapshot collected over a short period of time. The data also only includes interactions between a limited number of accounts. One may view this experiment as a series of "what-if" experiments, which demonstrate how recommendations in the system would have changed if retweet activity differed from originally recorded behaviour.


# Results and discussion

## Experiment 1: Randomly chosen amplifiers

The following bar charts depict the effect of varying numbers of amplifier accounts and retweet counts on both the US2020 and UK2019 datasets. Each bar represents the mean percentage of control accounts that observed the target account in the top-3 source-based recomendations over 10 runs. Error bars indicate the minimum and maximum values across those ten runs.

These experiments had no effect on the target-based recommendation algorithm. All bar charts associated with those experiments are empty, and thus are not shown. This is the case across all of the conducted experiments.

XXX discussion as to why the target-based recommendation algorithm didn't work?

![experiment 1 US2020 source-based recommendations](images/fastai_US2020_exp1.png)
![experiment 1 UK2019 source-based recommendations](images/fastai_UK2019_exp1.png)

Upon inspection, it is clear that the experimental methodology was more effective at manipulating recommendations for the UK2019 dataset. This is noticeable at lower values (200 amplifiers with 50 retweets were required to obtain 50% coverage in the US2020 dataset versus 200 amplifiers and 10 retweets for the UK2019 dataset). At higher values, we observe that 1000 amplifiers with 1 retweet and 2000 amplifiers with 1 retweet were much more impactful on the UK2019 collaborative filtering models.

It is interesting to observe what happens to graph visualizations of these networks as we add edges. Here's the baseline UK2019 dataset with the target and high-profile accounts highlighted. Note the separation between the two accounts of interest.

![UK2019 baseline annotated](images/fastai_UK2019_base_trimmed_anon.png)

Here's the same graph, but with 500 amplifiers, 1 retweet each. The target and high-profile nodes have moved closer together.

![UK2019 500_1 annotated](images/fastai_UK2019_exp1_500_1_9_anon.png)

With 1000 amplifiers, 1 retweet each. Nodes move even closer.

![UK2019 1000_1 annotated](images/fastai_UK2019_exp1_1000_1_1_anon.png)

Note how close together the two nodes of interest are when we use 200 amplifiers, 20 retweets each.

![UK2019 200_20 annotated](images/fastai_UK2019_exp1_200_20_4_anon.png)

This attack methodology becomes more effective at low numbers of amplifiers (100-500) when the number of retweets per account is increased. This is a significant finding for individual attackers who don't have access to large numbers of fake accounts. The effectiveness of attacks involving higher numbers of accounts and lower retweet counts are especially revealing, since these are exactly the sorts of actions that are orchestrated by communities on Twitter, especially when sharing disinformation.

Since the US2020 dataset was created by following specific Twitter accounts, it is representative of the sort feed Twitter's own algorithms would need to process in order to deliver that user a curated timeline. Twitter's curated timeline is the default display option for a user's home timeline, and thus if it were to use a retweet similarity algorithm similar to the one implemented in these experiments, it would be susceptible to such manipulation. Twitter has openly stated that they factor "Likes" into building timelines, so it may be possible that a similar mechanism to the one documented here using the "Like" button instead of the "Retweet" button may be effective at surfacing content on the control users' timelines. This method may also be relevant if Twitter's follower recommendations are based on accounts they most often interact with (although that is not likely to be the case).

## Experiment 2: Amplifiers chosen based on community

The following bar charts depict experimental results from selecting amplifiers based on Louvain community detection applied to a node-edge graph of the baseline datasets, and using fixed 200 amplifiers with 20 retweets parameters.

Here's the bar chart for the US2020 dataset. Bear in mind that using randomly chosen amplifiers resulted in about 30% of the control set being recommended the target account.
![experiment 2 US2020 source-based recommendations](images/fastai_US2020_exp2_1_annotated.png)

Here we can see that community had significant effects on the resulting recommendations. Communities 6 and 7 saw a marked reduction in effectiveness of the attack as compared to using randomly selected amplifiers. Communities 8 and 18 worked significantly better than the baseline. Note that assigning amplifier accounts from communities 1 and 3 (the communities that the high-profile and target accounts belonged to) had very little impact on the effectiveness of the attack.

Here's the bar chart for the UK2019 dataset. Bear in mind that using randomly chosen amplifiers resulted in about 50% of the control set being recommended the target account.
![experiment 2 UK2019 source-based recommendations](images/fastai_UK2019_exp2_1_annotated.png)

Community membership had little effect on recommendation outcomes in the UK2019 case.

This is an interesting result. I would have assumed that conducting an attack using accounts that behave in a similar way to the target or high-profile account would be more effective. That didn't turn out to be the case. The fact that community membership had a significant effect in one dataset and not in another leads me to believe that network structure plays a role in the effectiveness of these attack mechanisms in general. It would be interesting to capture more datasets in order to determine what graph structural features impede or assist these attack mechanisms.

## Experiment 3: Amplifiers chosen based on similarity to control accounts

The following bar charts depict experimental results where amplifiers with high similarity to the twenty control accounts were selected. Similarity values were obtained from the source-based similarity matrix calculated for the baseline dataset in each case. A range of amplifier count - retweet count parameters were tried.

![experiment 3 US2020 source-based recommendations](images/fastai_US2020_exp3.png)

![experiment 3 UK2019 source-based recommendations](images/fastai_UK2019_exp3.png)

Generally speaking, choosing accounts similar to the control set imparted a roughly 10% increase to the effectiveness of baseline attacks that achieved 60% or less coverage (as compared to randomly selected accounts). Since more aggressive attacks (those that utilized 1000 or more amplifiers) were already nearing full coverage, choosing amplifiers similar to the control in this case set had no noticeable effect.

This result is interesting when considering a social engineering attack scenario. An attacker who wishes to specifically target an individual can instruct fake accounts to behave the same way as the victim account - i.e. have those fake accounts spend a few days retweeting the same content as the victim account prior to launching the actual attack. Since social network recommendations are likely only calculated across a recent slice of activity, it should be easy for an attacker to sync their fake accounts with the victim's account in this way.


# Contribution

While researching this topic, I found many github repositories implementing fastai's collab functions, but almost all of them were copy-pasted from the tutorial on fastai's site. The notebooks in this repository contain fully coded and documented examples of how to use fastai's collab library to perform collaborative filtering and recommendation mechanisms, which I hope will be of interest to the community. The experiments described here utilize real-world Twitter data, as opposed to the movielens or goodbooks datasets that were used in all other code examples I could find. I have made these datasets publicly available in hopes that they may be used in future research. This github repository contains well-documented steps (and code) that illustrate a number of interesting attacks against collaborative filtering models and recommendation mechanisms. These attacks can be mapped to real-world Twitter usage. Finally, this report contains a number of suggestions for directions to take this research in.

# Conclusions

The experiments detailed in this report illustrate that Sybil attacks against recommendation mechanisms are fairly easily to carry out. Even using relatively low numbers of amplifiers and retweet counts, it was possible to significantly alter both network structure and collaborative filtering models. The attacks detailed in this report bear a likeness to tactics already used on Twitter to spread disinformation and amplify political content. As such, it it likely that the groups of accounts participating in these attacks are already successfully altering Twitter's underlying recommendation mechanisms. The people involved in followback rings likely aren't considering Twitter's algorithms when they amplify content. Instead they've unconsciously discovered behavioural patterns that work get their content and messages out, albeit to others within their own filter bubble. If these groups were capable of running their own experiments to probe Twitter's recommendation mechanisms, they could, in theory, adopt different patterns of behaviour that would allow them to spread disinformation to the wider Twittersphere.

Detecting the mechanisms behind these attacks would likely be difficult, and may differ based on which attacks are conducted. Attacks involving large numbers of amplifiers all publishing a single, or small number of retweets are similar in nature to organic mechanisms such as tweets going viral (a user posts a tweet that resonates with people, and that tweet receives many retweets despite that user not normally receiving high levels of engagement). Some circles on Twitter consist of two types of accounts - influencers who mainly post original content, and consumers who don't usually post their own content but instead read and retweet content from others. In these circles, tweets that influencers publish usually receive hundreds or even thousands of retweets largely from the same group of consumer accounts. Because of the fact that much of Twitter's organic activity follows these mechanisms, identifying users that always retweet content from certain accounts is not a viable strategy for finding coordinated behaviour. Tweets with "retweet this" are, of course, a signal that the publisher of the tweet wants it to be amplified, but tweets containing that phrase are rather common on Twitter, and don't necessarily reflect true coordination (some of which is likely organized on platforms other than Twitter).


  - for larger numbers of retweets, it should be possible to notice a change in retweet freq dist and determine which accounts participated
  - XXX show retweet frequency distribution
  - if the distance between two nodes on a graph changes suddenly and drastically, it might be an indication of this

Future directions
- try different (perhaps more complex) recommendation logic
- try other datasets
- try other interaction schemes (e.g. user-tweet, user-hashtag, etc.)
- attempt to build defenses against these attacks
  - retweet count distribution changes
  - radical changes in the graph distance between two nodes
- understand in more detail how graph features (communities, etc.) change when nodes and edges are added

# Appendix

## Libraries used

- Numpy (https://numpy.org/)
- Pandas (https://pandas.pydata.org/)
- NetworkX (https://networkx.org/)
- Louvian community detection (https://github.com/taynaud/python-louvain)
- Collaborative filtering model: fastai collab_learner (https://docs.fast.ai/collab)
- Cosine similarity matrix: sklearn cosine_similarity (https://scikit-learn.org/stable/modules/generated/sklearn.metrics.pairwise.cosine_similarity.html)
- Visualizsation: matplotlib (https://matplotlib.org/) and seaborn (https://seaborn.pydata.org/)

## Additional graph visualizations


