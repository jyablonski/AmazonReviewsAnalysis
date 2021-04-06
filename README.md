# Amazon Reviews Analysis

## Introduction
This project goes through my process of analyzing Amazon Reviews to find trends & insights.  The dataset is available here https://registry.opendata.aws/amazon-reviews/.  The data includes a subset of Amazon Reviews related to Video Game products from December 2012 to August 2015.  There are a little over 1+ million rows in the dataset with variables like Product Title, the Headline of the Review, the Body of the Review, Date, and the total amount of Help Votes given to the Review.

I specifically analyzed the entire dataset for this project using R, including the Tidytext, Tidyverse, and Fpp3 packages.  Tidytext allows me to perform the Sentiment Analysis portion of the project, and Fpp3 consists of the forecasting packages.

I also built an interactive [R Shiny Dashboard](https://jyablonski.shinyapps.io/amazon-dashboard/) that allows user to analyze specific products from this particular dataset.

## Exploratory Analysis
This is just a simple (daily) time series of the data, the red lines represent the spikes which typically occur in December & January after the Holiday Season.  There's also a general upward trend as time increases from Amazon becoming more and more popular over the decade.  These spikes & the general upward trend are pretty normal & exactly what we would expect to see.

![amazonreviews](https://user-images.githubusercontent.com/16946556/113646472-bba11680-963d-11eb-94e7-2b42fa009f42.png)

Using basic dplyr I found a quick top 10 count for the top products & the most frequent reviewers.  I also contemplated filtering specific Titles that were popular (Products like the PlayStation 4 or games like Grand Theft Auto: V) which could have interesting results if you're interested in analyzing a certain product.

![customer count](https://user-images.githubusercontent.com/16946556/76690608-d94ca980-65fe-11ea-8326-4c930987dd3c.png)
![amazon top products](https://user-images.githubusercontent.com/16946556/76690609-d9e54000-65fe-11ea-8369-25138ae047ca.png)


Here is the distribution of the Rating Counts in the dataset, along with the actual numbers & percentages listed below.  Over 78% of the ratings were 4 or 5 stars, and the two least popular choices were 2 & 3 star ratings.  This seems pretty ordinary for rating data, customers tend to gravitate to the extreme ends of voicing their pleasure or displeasure when it comes to recommending a product to other people.

![amazonratingcounts](https://user-images.githubusercontent.com/16946556/113646470-bba11680-963d-11eb-8ab3-379da23f10d7.png)
![rating counts imgur](https://user-images.githubusercontent.com/16946556/77356710-16eaba00-6d04-11ea-98d0-b7f984f1c8b3.png)

Below is a distribution of the total number of words in each review.  Users typically tend to keep it short, with the majority of reviews containing less than 50 words.  

![AmazonHistogramWords](https://user-images.githubusercontent.com/16946556/81755474-b9eec300-946d-11ea-9395-2fc25db1909e.png)


## Sentiment Analysis
I then performed Sentiment Analysis and analyzed the top 25 words in the bodies of the Reviews and the top 15 most popular words by positive and negative connotation via the Bing Sentiment.  

![amazontop25wordsReviews](https://user-images.githubusercontent.com/16946556/75707751-57f72d80-5c74-11ea-8588-4a98a78d4624.png)

![amazontop15Bing](https://user-images.githubusercontent.com/16946556/113646462-ba6fe980-963d-11eb-8136-bae26ac663eb.png)

Here is a simple wordcloud of the most popular positive & negative words.  The more common the word, the larger it is. 

![Amazon Wordcloud](https://user-images.githubusercontent.com/16946556/75715404-369d3e00-5c82-11ea-8078-a64f19cd94ee.png)


This is a full bigram plot using the ggraph package utilizing the entire dataset.  I recommend clicking on it and zooming it, but these are the most commonly associated word pairs in the body text of these Amazon Reviews, colored by their average rating.  
![amazonggraphnew](https://user-images.githubusercontent.com/16946556/113646467-bb088000-963d-11eb-98e0-6cfcdb8d64c5.png)


## LASSO Regression
I utilized LASSO Regression to help give me a more formal and technical understanding of which words were most meaningful in classifying a positive or negative review.  For this analysis I considered a 'Positive' review to be a rating of 4 or 5, while 'Negative' reviews consisted of ratings that are 1, 2, or 3.  This dataset was limited to 100,000 of the dataset's 1+ million reviews because of limitations with the text mining & machine learning algorithms.

Below are a list of the steps taken using the Tidymodels package to conduct the Machine Learning.  This performs the text mining of the machine learning model, removes stop words, selects 500 of the most important words when taking into account their term frequency (tf-idf), and normalizes the data to have a standard deviation of 1 and a mean of 0.  The data was split into 75% Training Data, 25% Testing Data.

![amazon workflow](https://user-images.githubusercontent.com/16946556/81755962-3504a900-946f-11ea-8835-c5d07ca19eb5.png)

Here are the results of the most meaningful predictor words of Positive and Negative reviews.  It was much easier to distinguish positive reviews which is why there is noticeably more importance on those words than those of the negative words.  As this dataset is related to Video Game Products, it makes sense that the presence of words like broken, return, and stopped generally led to more negative reviews. 

![AmazonLASSO](https://user-images.githubusercontent.com/16946556/81755471-b9562c80-946d-11ea-9842-ffe136ab037e.png)

![amazon_heatmap](https://user-images.githubusercontent.com/16946556/113646466-bb088000-963d-11eb-9727-b7a0a6dcae86.png)

## Forecasting the Number of Reviews
I also wanted to work with forecasting a bit to see if I could predict future values of the number of reviews.  While it's obviously not as valuable as simply predicting sales, it allowed me to work with forecasting concepts to build a few realistic models.  Below are both ARIMA & Exponential Smoothing Models, with 80% and 95% confidence intervals.  The residuals checked out fine, and I ran a Ljung-Box Test to ensure there was no autocorrelation present in the time series.  

The original dataset stopped in August 2015, so I forecasted the next year (12 periods) of Amazon Reviews.

The AUTO ARIMA forecast provided (1, 0 ,0) (0, 1, 0) model, indicating there is an autoregressive component and a seasonal component included in the model.
![amazonautoarima](https://user-images.githubusercontent.com/16946556/113646461-ba6fe980-963d-11eb-830d-f7097642d8f3.png)

![amazonexponentialsmoothing](https://user-images.githubusercontent.com/16946556/113646459-b9d75300-963d-11eb-99ca-25f9e2a88a20.png)

There was an obvious trend in the data as well as seasonality around the Holiday Season, so both of these aspects were included in the two final models. 



![amazonpositivewords](https://user-images.githubusercontent.com/16946556/113646463-ba6fe980-963d-11eb-8092-1e7ea0ece498.png)
![amazonnegativewords](https://user-images.githubusercontent.com/16946556/113646464-ba6fe980-963d-11eb-865b-41ab9a2ceba2.png)
![amazon_importance](https://user-images.githubusercontent.com/16946556/113646465-bb088000-963d-11eb-8fb1-2f4f73885dc5.png)
![amazon_heatmap](https://user-images.githubusercontent.com/16946556/113646466-bb088000-963d-11eb-9727-b7a0a6dcae86.png)

![amazonwordpairs](https://user-images.githubusercontent.com/16946556/113646469-bba11680-963d-11eb-8eb1-8c912d23a65e.png)



