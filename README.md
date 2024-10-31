# Overview

# The Questions

Below are the questions I want to answer in my project:

# Tools I Used

For my deep dive into the top 1000 tv series on IMDB, I harnessed the power of several key tools:

## Import & Clean Up Data

I start by importing necessary libraries and loading the dataset.

```
# Importing Libraries
import pandas as pd
import seaborn as sns
import matplotlib.pyplot as plt

#Loading Data
dataset = pd.read_csv(r'data/IMDB_Top_1000_TV_shows.csv')
```

# The Analysis
Each Jupyter notebook for this project aimed at investigating specific aspects of the data. Here’s how I approached each question:

## 1. What are tv-shows trends quality-wise?

### Visualize Data

```python
# Plotting the trends
df.boxplot(column='averageRating', by='releaseYear', figsize=(32, 10))

# Show the plot
plt.show()
```

### Results
![Visualization Of Average Rating of 1000 TV shows by The Year](images/1_1_the_annual_distribution_of_ratings.png)

### Insights:

- **Higher Ratings in Earlier Years:** In the 1950s and 1960s, ratings generally trend higher, with medians closer to or above 8.5. This suggests that shows from these earlier decades received higher average ratings compared to later years.

- **Increased Rating Variability:** Starting from the 1970s, there’s a notable increase in variability, with larger interquartile ranges and more outliers, indicating a mix of high and low ratings. This suggests that shows from these periods vary widely in audience reception.

- **Steady Ratings in Recent Years:** From around 2010 onwards, the ratings show less variation, with medians consistently between 7.5 and 8. This could imply that recent TV shows maintain a more consistent rating level but rarely reach extreme highs or lows.

### Visualize Data

```python
# Group by 'releaseYear' and calculate the percentiles
rating_trends = df.groupby('releaseYear')['averageRating'].agg(
    lowest='min',
    Q1='quantile',
    median='median',
    Q3='quantile',
    highest='max'
).reset_index()

# Calculate 25% (Q1) and 75% (Q3)
rating_trends['Q1'] = df.groupby('releaseYear')['averageRating'].quantile(0.25).values
rating_trends['Q3'] = df.groupby('releaseYear')['averageRating'].quantile(0.75).values

# Plotting the trends
plt.figure(figsize=(25, 8))
plt.plot(rating_trends['releaseYear'], rating_trends['lowest'], label='Lowest', marker='o')
plt.plot(rating_trends['releaseYear'], rating_trends['Q1'], label='25th Percentile (Q1)', marker='o')
plt.plot(rating_trends['releaseYear'], rating_trends['median'], label='Median', marker='o')
plt.plot(rating_trends['releaseYear'], rating_trends['Q3'], label='75th Percentile (Q3)', marker='o')
plt.plot(rating_trends['releaseYear'], rating_trends['highest'], label='Highest', marker='o')

# Adjust the x-axis ticks to show every 5 years
plt.xticks(ticks=range(min(rating_trends['releaseYear'] - 1), max(rating_trends['releaseYear']) + 2, 5))

# Show the plot
plt.show()
```

### Results
![Visualization Of Average Rating Trend Over the Years](images/1_2_tracking_changes_in_key_percentiles_over_time.png)

### Insights:

- **Stabilization in Recent Years:** The median, along with the 25th and 75th percentiles, shows less fluctuation from around 2005 onwards, indicating that average ratings for TV shows have become more consistent in recent years. This could suggest a standardization in TV show quality or rating criteria.

- **Peaks in Highest Ratings:** The line for the highest ratings (purple) has several spikes, especially noticeable in the 1950s, late 1970s, and early 2010s. These peaks likely correspond to exceptionally well-received shows, hinting at notable classics or popular modern hits. Annotating these peaks could add value by pinpointing which shows made such an impact.

- **Wider Spread in the 1990s:** The distance between the highest and lowest ratings is more pronounced in the 1990s, indicating a broader range in the quality or reception of shows from that period. This may reflect a more experimental era in television, with some shows achieving high acclaim and others receiving lower scores.

*Even through these two plots might seem similar, one shows the annual distribution of ratings (Box Plot) and the other tracks changes in key percentiles over time (Line Plot).*



## 2. How does top shows ratings are distributed?

### Visualize Data
```python
# Distribution of Average Ratings
df['averageRating'].plot(kind='hist', bins=17, edgecolor='black')

# Show the plot
plt.show()
```
### Results
![Visualization Of Distribution of Average Ratings](images/2_distribution_of_average_ratings.png)

### Insights:

- High Overall Ratings: Given that this dataset represents the top 1000 shows on IMDb, the right-skewed distribution (with most ratings between 8.0 and 8.6) suggests that highly rated shows dominate, reflecting consistent quality among the best-rated series.

- Peak Popularity Around 8.2: The most common rating among the top shows is around 8.2, indicating that a large portion of these highly regarded series falls in this range. This rating can be seen as the "benchmark" for quality in the top 1000 shows.

- Exclusive Few Above 9.0: Only a select few shows achieve ratings above 9.0, underscoring their exceptional status even among the best. These outliers likely represent some of the most iconic and universally acclaimed shows on IMDb.

## 3.

### Visualize Data
```python
plt.figure(figsize=(25, 8))
plt.xlim(1950, 2025)
df.groupby('releaseYear')['averageRating'].mean().plot(kind='line', marker='.')

# Adjust the x-axis ticks to show every 5 years
plt.xticks(ticks=range(min(rating_trends['releaseYear'] - 1), max(rating_trends['releaseYear']) + 2, 5))

# Show the plot
plt.show()
```
### Results
![Visualization Of The Trend of Average Ratings Over The Years](images/3_trend_of_average_ratings_over_the_years.png)

## 4.

### Visualize Data
```python
from matplotlib.ticker import FuncFormatter

# Create a function to format the x-axis labels
def millions_formatter(x, _):
    return f'{x / 1_000_000:.1f}M' if x >= 500_000 else str(int(x))  # Format as '1.5M', '2.0M', etc.

# Create a scatter plot with enhancements
plt.figure(figsize=(25, 8))

# Plot marker size based on numVotes
scatter = plt.scatter(df['numVotes'], df['averageRating'], 
                      alpha=0.6, c=df['averageRating'], cmap='viridis', 
                      s=(df['numVotes'] / df['numVotes'].max()) * 100 + 10, edgecolor='k')

# Define criteria for outliers; you can adjust these as needed
outlier_conditions = (
    (df['averageRating'] >= 9.0) & (df['numVotes'] >= 2000000)  # Example condition for high-rated shows
)

# Select the outliers based on defined conditions
outliers = df[outlier_conditions]

# Annotate the outliers
for index, row in outliers.iterrows():
    plt.annotate(row['title'], 
                 xy=(row['numVotes'], row['averageRating']), 
                 xytext=(row['numVotes'] - 330000, row['averageRating'] - 0.2),  # Adjusted for reverse arrow direction
                 arrowprops=dict(facecolor='y', shrink=0.11, connectionstyle='arc3,rad=0.2'),  # Use angle3 with specified angles
                 fontsize=12)

# Set limiter
plt.xlim(0, 2_500_000)
    
# Apply the millions formatter to the x-axis
plt.gca().xaxis.set_major_formatter(FuncFormatter(millions_formatter))

# Show the plot
plt.show()
```
### Results
![Visualization Of Number of Votes vs. Average Rating](images/4_number_of_votes_vs_average_rating.png)

## 5.

### Visualize Data
```python
# Define top 10 shows by rating
top_rat = df.nlargest(10, 'averageRating')

# Create a horizontal bar plot with seaborn
plt.figure(figsize=(12, 8))
sns.barplot(data=top_rat, x='averageRating', y='title', hue='title',  palette='viridis')

# Show the plot
plt.show()
```
### Results
![Visualization Of Top 10 Shows By Rating](images/5_top_shows_by_rating.png)

## 6.

### Visualize Data
```python
# Define top 10 shows by popularity
top_num = df.nlargest(10, 'numVotes')

# Create a horizontal bar plot with seaborn
plt.figure(figsize=(12, 8))
sns.barplot(data=top_num, x='numVotes', y='title', hue='title',  palette='viridis')

# Set limiter
plt.xlim(0, 2_500_000)

# Apply the millions formatter to the x-axis
plt.gca().xaxis.set_major_formatter(FuncFormatter(millions_formatter))

# Show the plot
plt.show()
```
### Results
![Visualization Of Top 10 Genres By Rating](images/6_top_shows_by_popularity.png)
## 7.

### Visualize Data
```python

```
### Results
![Visualization Of Top 10 Genres By Rating](images/7_average_ratings_by_genre.png)
## 8.

### Visualize Data
```python

```
### Results