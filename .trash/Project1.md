1a.

| count | 77648.000000 |
| ----- | ------------ |
| mean  | 37.063414    |
| std   | 21.213698    |
| min   | 14.000000    |
| 10%   | 18.000000    |
| 50%   | 29.000000    |
| max   | 120.000000   |
1b.
	1. Most users  fall into the 15-45 range, as people gets older they tend to not be on the platform
	2. People are inputting false ages, maybe not to directly skew the data but these points need to be kept in mind when doing summary statistics or making inferences.

1c. 
	1. Male users outweigh female users of this application.  
	2. User counts of sex start to even out at the age of ~50.

1d.
	The first month of the year & first day of the month appear most often. These do not represent the true distribution of the birthdays, likely because the age input is defaulted to this day and users who are trying to skip this part of the data input process use this date for facility.

2a.
	`m_data = FB[FB['gender'] == 'male']`
	gender male 45984 female 31664 Name: count, dtype: int64 (31664, 15) (45984, 15)

2b. 
	Female users have more data on average; for every 1 friend a male has, a female has, on average, 1.48 friends.

2c. 
	This data point stays about the same for both sexes, but females initiate more, on average.

2d. 
	This code calls the new variables f/m_data.loc to be comprised of the number of friendships initiated divided by the total friend count. Then on the printing line, calls for the mean of these new variables.  
	The number of friends initiated as a percentage of number of friends for females are 56% for females, and 64% for males.  
	My interpretation of this is that males are more likely to initiate friendships of people they know, and females are slightly less likely to initiate the friendships.

2e.
	For female users, younger individuals tend to have slightly more friends, but the effect is small.  
	For male users, age does not explain friend count at all.  
	Overall, age is not a strong predictor of friend count for either gender, though it has a marginal effect for females.

2f. 
1. . The slope is negative of the correlation line for females, indicating that as females grow older, they have less friends on the platform.  
2. In the male scatter plot, the outliers of age in the higher range skew the results and make it so that there is no reasonable indication of friendships based on age.**

2g. 
1. Correlation does not imply causation, even if two variables are correlated, it does not mean that one causes the other. External variables or other factors may drive the relationship.
2. Correlation captures only linear relationships, if the relationship is nonlinear, the correlation coefficiant may underestimate or entirely miss the relationship.

2f. 
	In the scatter plot for male users, point clouds may suggest a slight negative trend, but the linear fit is slightly positive. This contradiction can arise because outliers or uneven distribution of data influence the fitted line and corerlation differently than what we may percieve.
	A scatter plot can be visually misleading when outliers or uneven clusters dominate the appearance of the trend. Similarly a single correlation coefficient oversimplifies the data.
	To use both correlation and scatter plots together, we should remove or analyze outliers separately, calculate nonlinear correlations, or do regresssion models to better understand the data.

3a. 
	No, as they are all on the same bar. It shows that the overall distribution is between 0 and ~18,000 likes.

3b. 
	Most people only have like counts of 1- ~25, then the dropoff is steep. This implies there is a lot more 1-off content compared to viral content.

3c. 
	We should use the log scale because it does a better job of capturing the ratios of like count compared to post count.

3d. 
	The mean will be higher than the median because of the right skew.

3e. 
	Females have a higher average number of likes received since their histogram is shifted further to the right.
	Females also have a higher median number of likes since their distribution is less concentrated at the low end and is more balanced.

3g. 
```
stats_by_gender = FB.groupby('gender').agg(

    mean_age = ('age', 'mean'),
    mean_likes = ('likes', 'mean'),
    mean_www_likes = ('www_likes', 'mean'),
    mean_mobile_likes = ('mobile_likes', 'mean'),
    mean_likes_received = ('likes_received', 'mean'),
    mean_mobile_likes_received = ('mobile_likes_received', 'mean'),
    mean_www_likes_received = ('www_likes_received', 'mean')

).reset_index()

stats_by_gender
```
	The summary statistics reveal that female users are slightly older on average (39.3 vs. 35.5 years) and receive significantly more likes than male users. Females have an average of 262 likes, compared to only 82 likes for males. This pattern holds across platforms: females have higher averages for both web likes and mobile likes, as well as likes received. In particular, females receive nearly 245 likes on average, compared to just 64 for males. Overall, females not only tend to be slightly older but also show much higher engagement levels in terms of likes given and received, especially on mobile.

