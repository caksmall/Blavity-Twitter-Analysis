# Blavity Twitter Analysis
This repository contains the R analysis used to gather the insights, and create the figures, published in this [piece](https://blavity.com/blavity-original/green-twitter-how-environmental-twitter-fails-to-highlight-injustice) for Blavity : Politics. The tweets – and corresponding Twitter profile information – was compiled using the [rtweet](https://cran.r-project.org/web/packages/rtweet/index.html) package in conjunction with Twitter's [REST API](https://www.w3resource.com/API/twitter-rest-api/) to pull Tweets from a user's profile. It's worth noting that the REST API limits your requests to, approximately, the 3,200 most recent tweets from a user's profile. In order to collect as many tweets as I did (between 4,000 to 7,000), I compiled Tweets from each profile twice, roughly two weeks apart. This will give you duplicate Tweets given the time-overlap, so I had to remove the redundant tweets. This process is explained in greater detail below (and in the matching RMD file). 

## Getting Started

The following instructions – and the RMD file – will allow you to duplicate this same process, or utilize this process for a similar project. **_Important:_** In order to carry out analyses using the REST API, you must sign up for a [developer Twitter Account](https://www.extly.com/docs/autotweetng_joocial/tutorials/how-to-auto-post-from-joomla-to-twitter/apply-for-a-twitter-developer-account/#apply-for-a-developer-account). It's advisable to sign up for one a few days before you intend to do this analysis. The vetting process usually takes around 3 days. The verification process can be shorter _or longer_ than this duration depending the detail and length of your application answers. 

Aside from developer access on Twitter, you'll just need R and RStudio installed on your computer. You'll also need the following packages:

### Prerequisites

You'll need the following packages for your analysis:

For general data manipulation:
```
install.packages("dplyr")
```

In order to download the Twitter Data:
```
install.packages("rtweet")
```
For plotting:
```
install.packages("tidyverse")
install.packages("ggplot2")
```
To filter for specific words and phrases in Tweets:
```
install.packages("stringr")
```

To change the dates of the Tweets to date-time:
```
install.packages("lubridate")
```

Tweets can have emojis and other graphics in them that make exporting to Excel difficult. Data Table allows you to use `fwrite` which saves your data as a CSV by removing these graphics:
```
install.packages("data.table")
```

For sentiment analyses and specific word usage data **_(insights that didn't make it into the piece)_**, you'll need:

```
install.packages("syuzhet")
install.packages("tidytext")
```

Finally (in the event I forgot a package), use this to figure out which packages correspond to which packages:
```
install.packages("sos")
```
For example, if you need to find the `get_timeline` function (it's in `rtweet`)
```
findFn("get_timeline")
```

### Enabling Twitter Access
After your Developer Twitter account is approves, you'll still need to create an [app](https://docs.inboundnow.com/guide/create-twitter-application/) in order to use R for Twitter analysis. Once you have your credentials – which you should keep in a safe place, and not share – you'll have to run the following code to pull users' profile timelines.

```
Your_token <- create_token(
  app = "The name of your app",
  consumer_key = "******************",
  consumer_secret = "*************************************************",
  set_renv = TRUE)
```
If you're not already logged into Twitter, R will open a window in your browser and you'll be prompted to sign in. After you sign in, you'll be able to request users' Twitter profiles.

## Requesting Tweets, Organizing Your Data, and Creating Insights

Below, I'll explain what you'll need to do in order to create a word analysis from Twitter. This readme only shows examples of how to do the analysis, but if you'd like to see exactly what I did, take a look at my R markdown.

### Requesting the Tweets

The `get_timelines` function will pull every tweet in a users' timeline between certain dates. As I mentioned above, Twitter limits you to the last ~3,200 tweets in a user's profile. If you want all of the available Tweets from a user, set your date bounds as wide as possible, and set your n (number of tweets) far above 3,200.

For example, the following code will pull all of the available tweets from the EPA:

```
EPA <- get_timelines("EPA", 
                        n = 10000, 
                        language = 'en',
                        since = '2016-01-01', 
                        until = '2019-10-23',
                  )
```

Let's say you want to add more tweets to your analysis, so you wait an extra three weeks to do another analysis:

```
EPA_New <- get_timelines("EPA", 
                        n = 10000, 
                        language = 'en',
                        since = '2016-01-01', 
                        until = '2019-11-30',
                  )
```
If you want to combine the data frames together, you can `rbind` the data frames, and then remove the duplicates _(there will be necessary overlap in the dates)_.

Merge the data frames:

```
EPA_total <- rbind(EPA, EPA_New)
```

Remove the duplicates:

```
EPA_total <- EPA_total[!duplicated(EPA_total$status_id), ]
```
I chose to filter based on redundant `status_id`, but you can also reasonably filter by another distinct characteristic – like `created_at`. Now you have the data you'll be investigating!

### Organic Tweets, Retweets, and Replies

Now we'll break the data up into organic tweets (original tweets from the specific user), retweets, and replies. **This information wasn't necessarily useful for my piece, but it might be for other insights.**

First, we're going to use the favorite and retweet counts to determine whether a Tweet is organic:

```
#adding favorite count

EPA_Organic  <- EPA_Organic  %>%
  arrange(-favorite_count)
  
#adding retweet count from other users

EPA_Organic <- EPA_Organic %>%
  arrange(-retweet_count)
```
We'll have all of our organic tweets set up in `EPA_Organic`.

Now we'll filter just the retweets:

```
#keeping only retweets

EPA_retweets <- EPA_total[EPA_total$is_retweet==TRUE,]
```

Finally, replies:

```
EPA_replies <- subset(EPA_total, !is.na(EPA_total$reply_to_status_id))

```

### Searching for Certain Terms and Words in Tweets

For this piece, I searched for tweets related to Environmental Racism by filtering for a variety of terms and words: **_racism_**, **_environmental racism_**, **_people of color_**, **_communities of color_**, and **_environmental justice_**. I also searched for the following hashtags: **_#environmentalracism_**, and **_#environmentaljustice_**. This is primarily done with `str_detect`.

You can check for these terms within the entire data set by investigating the `text` variable (column):

```
EPA_enve_racism <- EPA_total %>%
  filter(str_detect(text, "racism|environmental racism|people of color|communities of color|#environmentalracism|environmental justice|#environmentaljustice"))
```
Or you can check for the breakdowns by organic tweet, retweet, or reply:
```
EPA_racism_org <- EPA_Organic %>%
  filter(str_detect(text, "racism|environmental racism|people of color|communities of color|#environmentalracism|environmental justice|#environmentaljustice"))

EPA_racism_rt <- EPA_retweets %>%
  filter(str_detect(text, "racism|environmental racism|people of color|communities of color|#environmentalracism|environmental justice|#environmentaljustice"))

EPA_racism_rply <- EPA_replies %>%
  filter(str_detect(text, "racism|environmental racism|people of color|communities of color|#environmentalracism|environmental justice|#environmentaljustice"))
```
**Note:** If you want to pull up a Tweet in Twitter, copy the `status_id`, and past it into the end of the URL for any tweet from that Twitter profile. For example, a Tweet from the EPA with the `status_id` of "821725248483684352" takes you [here](https://twitter.com/EPA/status/821725248483684352).

### Word Usage and Sentiment Analysis
Once you have all of the Tweets you want to use, you can also do a [sentiment analysis](https://www.datacamp.com/community/tutorials/sentiment-analysis-R) of a user's tweets. It's also possible to determine the most frequent words used by a user in their tweets. 
#### Getting the Data Organized
Before you can analyze the tweets for specific words, you'll have to break up the tweets into a "bag of words." In order to do that, however, you'll have to clean the text by removing punctuation, "http://", and other objects that will be in your tweets. 

First, we'll create a separate data frame for our cleaned date:
```
EPA_total_words <- EPA_total
```
Now we can remove unnecessary items:
```
EPA_total_words$text <- gsub("https\\S*", "", EPA_total_words$text)

EPA_total_words$text <- gsub("@\\S*", "", EPA_total_wordsd$text)

EPA_total_words$text <- gsub("amp", "", EPA_total_words$text)

EPA_total_words$text <- gsub("[\r\n]", "", EPA_total_words$text)

EPA_total_words$text <- gsub("[[:punct:]]", "", EPA_total_words$text)
```
From here, we can break up the tweets into a "bag of words."

```
EPA_words <- EPA_total_words %>%
  select(text) %>%
  unnest_tokens(word, text)
```

Finally, we can remove articles (by eliminating `stop_words`), and retain just the meaningful words:
```
EPA_words <- EPA_words %>%
    anti_join(stop_words)
```

### Plotting
#### Time Series (Histogram)

Using `ts_plot`, we can visualize how frequently a profile may tweet on a certain topic. 

```
EPA_enve_racism %>%
  ts_plot("1 day") +
  ggplot2::theme_minimal() +
  ggplot2::theme(plot.title = ggplot2::element_text(face = "bold")) +
  ggplot2::labs(
    x = NULL, y = NULL,
    title = "Frequency of EPA Tweets on Environmental Racism",
    subtitle = "Whatever subtitle you find appropriate",
    caption = "Source: Data collected from Twitter's REST API via rtweet"
  )
```
![Graphic Used For the Peice](https://s3-us-west-1.amazonaws.com/21ninety-media/images%2F1575421372885-1575421384956-EPA+Racism+Frequency.jpeg)
This analysis uses daily intervals for the `ts_plot` argument, but the interval can be changed. To get the best definition, set the export dimensions to 1000 x 533.

#### Most Used Words

The following code will let you generate a graph of the most used words via `ggplot`.

```
EPA_words %>%
  count(word, sort = TRUE) %>%
  top_n(15) %>%
  mutate(word = reorder(word, n)) %>%
  ggplot(aes(x = word, y = n)) +
  geom_col() +
  xlab(NULL) +
  coord_flip() +
  labs(y = "Count",
       x = "Unique Words",
       title = "Most Frequent Words EPA Tweets",
       subtitle = "Articles Removed")
```
**_Note:_** This repository will show this kind of word analysis for different Twitter accounts.

#### Sentiment Analysis

Sentiment analysis splits up the words into positive and negative emotional categories. 

To start, you'll have to convert the text encoding from UTF-8 to ASCII:

```
EPA_words <- iconv(EPA_words, from = "UTF-8", to = "ASCII", sub = "")
```
Then, we need to remove any references to retweets or mentions:

```
#removing retweets in case needed
EPA_words <- gsub("(RT|via) ((?:\\b\\w*@\\w+)+)", "", EPA_words)

#removing mentions
EPA_words <- gsub("@\\w+", "", EPA_words)
```

Next, we'll use the `get_nrc_sentiment` to access the NRC lexicon in order to assign sentiments to the words:

```
EPA_sentiments <- get_nrc_sentiment((EPA_words))

sentimentscores <- data.frame(colSums(EPA_sentiments[,]))
```
We'll now organize this data frame for use in `ggplot`:

```
names(sentimentscores) <- "Score"

sentimentscores <- cbind("sentiment"=rownames(sentimentscores), sentimentscores)

rownames(sentimentscores) <- NULL
```

And now we can plot with `ggplot`:

```
ggplot(data = sentimentscores, aes(x = sentiment, y = Score)) +
  geom_bar(aes(fill = sentiment), stat = "identity") +
             theme(legend.position = "none") +
             xlab("Sentiments") + ylab("Scores") +
             ggtitle("Total EPA sentiment based on scores") + theme_minimal()
```

**_Refer to repository for these graphics_**
### Saving as CSV
If for some reason, you want to save your tweets, the `fwrite` function makes it so easy. This function will remove any emojis that would stop other functions from writing Excel spreadsheets or comma delimited files:

```
fwrite(EPA_retweets, file ="US EPA Retweets.csv")
```

## Final Thoughts
Feel free to use this template to conduct your own Twitter analysis in R. Files used for the piece, and graphics produced, are in folders in this repository.
