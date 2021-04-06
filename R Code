# loading packages and adjusting keywords ----
# The Data - https://registry.opendata.aws/amazon-reviews/
# Data from 12-11-2012 to 8-31-2015

library(tidytext)
library(tidyverse)
library(extrafont)
library(magick)
library(scales)
library(lubridate)
library(igraph)
library(ggraph)
library(widyr)
library(wordcloud)
library(reshape2)
library(GGally)
library(fpp3)
library(sugrrants)
library(textrecipes)
library(knitr)
library(vip)
library(skimr)
library(tidymodels)

dateVariable <- Sys.Date()
dateVariable <-  format(dateVariable, format = "%B %d, %Y")

# custom theme
theme_jacob <- function(..., base_size = 11) {
  theme(panel.grid.minor = element_blank(),
        panel.grid.major =  element_line(color = "#d0d0d0"),
        panel.background = element_rect(fill = "#f0f0f0", color = NA),
        plot.background = element_rect(fill = "#f0f0f0", color = NA),
        legend.background = element_rect(fill = '#f0f0f0', color = NA),
        legend.position = 'top',
        panel.border = element_blank(),
        strip.background = element_blank(),
        plot.margin = margin(0.5, 1, 0.5, 1, unit = "cm"),
        axis.ticks = element_blank(),
        text = element_text(family = "Gill Sans MT", size = base_size),
        axis.text = element_text(face = "bold", color = "grey40", size = base_size),
        axis.title = element_text(face = "bold", size = rel(1.2)),
        axis.title.x = element_text(margin = margin(0.5, 0, 0, 0, unit = "cm")),
        axis.title.y = element_text(margin = margin(0, 0.5, 0, 0, unit = "cm"), angle = 90),
        plot.title = element_text(face = "bold", size = rel(1.67), hjust = 0.52, margin = margin(0, 0, .2, 0, unit = 'cm')),
        plot.title.position = "plot",
        plot.subtitle = element_text(size = 11, margin = margin(0, 0, 0.2, 0, unit = "cm"), hjust = 0.5),
        plot.caption = element_text(size = 10, margin = margin(1, 0, 0, 0, unit = "cm"), hjust = 1),
        strip.text = element_text(size = rel(1.05), face = "bold"),
        strip.text.x = element_text(margin = margin(0.1, 0, 0.1, 0, "cm")),
        ...
  )
}
theme_set(theme_jacob())


# loading data in and formatting it ----
df <- read_csv('Amazon Video Game Review Data.csv ')

df2 <- df %>%
  select(customer_id, product_title, star_rating, helpful_votes, total_votes,
         verified_purchase, review_headline, review_body, review_date) %>%
  rename(CustID = customer_id, Title = product_title, Rating = star_rating,
         HelpVotes = helpful_votes, TotalVotes = total_votes, 
         VerifiedPurchase = verified_purchase, Headline = review_headline,
         Review = review_body, Date = review_date) %>%
  filter(!is.na(Date)) %>%
  mutate(review_id = row_number(),
         Date = mdy(Date))

rm(df)


## EDA ----
skim(df2)
# Top 10 Customers
df2 %>%
  select(CustID, Review) %>%
  group_by(CustID) %>%
  summarise(Count = n()) %>%
  arrange(desc(Count)) %>%
  top_n(10)
  
df2 %>%
  select(Title) %>%
  group_by(Title) %>%
  summarise(Count = n()) %>%
  arrange(desc(Count)) %>%
  top_n(10)

# building ratings table + graph
df2ratings <- df2 %>%
  count(Rating) %>%
  mutate(Pct = (n / sum(n)) * 100) %>%
  mutate('Percent of Total' = round(Pct, 2)) %>%
  select(-Pct)

ggplot(df2, aes(x = Rating)) +
  geom_histogram() +
  scale_y_continuous(labels = comma) +
  labs(title = 'Amazon Rating Counts', 
       y = NULL)
ggsave("amazonratingcounts.png", width = 15, height = 9)

# time series graph 
ggplot(df2days, aes(Date, Count, fill = impDate)) +
  geom_col(show.legend = FALSE, width = 1.2) +
  scale_y_continuous(labels = comma) +
  scale_fill_manual(values = c("#969696", "#de2d26")) +
  scale_x_date(date_breaks = "4 months", date_labels = "%b %Y") +
  annotate(geom = 'text', y = 3400, x = as.Date("2015-03-20"),
           label = "Red Lines Indicate Top 10 Most Popular Days", family = "Gill Sans MT", size = 2.25, hjust = 0.2) +
  annotate(geom = 'text', y = 3250, x = as.Date("2015-03-20"),
           label = "(The weeks following Christmas Day)", family = "Gill Sans MT", size = 2.25, hjust = 0.2) +
  labs(title = "Count of Amazon Video Game Reviews", 
       subtitle = "Data Ranging from Dec. 2012 - Aug. 2015",
       x = "Date", 
       y = "Total Reviews",
       fill = NULL)
ggsave("amazonreviews.png", width = 15, height = 9)
str(df2days)

# Forecasting next 12 months of Number of Reviews ----
df2Months <- df2 %>%
  mutate(Month = yearmonth(Date)) %>%
  group_by(Month) %>%
  summarise(Count = n())

df2days <- df2 %>%
  group_by(Date) %>%
  summarise(Count = n()) %>%
  mutate(impDate = ifelse(Count %in% tail(sort(Count), 10), 'Important Date', " "))

dfForecast <- as_tsibble(df2)
dfForecast <- df2Months %>%
  as_tsibble(index = Month)

autoplot(dfForecast)

# exponential smoothing - log cost to get rid of potential negative forecast (can't have a negative forecast for this.)
pfit <- dfForecast %>%
  model(ETS(log(Count) ~ error("M") + trend("Ad") + season("M")))
report(pfit)
abc <- augment(pfit) # fitted values & forecast horizon stuff
gg_tsresiduals(pfit) # checking residuals.  
accuracy(fc, dfForecast)
fc <- pfit %>%
  forecast(h = 12)


pfit %>%
  forecast(h = 12) %>%
  autoplot(dfForecast) +
  # scale_x_date(date_breaks = "4 months", date_labels = "%b %Y") +
  xlab("Year") + ylab("Count") +
  ggtitle("Number of Monthly Amazon Reviews for Video Game Related Products") +
  labs(subtitle = 'Exponential Smoothing Forecast (Damped Trend + Seasonality)')

ggsave('amazonexponentialsmoothing.png', width = 15, height = 9)

# ARIMA
fit_arima <- dfForecast %>%
  model(ARIMA(Count)) #(1, 0, 0) (0, 1, 0) [12]
report(fit_arima)
gg_tsresiduals(fit_arima, lag_max = 6)
augment(fit_arima) %>%
  features(.resid, ljung_box, lag = 12, dof = 3) 

fit_arima %>%
  forecast(h = 12) %>%
  autoplot(dfForecast) +
  # scale_x_date(date_breaks = "4 months", date_labels = "%b %Y") +
  xlab("Year") + ylab("Count") +
  ggtitle("Number of Monthly Amazon Reviews for Video Game Related Products") +
  labs(subtitle = 'Auto ARIMA Forecast')
ggsave('amazonautoarima.png', width = 15, height = 9)
  
# maybe do tidy text on ratings = 1 vs ratings = 5
# maybe model help ratings number of help votes vs number of words in the text????????
df2helpRatings <- df2 %>%
  filter(HelpVotes >= 200)

df2lowRatings <- df2 %>%
  filter(Rating == 1)

df2highRatings <- df2 %>%
  filter(Rating == 5)

# Tidytext stuff on Review Body ----
dfWords <- df2 %>%
  dplyr::select(Review) %>%
  unnest_tokens(word, Review)

bb <- df2 %>%
  mutate(general_rating = case_when(Rating > 3 ~ "good",
                                    TRUE ~ "bad")) %>%
  select(review_id, Review, general_rating) %>%
  unnest_tokens(word, Review) %>%
  group_by(review_id) %>%
  mutate(word_count = n())

bb %>%
  filter(word_count <= 250) %>%
  ggplot(aes(word_count, fill = general_rating)) +
  geom_histogram(bins = 50, color = 'black') %>%
  facet_wrap(~general_rating)

hh <- bb %>%
  group_by(review_id) %>%
  count(word)

# a lot of stop words in the top 15 
dfWords %>%
  count(word, sort = TRUE) %>%
  top_n(15) %>%
  mutate(word = reorder(word, n)) %>%
  ggplot(aes(x = word, y = n)) +
  geom_col() +
  xlab(NULL) +
  coord_flip() +
  labs(x = "Count",
       y = "Unique words",
       title = "Count of unique words found in tweets")

# filter out the stop words (in, of, as, the etc.)
data(stop_words)
nrow(dfWords) # 60,081,053 words before

dfWords <- dfWords %>%
  anti_join(stop_words)
nrow(dfWords) # 22,019,080 words after

# top 25words graph - Picture 1
dfWords %>%
  count(word, sort = TRUE) %>%
  top_n(25) %>%
  mutate(word = reorder(word, n)) %>%
  ggplot(aes(x = word, y = n)) +
  geom_col() +
  xlab(NULL) +
  coord_flip() +
  labs(y = "Count",
       x = "Unique Words",
       title = "Top 25 Most Popular Words")


# bigram stuff
dfbigramz <- df2 %>%
  head(5000)
# looking at paired words in tweets - bigrams
bigram <- dfbigramz %>%
  dplyr::select(Review) %>%
  unnest_tokens(bigram, Review, token = "ngrams", n = 2)

bigrams_separated <- bigram  %>%
  separate(bigram, c("word1", "word2"), sep = " ")

bigrams_filtered <- bigrams_separated %>%
  filter(!word1 %in% stop_words$word) %>%
  filter(!word2 %in% stop_words$word)

bigram_counts <- bigrams_filtered %>%
  count(word1, word2, sort = TRUE) %>%
  filter(word1 != 'br') %>%
  filter(word2 != 'br')

head(bigram_counts, 15)

bigrams_united <- bigrams_filtered %>%
  unite(bigram, word1, word2, sep = " ")

bigram_graph <- bigram_counts %>%
  filter(n >= 15) %>%
  graph_from_data_frame()

ggraph(bigram_graph, layout = "fr") +
  geom_edge_link(aes(edge_alpha = n), show.legend = FALSE,
                 arrow = grid::arrow(type = "closed", length = unit(.15, "inches")), end_cap = circle(.07, 'inches')) +
  geom_node_point(color = "lightblue", size = 5) +
  geom_node_text(aes(label = name), vjust = 1, hjust = 1) +
  labs(title = 'Popular Word Pairs in Amazon Reviews')
ggsave('amazon_bigrams2.png', width = 25, height = 12)


# filter out any noticeable sentiment word that is out of place here
bing_word_counts <- dfWords %>%
  inner_join(get_sentiments("bing")) %>%
  count(word, sentiment, sort = TRUE) %>%
  ungroup()

# Top 15 Graph for each Sentiment - Picture 3
bing_word_counts %>%
  group_by(sentiment) %>%
  top_n(15) %>%
  ungroup() %>%
  mutate(word = reorder(word, n)) %>%
  ggplot(aes(word, n, fill = sentiment)) +
  geom_col(show.legend = FALSE) +
  facet_wrap(~sentiment, scales = "free_y") +
  labs(title = "Top 15 Most Popular Words Tweeted by Each Sentiment",
       x = NULL,
       y = 'Word Count') +
  coord_flip()
ggsave('amazontop15Bing.png', width = 15, height = 9)

dfWords %>%
  count(word) %>%
  with(wordcloud(word, n, max.words = 50))

dfWords %>%
  inner_join(get_sentiments("bing")) %>%
  count(word, sentiment, sort = TRUE) %>%
  acast(word ~ sentiment, value.var = "n", fill = 0) %>%
  comparison.cloud(colors = c("red", "blue"),
                   max.words = 100)

# word Counts
word_counts <- dfWords %>%
  count(word, sort = TRUE) %>%
  mutate(total = sum(n))

# Frequency of word counts
freq_by_rank <- word_counts %>% 
  mutate(rank = row_number(), 
         `term frequency` = n/total)
freq_by_rank
head(freq_by_rank, 6) # top 6 words include ~19 of all data
top6 <- sum(head(freq_by_rank$`term frequency`, 6)) * 100
cat("Top 6 Words represent ", top6, "% of the entire dataset", sep = "" )


# Headline Text Mining ----
dfWordsHeadline <- df2 %>%
  dplyr::select(Headline) %>%
  unnest_tokens(word, Headline)

# a lot of stop words in the top 15 
dfWordsHeadline %>%
  count(word, sort = TRUE) %>%
  top_n(15) %>%
  mutate(word = reorder(word, n)) %>%
  ggplot(aes(x = word, y = n)) +
  geom_col() +
  xlab(NULL) +
  coord_flip() +
  labs(x = "Count",
       y = "Unique words",
       title = "Count of unique words found in tweets")

# filter out the stop words (in, of, as, the etc.)
data(stop_words)
nrow(dfWordsHeadline) # 4,160,561 words before

dfWordsHeadline <- dfWordsHeadline %>%
  anti_join(stop_words)
nrow(dfWordsHeadline) # 1,924,161 words after

# top 25words graph - Picture 1
dfWordsHeadline %>%
  count(word, sort = TRUE) %>%
  top_n(25) %>%
  mutate(word = reorder(word, n)) %>%
  ggplot(aes(x = word, y = n)) +
  geom_col() +
  xlab(NULL) +
  coord_flip() +
  labs(y = "Count",
       x = "Unique words",
       title = "Top 25 Most Popular Words")


# filter out any noticeable sentiment word that is out of place here
bing_word_counts_headline <- dfWordsHeadline %>%
  inner_join(get_sentiments("bing")) %>%
  count(word, sentiment, sort = TRUE) %>%
  ungroup()

# Top 15 Graph for each Sentiment - Picture 3
bing_word_counts_headline %>%
  group_by(sentiment) %>%
  top_n(15) %>%
  ungroup() %>%
  mutate(word = reorder(word, n)) %>%
  ggplot(aes(word, n, fill = sentiment)) +
  geom_col(show.legend = FALSE) +
  facet_wrap(~sentiment, scales = "free_y") +
  labs(title = "Top 15 Most Popular Words Tweeted by Each Sentiment",
       x = NULL,
       y = 'Count')  +
  coord_flip()

# all 3 sentiments at once
afinn <- dfWordsHeadline %>%
  inner_join(get_sentiments('afinn')) %>%
  group_by(value) %>%
  summarise(Count = n()) %>%
  mutate(method = 'AFINN')

bing_and_nrc <- bind_rows(dfWordsHeadline %>% 
                            inner_join(get_sentiments("bing")) %>%
                            mutate(method = "Bing et al."),
                          dfWordsHeadline %>% 
                            inner_join(get_sentiments("nrc") %>% 
                                         filter(sentiment %in% c("positive", 
                                                                 "negative"))) %>%
                            mutate(method = "NRC")) %>%
  count(method, sentiment) %>%
  spread(sentiment, n, fill = 0) %>%
  mutate(sentiment = positive - negative)

badwords <- dfWordsHeadline %>%
  inner_join(get_sentiments('afinn')) %>%
  filter(value == -5) %>%
  group_by(word) %>%
  summarise(Count = n())

afinnbad <- dfWordsHeadline %>%
  inner_join(get_sentiments('afinn')) %>%
  filter(value < 0)

afinngood <- dfWordsHeadline %>%
  inner_join(get_sentiments('afinn')) %>%
  filter(value > 0)

goodwords <- dfWordsHeadline %>%
  inner_join(get_sentiments('afinn')) %>%
  filter(value == 5) %>%
  group_by(word) %>%
  summarise(Count = n())


# top 25words graph - Picture 1
afinnbad %>%
  count(word, sort = TRUE) %>%
  top_n(25) %>%
  mutate(word = reorder(word, n)) %>%
  ggplot(aes(x = word, y = n)) +
  geom_col() +
  xlab(NULL) +
  coord_flip() +
  labs(y = "Count",
       x = "Unique Words",
       title = "Top 25 Most Popular Words in Review Headlines",
       subtitle = 'AFINN Sentiment Analysis - Negative Words')
ggsave('amazonnegativewords.png', width = 15, height = 9)

# top 25words graph - Picture 1
afinngood %>%
  count(word, sort = TRUE) %>%
  top_n(25) %>%
  mutate(word = reorder(word, n)) %>%
  ggplot(aes(x = word, y = n)) +
  geom_col() +
  xlab(NULL) +
  coord_flip() +
  labs(y = "Count",
       x = "Unique Words",
       title = "Top 25 Most Popular Words in Review Headlines",
       subtitle = 'AFINN Sentiment Analysis - Positive Words')
ggsave('amazonpositivewords.png', width = 15, height = 9)

##### ggraph and igraph try
df2_rating_bigrams <- df2 %>%
  mutate(id = row_number())

df2_rating_ids <- df2_rating_bigrams %>%
  select(id, Rating)

df_tokenized <- df2_rating_bigrams %>%
  unnest_tokens(word, Review) %>%
  anti_join(stop_words, by = 'word') %>%
  inner_join(df2_rating_ids, by = 'id')
head(df_tokenized)

df_tokenized_clean <- df_tokenized %>%
  group_by(word) %>%
  summarize(number = n(),
            reviews = n_distinct(id),
            avg_rating = mean(Rating.y)) %>%
  arrange(desc(number))

df_words_filtered <- df_tokenized_clean %>%
  filter(reviews > 5000, number > 5000)

word_cors <- df_tokenized %>%
  semi_join(df_words_filtered, by = 'word') %>%
  distinct(id, word) %>%
  pairwise_cor(word, id, sort = TRUE)

filtered_cors <- word_cors %>%
  head(125)

nodes <- df_words_filtered %>%
  filter(word %in% filtered_cors$item1 | word %in% filtered_cors$item2)
set.seed(2020)
word_cors %>%
  head(125) %>%
  graph_from_data_frame(vertices = nodes) %>%
  ggraph() +
  geom_edge_link() +
  geom_node_point(aes(size = reviews * 6.0)) +
  geom_node_point(aes(size = reviews, color = avg_rating)) +
  geom_node_text(aes(label = name), repel = TRUE) +
  scale_color_gradient2(low = 'red', high = 'blue', midpoint = 4) +
  scale_size_continuous(label = comma) +
  theme_void() +
  labs(color = 'Average Rating',
       size = 'Number of Reviews',
       title = 'Network of Most Popular Word Pairs in Video Game Product Amazon Reviews',
       subtitle = 'Data from Dec. 2012 - Aug. 2015')
ggsave('amazonggraphnew.png', width = 30, height = 9)

#############################################################################################

# tidymodels ----
set.seed(3031)
reviews_parsed <- df2 %>%
  mutate(review_id = row_number(),
         general_rating = case_when(Rating > 3 ~ "good",
                                    TRUE ~ "bad")) %>%
  filter(Rating != 3) %>% # removing average ratings - it noticeably increases prediction accuracy 
  group_by(general_rating) %>%
  slice_sample(n = 5000) %>% # 5000 reviews each 
  ungroup()

reviews_parsed %>%
  group_by(general_rating) %>%
  count()

# preprocessing
set.seed(123)
review_split <- initial_split(reviews_parsed, strata = general_rating)
review_train <- training(review_split)
review_test <- testing(review_split)

review_rec <- recipe(general_rating ~ Review, data = review_train) %>%
  step_tokenize(Review) %>%
  step_stopwords(Review) %>%
  step_tokenfilter(Review, max_tokens = 500) %>%
  step_tfidf(Review) %>%
  step_normalize(all_predictors(), -all_outcomes())

# model building
log_spec <- logistic_reg() %>%
  set_engine(engine = "glm") %>%
  set_mode('classification')

rf_spec <- rand_forest() %>% 
  set_engine("ranger", importance = "impurity") %>% 
  set_mode("classification")

xgb_spec <- boost_tree() %>% 
  set_engine("xgboost") %>% 
  set_mode("classification") 

knn_spec <- nearest_neighbor(neighbors = 8) %>% # can adjust the number of neighbors 
  set_engine("kknn") %>% 
  set_mode("classification") 

log_wf <- workflow() %>%
  add_recipe(review_rec) %>%
  add_model(log_spec)


rf_wf <- workflow() %>%
  add_recipe(review_rec) %>% 
  add_model(rf_spec) 

xgb_wf <- workflow() %>%
  add_recipe(review_rec) %>% 
  add_model(xgb_spec)

knn_wf <- workflow() %>%
  add_recipe(review_rec) %>% 
  add_model(knn_spec)

set.seed(100)
cv_folds <- vfold_cv(review_train, v = 5, strata = general_rating) 

get_model <- function(x) {
  pull_workflow_fit(x) %>% tidy()
}

log_res <- log_wf %>% 
  fit_resamples(
    resamples = cv_folds, 
    metrics = metric_set(
      recall, precision, f_meas, 
      accuracy, kap,
      roc_auc, sens, spec),
    control = control_resamples(
      save_pred = TRUE,
      extract = get_model))

rf_res <- rf_wf %>% 
  fit_resamples(
    resamples = cv_folds, 
    metrics = metric_set(
      recall, precision, f_meas, 
      accuracy, kap,
      roc_auc, sens, spec),
    control = control_resamples(save_pred = TRUE)) 

xgb_res <- xgb_wf %>% 
  fit_resamples(
    resamples = cv_folds, 
    metrics = metric_set(
      recall, precision, f_meas, 
      accuracy, kap,
      roc_auc, sens, spec),
    control = control_resamples(save_pred = TRUE)) 

knn_res <- knn_wf %>% 
  fit_resamples(
    resamples = cv_folds, 
    metrics = metric_set(
      recall, precision, f_meas, 
      accuracy, kap,
      roc_auc, sens, spec),
    control = control_resamples(save_pred = TRUE)) 

## collecting results
log_metrics <- 
  log_res %>% 
  collect_metrics(summarise = TRUE) %>%
  mutate(model = "Logistic Regression") # add the name of the model to every row

rf_metrics <- 
  rf_res %>% 
  collect_metrics(summarise = TRUE) %>%
  mutate(model = "Random Forest")

xgb_metrics <- 
  xgb_res %>% 
  collect_metrics(summarise = TRUE) %>%
  mutate(model = "XGBoost")

knn_metrics <- 
  knn_res %>% 
  collect_metrics(summarise = TRUE) %>%
  mutate(model = "Knn")

model_compare <- bind_rows(
  log_metrics,
  rf_metrics,
  xgb_metrics,
  knn_metrics) 

## conf matrixes
log_res %>% 
  collect_predictions() %>%
  conf_mat(general_rating, .pred_class) 

rf_res %>% 
  collect_predictions() %>%
  conf_mat(general_rating, .pred_class) 

xgb_res %>% 
  collect_predictions() %>%
  conf_mat(general_rating, .pred_class) 

knn_res %>% 
  collect_predictions() %>%
  conf_mat(general_rating, .pred_class) 

log_res %>% 
  collect_predictions() %>%
  roc_curve(general_rating, .pred_bad) %>% 
  autoplot()

log_roc <- log_res %>% 
  collect_predictions() %>%
  mutate(Model = 'Logistic Regression')

rf_roc <- rf_res %>% 
  collect_predictions() %>%
  mutate(Model = 'Random Forest')

xgb_roc <- xgb_res %>% 
  collect_predictions() %>%
  mutate(Model = 'XGBoost')

knn_roc <- knn_res %>% 
  collect_predictions() %>%
  mutate(Model = 'KNN')

all <- log_roc %>%
  rbind(rf_roc) %>%
  rbind(xgb_roc) %>%
  rbind(knn_roc)

all %>% 
  group_by(Model) %>%
  roc_curve(general_rating, .pred_bad) %>% 
  autoplot() +
  labs(title = 'Model Performance Comparison')
ggsave('amazon_roc.png', width = 15, height = 9)


# change data structure
model_comp <- model_compare %>% 
  select(model, .metric, mean, std_err) %>% 
  pivot_wider(names_from = .metric, values_from = c(mean, std_err)) 

model_comp %>% 
  arrange(mean_f_meas) %>% 
  mutate(model = fct_reorder(model, mean_f_meas)) %>% # order results
  ggplot(aes(model, mean_f_meas, fill=model)) +
  geom_col(color = 'black') +
  coord_flip() +
  geom_text(size = 3, aes(label = round(mean_f_meas, 2), y = mean_f_meas + 0.08), vjust = 1) +
  labs(x = NULL,
       y = 'F1-Score',
       title = 'Best Models') +
  theme(legend.position = 'none')

model_comp %>% 
  arrange(mean_roc_auc) %>% 
  mutate(model = fct_reorder(model, mean_roc_auc)) %>%
  ggplot(aes(model, mean_roc_auc, fill=model)) +
  geom_col(color = 'black') +
  coord_flip() +
  geom_text(size = 3, aes(label = round(mean_roc_auc, 2), y = mean_roc_auc + 0.08), vjust = 1) +
  labs(x = NULL,
       y = 'Mean AUC',
       title = 'Best Models') +
  theme(legend.position = 'none')

model_comp %>% slice_max(mean_f_meas)

last_fit_log <- last_fit(log_wf, 
                        split = review_split,
                        metrics = metric_set(
                          recall, precision, f_meas, 
                          accuracy, kap,
                          roc_auc, sens, spec))

last_fit_rf <- last_fit(rf_wf, 
                         split = review_split,
                         metrics = metric_set(
                           recall, precision, f_meas, 
                           accuracy, kap,
                           roc_auc, sens, spec))

last_fit_log %>% 
  collect_metrics() %>%# .842 f score
  select(-.estimator, -.config) %>%
  mutate(.metric = fct_reorder(.metric, .estimate)) %>%
  ggplot(aes(.estimate, .metric, label = round(.estimate, 3))) +
  geom_col() +
  geom_text(aes(.estimate, .metric), hjust = -0.25) +
  labs(x = 'Coefficient Value',
       y = NULL,
       title = 'Final Performance Metrics on Testing Set')
ggsave('amazon_metrics.png', width = 15, height = 9)

last_fit_rf %>%
  collect_metrics() # .832 f score

last_fit_log %>%
  collect_predictions() %>%
  conf_mat(general_rating, .pred_class) %>%
  autoplot(type = "heatmap")
ggsave('amazon_heatmap.png', height = 5, width = 9)

last_fit_rf %>%
  collect_predictions() %>%
  conf_mat(general_rating, .pred_class) %>%  
  autoplot(type = "heatmap") # heat map confirms logistic regression a tad better.
  
last_fit_log %>%
  pluck(".workflow", 1) %>% 
  pull_workflow_fit() %>%
  vi(lambda = best_auc$penalty) %>%
  group_by(Sign) %>%
  top_n(30, wt = abs(Importance)) %>%
  ungroup() %>%
  mutate(
    Importance = abs(Importance),
    Variable = str_remove(Variable, "tfidf_Review_"),
    Variable = fct_reorder(Variable, Importance)
  ) %>%
  ggplot(aes(x = Importance, y = Variable, fill = Sign)) +
  geom_col(show.legend = FALSE) +
  facet_wrap(~Sign, scales = "free_y") +
  labs(y = NULL,
       x = 'Importance (TF-IDF)',
       title = 'Which Words lead to Positive or Negative Reviews?')
ggsave('amazon_importance.png', width = 15, height = 9)
