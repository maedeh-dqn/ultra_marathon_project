# Ultra Marathon Project -- Pandas Analysis 

## Project Overview

**Project Name**: Ultra Marathon Project -- Pandas Analysis 
**Level**: Beginner to Intermediate 
**Database**: `Kaggle`

This project is designed to demonstrate Pandas and Python skills and techniques typically used by data analysts to explore, clean, and analyze a niche in a big dataset. The project involves downloading an ultra marathon database registered between 1798 and 2022, performing exploratory data analysis (EDA), and answering specific analytical questions through Pandas and Python queries. This project is ideal for those who are starting their journey in data analysis and want to build a solid foundation in Pandas.

## Objectives

1. **Downloading a Big dataset**: Download the ultra marathon dataset from Kaggle.
2. **Data Cleaning**: Identify and remove any records with missing or null values.
3. **Exploratory Data Analysis (EDA)**: Perform basic exploratory data analysis to understand the dataset.
4. **Further Analysis and Data Manipulations**: Use Pandas to answer specific analytical questions and derive insights from the marathon data. Utilize Seaborn to create data visualizations for some questions.

## Project Structure ##

### 1. Download the dataset, and Install required libraries

- **Installing Libraries**: The project starts by installing the libraries like Pandas, seaborn, and re.
- **Uploading the dataset on Jupyter Notebook**.
```python
#Import required libraries
import pandas as pd
import seaborn as sns
import re

#Load the dataset
df = pd.read_csv('TWO_CENTURIES_OF_UM_RACES.csv')

#See the data that has been imported
df.head(10)
```

### 2. Data Exploration & Cleaning

- **Check the dataframe**: Check the shape, data types, and general information.
- **Check the duplicates**: Check fro the duplicate values in the data frame.
- **Null Value Check**: Check for any null values in the dataset to decide about your dealing method in further analysis.
```python
#Check dataframe shape
df.shape

#Check data types
df.dtypes

#Check general information for this data frame
df.info()

#Check for null values
df.isnull()

#Data Cleaning Phase
As the dataset was massive, I decided to analyze a small portion of it. Consequently, I chose one country, the USA, and conducted the analysis based on related data. I selected my dataframe based on these criteria:
1. Only want the USA races
2. Races with 50mi or 50 km length
3. The chosen year is 2020

df[df['Event distance/length'] == '50km']
df[df['Event distance/length'] == '50mi']
df[df['Event distance/length'] == '50k']

#Combine 50k/50mi with isin
df[df['Event distance/length'].isin(['50mi','50km'])]

#Combine the filters above with my chosen year
df[(df['Event distance/length'].isin(['50mi','50km'])) & (df['Year of event'] == 2020)]

#Find all the events that took place in the USA
df[df['Event name'].str.split('(').str.get(1).str.split(')').str.get(0) == 'USA']

#Combine all the filters together for the USA
USA_df = df[(df['Event distance/length'].isin(['50mi','50km'])) & (df['Year of event'] == 2020) & (df['Event name'].str.split('(').str.get(1).str.split(')').str.get(0) == 'USA')]

#Remove USA from event name
USA_df['Event name'] = USA_df['Event name'].str.split('(').str.get(0)

#Clean up athlete age and give it a proper format (i.e 23)
#Firstly, we should change float datatype to int to be able to do the calculations.
#As there are some null values in [athlete year of birth] column, we cannot change the datatype from float to integer. So, follow these steps:
USA_df['Athlete year of birth'] = USA_df['Athlete year of birth'].fillna(0)

#In this step we can change the datatype using astype and save it to the current col.
USA_df['Athlete year of birth'] = USA_df['Athlete year of birth'].astype(int)

#Now we can do the age calculations:
USA_df['Athlete age category'] = USA_df['Year of event'] - USA_df['Athlete year of birth'].astype(int)

#Finally, we rename Athlete age category col.
USA_df = USA_df.rename(columns={'Athlete age category':'Athlete age'})

#Remove "h" from athlete performance
#Change the datatype to str
USA_df['Athlete performance'] = USA_df['Athlete performance'].str.split(' ').str.get(0)

#Drop redundant cols: Athlete club, Athlete year of birth
USA_df = USA_df.drop(['Athlete club', 'Athlete year of birth'], axis=1)

#Check and delete null values
USA_df.isna()
USA_df = USA_df.dropna()

#Check for duplicate values
USA_df[USA_df.duplicated() == True]

#Reset index
USA_df.reset_index(drop=True)

#Modify datatypes
USA_df['Athlete average speed'] = USA_df['Athlete average speed'].astype(float)

#Rename cols for USA dataset
USA_df = USA_df.rename(columns={'Year of event':'year',
                                'Event dates':'race_day',
                                'Event name':'race_name',
                               'Event distance/length':'race_length',
                               'Event number of finishers':'number_of_finishers',
                               'Athlete performance':'athlete_performance',
                               'Athlete country':'athlete_country',
                               'Athlete gender':'athlete_gender',
                               'Athlete age':'athlete_age',
                               'Athlete average speed':'athlete_average_speed',
                               'Athlete ID':'athlete_id'})

#Reorder cols:
USA_df = USA_df[['athlete_id', 'athlete_gender', 'athlete_age', 'athlete_country',
                 'athlete_performance', 'athlete_average_speed', 'number_of_finishers', 'race_length',
                 'race_name', 'race_day', 'year']]
```

### 3. Data Analysis

The following Pandas queries were developed to answer specific business questions.

1. **Find top 10 atheletes with highest average speed in 2020 in the USA.**
```python
USA_df.sort_values('athlete_average_speed', ascending=False).head(10)
```

2. **How many male and female participants were there in the 2020 marathon in the USA? provide the numbers separately?**
```python
USA_df['athlete_gender'].value_counts()
```

3. **What percent of participants were male / female?**
```python
#The result of the code below is a fraction in range (0, 1]. We multiply by 100 here in order to get the %.
USA_df['athlete_gender'].value_counts(normalize=True) * 100
```

4. **What was the average age of the participants?**.
```python
#I suddenly recognized that I have 233 wrong data in age col. these rows were 0 at first and in my age calculations they turned to 2020. so I deleted them with the code below. 
USA_df = USA_df[USA_df.athlete_age != 2020]
USA_df.athlete_age.mean()
```

5. **Calculate the average age for each gender.**
```python
#Because of the gender col dtype (object), I can't easily calculate the age usin group by. So, at first I have to create two new dataframes (one for males and one for females), and then I can calculate the mean age for each group.
male_df = USA_df[USA_df['athlete_gender'] == 'M']
male_df.athlete_age.mean()
female_df = USA_df[USA_df['athlete_gender'] == 'F']
female_df.athlete_age.mean()
```

6. **How many unique events are there in the USA dataframe?**
```python
USA_df['race_name'].value_counts().reset_index()
```

7. **Identify top 20 (unique) events with highest number of finishers.**
```python
uniq_event_and_finisher = USA_df[['race_name', 'number_of_finishers']].value_counts()
uniq_event_and_finisher = uniq_event_and_finisher.sort_values(ascending=False).head(20).reset_index()
uniq_event_and_finisher.drop('count', axis=1)
```

8. **How many times has each athlete participated in these marathons?**
```python
USA_df.value_counts('athlete_id').reset_index()
```

9. **Which countries had the highest number of participants in  these marathons? Select top 10.**
```python
USA_df.value_counts('athlete_country').reset_index().head(10)
```

10. **Calcualte differences in average speed for the 50mi, 50km male to female.**
```python
USA_df.groupby(['race_length', 'athlete_gender'])['athlete_average_speed'].mean()
```

11. **What age groups are the BEST in the 50mi race (keep groups which have more than 19 memebers). Select top 20.**
```python
USA_df.query('race_length == "50mi"').groupby(['athlete_age'])['athlete_average_speed'].agg(['mean', 'count']).sort_values(by='mean', ascending=False).query('count>19').head(20)
```

12. **What age groups are the WORST in the 50mi race (keep groups which have more than 19 memebers). Select bottom 15.**
```python
USA_df.query('race_length == "50mi"').groupby(['athlete_age'])['athlete_average_speed'].agg(['mean', 'count']).sort_values(by='mean', ascending=True).query('count>19').head(15
```

13. **In which season was each competition held?**
```python
#In the first step, extract the month of the competitions from the race_day column.
race_season = USA_df['race_day'].str.split('.').str.get(1).astype(int)

#In the second step, add this new column to the usa dataframe.
idx = 9
USA_df.insert(loc=idx, column='race_season', value=race_season)

#In the third step, we define a dictionary to map the integer 1-12 to their respective season.
season_dict = {
    '1':'Winter',
    '2':'Winter',
    '3':'Spring',
    '4':'Spring',
    '5':'Spring',
    '6':'Summer',
    '7':'Summer',
    '8':'Summer',
    '9':'Autumn',
    '10':'Autumn',
    '11':'Autumn',
    '12':'Winter'
}
#Changing the season column data type to string
USA_df.race_season = USA_df.race_season.astype(str)

#Map the values 1-12 to their respective season
USA_df.race_season = USA_df.race_season.map(season_dict)
```

14. **Calculate the mean for athlete average speed and the frequency of athlete in each season.**
```python
USA_df.groupby('race_season')['athlete_average_speed'].agg(['mean', 'count']).sort_values('mean', ascending=False)
```

15. **Calculate the mean for athlete average speed and the frequency of athlete in each season. Select 50 milers only.**
```python
USA_df.query('race_length == "50mi"').groupby('race_season')['athlete_average_speed'].agg(['mean', 'count']).sort_values('mean', ascending=False)
```

### 4. Data Visualization

16. **Create a bar chart for question 9: Which countries had the highest number of participants in  these marathons? Select top 10.**
```python
q_9_plot = sns.countplot(x='athlete_country',data=USA_df,order=pd.value_counts(USA_df['athlete_country']).iloc[:10].index)
q_9_plot.bar_label(q_9_plot.containers[0])
```

17. **Create a bar plot for athlete countries based on their gender.**
```python
q_9_plot = sns.countplot(x='athlete_country',hue='athlete_gender', data=USA_df,order=pd.value_counts(USA_df['athlete_country']).iloc[:10].index)

#By executing the following code, Seaborn can consider labels for both male and female bars in the chart.
for i in q_9_plot.containers:
    q_9_plot.bar_label(i,)
```

18. **Create a distribution plot based on race length and athletes average speed.**
```python
sns.displot(USA_df[USA_df['race_length'] == '50mi']['athlete_average_speed'])
```

19. **Create a violin plot based on race length, athletes average speed, and their gender**
```python
sns.violinplot(data=USA_df, x='race_length', y='athlete_average_speed', hue='athlete_gender', split=True, inner='quart')

-- END OF PROJECT --
```

### 5. Findings

- **Athlete Demographics**: The dataset includes athletes from various age groups, and countries, with different average speeds throughout the races held in 2020.
- **Leading countries in competitions**: Only two countries had the highest number of participants in the competitions analyzed in the USA dataframe. The united states of America and Canada.
- **Participation Trends**: Seasonal analysis shows variations in participations, helping identify peak seasons.
- **Analytical Insights**: The analysis identifies the top-performing countries, proactive athletes, and seasons with greater numbers of participants.

### 6. Conclusion

This project serves as a comprehensive introduction to Pandas for data analysts, covering importing libraries, data cleaning and wrangling, exploratory data analysis, and data-driven pandas queries. The findings from this project can help drive analytical decisions by understanding participation patterns, athlete behavior, and athlete performance.

## User Guide

1. **Clone the Repository**: Clone this project repository from this link: https://www.kaggle.com/datasets/aiaiaidavid/the-big-dataset-of-ultra-marathon-running/discussion?sort=hotness.
2. **Upload the dataset**: Upload `TWO_CENTURIES_OF_UM_RACES` file to your coding environment.
3. **Run the Queries**: Use the Pandas queries provided in the `Analysis.ipynb` file to perform your analysis.
4. **Explore and Modify**: Feel free to modify the queries to explore different aspects of the dataset or answer additional analytical questions.

## Author - Maedeh Dehghan

This project is part of my portfolio, showcasing the Pandas skills essential for data analyst roles. If you have any questions, feedback, or would like to collaborate, feel free to get in touch!

**E-mail Address**: [mdehghan9397@gmail.com]

Thank you for your support, and I look forward to connecting with you!
