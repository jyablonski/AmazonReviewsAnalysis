# AmazonReviewsAnalysis
This project goes through my process of analyzing Amazon Reviews to find trends & insights.  The dataset is available here https://registry.opendata.aws/amazon-reviews/.  The data includes a subset of Amazon Reviews related to Video Game products from December 2012 to August 2015.  There are a little over 1+ million rows in the dataset with variables like Product Title, the Headline of the Review, the Body of the Review, Date, and the total amount of Help Votes given to the Review.

I specifically analyzed the entire dataset for this project using R, including the Tidytext, Tidyverse, and Fpp3 packages.  Tidytext allows me to perform the Sentiment Analysis portion of the project, and Fpp3 consists of the forecasting packages.

This is just a simple time series of the data, the red lines represent the spikes which typically occur in December & January around the Holiday Season.  There's also a general upward trend as time increases from Amazon becoming more and more popular.  These spikes & general upward trend are pretty normal & exactly what we would expect to see.

![amazonreviews](https://user-images.githubusercontent.com/16946556/75714753-246ed000-5c81-11ea-8c7a-ef3c175e144e.png)

Using basic dplyr I found a quick top 10 count for the top products & the most frequent reviewers.  I also contemplated filtering specific Titles that were popular (Products like the PlayStation 4 or games like Grand Theft Auto: V) which could have interesting results if you're interested in analyzing a certain product.

![customer count](https://user-images.githubusercontent.com/16946556/76690608-d94ca980-65fe-11ea-8326-4c930987dd3c.png)
![amazon top products](https://user-images.githubusercontent.com/16946556/76690609-d9e54000-65fe-11ea-8369-25138ae047ca.png)


I then performed Sentiment Analysis and analyzed the top 25 words in the bodies of the Reviews and the top 15 most popular words by positive and negative connotation via the Bing Sentiment.  

![amazontop25wordsReviews](https://user-images.githubusercontent.com/16946556/75707751-57f72d80-5c74-11ea-8588-4a98a78d4624.png)

![amazontop15Bing](https://user-images.githubusercontent.com/16946556/75707757-59c0f100-5c74-11ea-8808-b8cae7c365dd.png)

The AFINN Sentiment is another way of splitting words up by applying a value of -5 (most negative) to 5 (most positive) to each word that is analyzed.  Here i'm just separating them into 2 distinct categories and trying to spot any obvious differences because positive reviews can be quite different than negative ones.  Words that got assigned values of -5 and 5 were extremely rare and difficult to conduct any meaningful analysis on because of the small sample size

![amazonnegativewords](https://user-images.githubusercontent.com/16946556/75707755-59c0f100-5c74-11ea-8c84-96dfbd4d6ec7.png)
![amazonpositivewords](https://user-images.githubusercontent.com/16946556/75707756-59c0f100-5c74-11ea-92f9-3704e20ce6ba.png)

Here is a simple wordcloud of the most popular positive & negative words.  The more common the word, the larger it is. 

![Amazon Wordcloud](https://user-images.githubusercontent.com/16946556/75715404-369d3e00-5c82-11ea-8078-a64f19cd94ee.png)


I also wanted to work with forecasting a bit to see if I could predict future values of the number of reviews.  While it's obviously not as valuable as simply predicting sales, it allowed me to work with forecasting concepts to build a few realistic models.  Below are both ARIMA & Exponential Smoothing Models, with 80% and 95% confidence intervals.  The residuals checked out fine, and I ran a Ljung-Box Test to ensure there was no autocorrelation present in the time series.  

The original dataset stopped in August 2015, so I forecasted the next year (12 periods) of Amazon Reviews.

The AUTO ARIMA forecast provided (1, 0 ,0) (0, 1, 0) model, indicating there is an AR component and a seasonal component included in the model.
![amazonautoarima](https://user-images.githubusercontent.com/16946556/75707752-59285a80-5c74-11ea-8ee0-585e1cbd118c.png)

![amazonexponentialsmoothing](https://user-images.githubusercontent.com/16946556/75707753-59285a80-5c74-11ea-8d17-ed225f31399d.png)

There was an obvious trend in the data as well as seasonality around the Holiday Season, so both of these aspects were included in the two final models.  
