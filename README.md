# AmazonReviewsAnalysis
This project goes through my process of analyzing Amazon Reviews to find trends & insights.  The dataset is available here https://registry.opendata.aws/amazon-reviews/.  The data includes a subset of Amazon Reviews related to Video Game products from December 2012 to August 2015.  There are a little over 1+ million rows in the dataset with variables like Product Title, the Headline of the Review, the Body of the Review, Date, and the total amount of Help Votes given to the Review.

This is just a rough time series of the data, the red lines represent the spikes which typically occur in December & January around the Holiday Season.  There's also a general upward trend as time increases from Amazon becoming more and more popular.  This is pretty normal & exactly what we would expect to see.

![amazonreviews](https://user-images.githubusercontent.com/16946556/75714753-246ed000-5c81-11ea-8c7a-ef3c175e144e.png)

I then analyzed the top 25 words in the bodies of the Reviews and the top 15 most popular words by positive and negative connotation via the Bing Sentiment.  

![amazontop25wordsReviews](https://user-images.githubusercontent.com/16946556/75707751-57f72d80-5c74-11ea-8588-4a98a78d4624.png)

![amazontop15Bing](https://user-images.githubusercontent.com/16946556/75707757-59c0f100-5c74-11ea-8808-b8cae7c365dd.png)

The AFINN Sentiment is another way of splitting words up by applying a value of -5 to 5 to each word.  Here i'm just separating them into 2 distinct categories and trying to spot any obvious differences because positive reviews can be quite different than negative ones.  Words that got assigned values of -5 and 5 were extremely rare and difficult to conduct any meaningful analysis on.

![amazonnegativewords](https://user-images.githubusercontent.com/16946556/75707755-59c0f100-5c74-11ea-8c84-96dfbd4d6ec7.png)
![amazonpositivewords](https://user-images.githubusercontent.com/16946556/75707756-59c0f100-5c74-11ea-92f9-3704e20ce6ba.png)

Here is a simple wordcloud of the most popular words.  The more common the word, the larger it is. 

![Amazon Wordcloud](https://user-images.githubusercontent.com/16946556/75715404-369d3e00-5c82-11ea-8078-a64f19cd94ee.png)


I also wanted to work with forecasting a bit to see if I could predict future values of the number of reviews.  While it's obviously not as valuable as simply predicting sales, it allowed me to work with forecasting concepts to build a few realistic models.  Below are both an ARIMA & Exponential Smoothing Models, with 80% and 95% confidence intervals.  The residuals checked out fine, and i ran a Ljung-Box Test to ensure there was no more autocorrelation in the time series.  

The original dataset stopped in August 2015, so I forecasted the next year (12 periods) of Amazon Reviews.

![amazonautoarima](https://user-images.githubusercontent.com/16946556/75707752-59285a80-5c74-11ea-8ee0-585e1cbd118c.png)

![amazonexponentialsmoothing](https://user-images.githubusercontent.com/16946556/75707753-59285a80-5c74-11ea-8d17-ed225f31399d.png)
