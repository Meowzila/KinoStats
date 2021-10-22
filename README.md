# KinoStats Description
A mini project that visualizes data from a PostgreSQL database with 5 million+ rows of scraped and cleaned data.

# Data Visualization Examples

## Select films where the director played a role in their own film and the film is rated.
```sql
df = pd.read_sql_query(f"""

    SELECT DISTINCT film_name, director, actor, rating FROM public.films
    INNER JOIN actors ON actorid = id
    WHERE REPLACE(LOWER(director), ' ', '-') = actor and rating > 0
    ORDER BY rating DESC
    
    """, engine)
display(df)
```
![alt text](https://i.imgur.com/ackgjF6.png)

## Select films where a specific director (Ex. Quentin Tarantino) played a role in their own film and the film is rated.
```sql
df_casted = pd.read_sql_query(f"""

    SELECT * FROM public.films
    INNER JOIN actors ON actorid = id
    WHERE REPLACE(LOWER(director), ' ', '-') = actor and rating > 0 and director = 'Quentin Tarantino'
    
    """, engine)
display(df_casted)
```
![alt text](https://i.imgur.com/wf2RweN.png)

## Select Top 10 films where the director played a role in the film.
```sql
df = pd.read_sql_query(f"""

    SELECT director, count(director) FROM public.films
    INNER JOIN actors ON actorid = id
    WHERE REPLACE(LOWER(director), ' ', '-') = actor and rating > 0
    GROUP BY director
    ORDER BY COUNT(director) DESC
    LIMIT 10
    
    """, engine)
```
### Plotting the data...
```py
directors = list(df['director'])
num_films = list(df['count'])
figure = sns.barplot(x=directors, y=num_films)
figure.set_xticklabels(figure.get_xticklabels(), rotation=45, horizontalalignment='right')
figure.set_title('Top 10 Directors Who Also Played Roles in Their Films')
figure.set_ylabel('Number of Films')
plt.show()
```
![alt text](https://i.imgur.com/POgjf3n.png)

## Select number of animated movies in the last 20 years
```sql
df = pd.read_sql_query(f"""
    SELECT year, count(year) FROM public.films
    INNER JOIN genres ON genreid = id
    WHERE genre = 'animation' and country = 'usa' and rating > 0 and year BETWEEN 2000 and 2020
    GROUP BY year
    """, engine)
```
### Plotting the data with trendline...
```py
# Plot animated movie data with trendline
years = list(df['year'])
num_films = list(df['count'])
figure = sns.barplot(x=years, y=num_films, palette=sns.color_palette('bright')[0:1])
figure.set_title('Animated Movies Released in the U.S. from 2000 to 2020')
figure.set_xticklabels(figure.get_xticklabels(), rotation=45, horizontalalignment='right')
figure.set_ylabel('Number of Films Released')

for c in figure.patches:
    c.set_zorder(0)
sns.regplot(x=np.arange(0, len(years)), y=num_films, ax=figure)

from scipy.stats import linregress
rvalue = mpatches.Patch(label=f'R: {linregress(years,num_films)[2].__round__(2)}')
plt.legend(handles=[rvalue])
plt.show()
```
![alt text](https://i.imgur.com/Vqpd5hx.png)

## Select number of action movies released in the last 10 years compared to all movies in the last 10 years
```sql
df = pd.read_sql_query(f"""
    SELECT year, count(year) FROM public.films
    INNER JOIN genres ON genreid = id
    WHERE genre = 'action' and country = 'usa' and rating > 0 and year BETWEEN 2010 and 2020
    GROUP BY year
    UNION ALL
    SELECT year, count(year) FROM public.films
    WHERE country = 'usa' and rating > 0 and year BETWEEN 2010 and 2020
    GROUP BY year
    """, engine)
```
### Plotting the data...
```py
fig_df = pd.DataFrame({"Action Films": list(df['count'][0:11]), 
                       "All Films": list(df['count'][11:22]),
                       "Year": list(df['year'][0:11])})
s1 = sns.barplot(x = 'Year', y = 'All Films', data = fig_df, color=sns.color_palette('bright')[0], label = 'All Films')
s2 = sns.barplot(x = 'Year', y = 'Action Films', data = fig_df, color=sns.color_palette('bright')[1], label = 'Action Films')
s1.set_title('Action Movies vs Total Movies Released from 2010 to 2020')
s1.set_ylabel('Number of Films Released')
color1 = mpatches.Patch(color=sns.color_palette('bright')[0], label='All Films')
color2 = mpatches.Patch(color=sns.color_palette('bright')[1], label='Action Films')
plt.legend(handles=[color1, color2])
```
![alt text](https://i.imgur.com/VfcspwH.png)

## Top 20 Non-English movies released in the US by number of movies released
```sql
df = pd.read_sql_query(f"""
    SELECT language, count(language) FROM public.films
    WHERE country = 'usa' and rating > 0 and language != 'english' and language != 'no-spoken-language'
    GROUP BY language
    ORDER BY count(language) desc
    LIMIT 20
    """, engine)
```
### Plotting the data...
```py
language = list(df['language'])
num_films = list(df['count'])
figure = sns.barplot(x=language, y=num_films)
figure.set_title('Non-English Movies Made in the United States')
figure.set_xticklabels(figure.get_xticklabels(), rotation=45, horizontalalignment='right')
figure.set_ylabel('Number of Films Released')
plt.show()
```
![alt text](https://i.imgur.com/M8ceH2z.png)
