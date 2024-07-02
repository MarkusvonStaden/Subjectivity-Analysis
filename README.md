# Objectivity/Subjectivity analysis on newspaper articles

Training a model to detect wether a given text is objective or not.

## Idea

Inspired from the popular IMDB Dataset, which contains 50.000 reviews with label "positive" or "negative" and is often used as a hello-world project for sentiment analysis, I had the Idea to scrape Movie Reviews and Summaries from IMDB.
The Idea is, that reviews are subjective while summaries are objective.

## Implementation

### Get Movie URLs

In order to scrape storylines and reviews, we first need urls to as many movies as possible.
Here I used the "advanced search" feature from IMDB with the only filter beeing the number of votes in order to filter out small movies with unreliable summaries. The main advantage of using the advanced search is, that all urls are present nicely in order on one single page.

The Drawback of that approach, or IMDB in general, is that most content is loaded dynamically and thus very difficuly to scrape using scrapy. I tried to reverse engineer the requests, but had no luck.

So I moved from scrapy to selenium, which opens a browser instance and renders the webpage. The script would open the advanced search, scrape the shown 50 Titles and simulate a button click on "show 50 more". After that it waits until 50 more movies are loaded and then does the same again. Although this approach worked fine, there where some hurdles:

1. Since the page is dynamic and on every click appends 50 movies, the page gets huge and needs a lot of memory and computaion power to render

2. In order to prevent scraping, IMDB adds a penalty after a few requests. This significantly slows down the process

Initially I had planned to get urls to 30.000 movies, but the script crashed multiple times around 7500 urls. Sometimes due to memory consumption, sometimes the browser crashed.

Finally I had 7.750 URLs

### Get Reviews and Summaries

Initially I started using scrapy as well. That turned out more difficult than I thought, since here also the content is loaded dynamically and even worse: The container names and ids differ from movie to movie. So here also I moved to selenium. Most content on these pages is lazy-loaded when the container is scrolles into view.
So at first I located the element containing the text "Storyline" and scrolled that into view. This action triggers the storyline itself to load. The same was done for the Review. Now that I had Summary and Review, I stored that away together with the URL.

This process took quite some time, but fortunally I could run multiple instances (12) in parallel. This resulted in 9344 texts. (I was impatient and did not want to wait for the entire list to finish)

### Data Preprocessing

This data now contained some impurities like the name of the author and sometimes a rating at the beginning.
Also the JSON format used before cannot be read with tensorflow.

So the cleanup script takes all unwanted elements and stores them in a file structure, that tensorflow can easly read.

### Classification

For classification I decided to use the BERT model by Google.
BERT is a Large Language Model based on the transformer architecture and comes in different variants, different sizes and pretrained with different data.

Google has an excellent tutorial on how to build a taxt classifier based on BERT, which this is heavily based on.
This resulted in an accuracy of 99.3% after 5 epochs of training.

The model is build as following:

First the text is preprocessed and converted to tokens, before fed into the BERT layers.
The BERT layers are trainable, meaning altough the model is already pretrained, during training the model weights can still change.
The output of the BERT-encoders are fed into a dropout layer to prevent overfitting.
The final layer is a dense/fully-connected layer with 1 output neuron.
The output of this neuron can now be used as a classification when combined with a threshold.

## Conclusion

The Idea was, that a model trained using movie reviews and summaries can also distinguish betwenn objective and subjective in other texts. Altough I tested it succesful with a handful of texts, this is not a reliable metric.

The other problem is, that this model is only based on phrasing and does not perform any type of fact checking.
If a opinion is formulated as facts, the model still detects that the text is objective.
Thus the model is not useful for fake news detection.

Tested on 500 Articles on the "tagesschau-meldungen" dataset returns 12 subjective results.
Given that tagesschau is always objective this is an accuracy of 97.6%, which is quite remarkable.
This also shows a great benefit of using the pretrained BERT model: All movie summaries and reviews were english, but the model can still classify the german news articles.

# Number

Training time for all final models combined: 9h on Google Colab Pro (NVidia Tesla T4 GPU)
