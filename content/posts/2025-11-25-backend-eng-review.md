---
title: "📝 A Review Of Backend Engineer Jobs, Part. I"
date: 2025-11-25T07:00:00+08:00
draft: false
---

## Why a review of backend engineering?

Like many people, I'm trying to break into the AI / software industry. This is really tough in general because entry-level jobs in the field has decreased by 10% in the past couple years while mid to senior level jobs still steadily increased ([Is Data Science Dying? - Marina Wyss - AI & Machine Learning](https://www.youtube.com/watch?v=VbB9irtpoDc)). Jobs like ML or AI engineer especially would require many years of experience. From stories I've heard, it seems like either you get an internship which would lead to a full-time job, or you get an ML-adjacent job and transfer to an ML job in the same company.

One of the best ML-adjacent job is naturally software engineer. Jobs titled with software engineer could be an umbralla term for a number of actual jobs. The job could focus on any of: frontend, backend, full-stack or even data engineering. Any of them are valid and interesting job functions. Here I focus on backend engineering just because I prefer it. It's more algorithmic and would touch on more challenging problems than simply interacting with database like software architecturing and database management. 

## Intro to this series

This notebook uses a pre-existing Python library [(github)](https://github.com/speedyapply/JobSpy) to scrape backend engineering jobs from major cities across the US, including most states. The choice of which city to scrape is arbitrary. In the end, the dataset was able to find 3060 jobs across the US. Each job is garanteed to have the following attributes:

- title
- location
- area
- company
- date posted
- job type
- is remote
- job level
- job function
- description

The city and state used to search for jobs is added as the `area` of jobs. The exact location of the actual jobs found could be different from the search term. 

A few notes:    
- I didn't write the scrapper so I don't know how it finds the jobs.  
- I'll attach in the end a list of cities covered in the dataset and the data aggregation pipeline.   
- All data are available on Kaggle: [**view and download**](https://www.kaggle.com/datasets/tianyimasf/backend-engineer-jobs-us/data)


## 🔍 Exploratory Data Analysis

For this analysis, I only scraped jobs from LinkedIn. There're also plenty of jobs on Indeed and Google Jobs, but LinkedIn's data is relatively clean and there would exist a lot of overlaps. 

I like to run `jobs.info()` to see what field exists, what is their data type and how many valid values they have. 

```python
# EDA
jobs = pd.read_csv(root_dir+"jobs.csv")
jobs.info()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 3060 entries, 0 to 3059
    Data columns (total 35 columns):
     #   Column                 Non-Null Count  Dtype  
    ---  ------                 --------------  -----  
     0   id                     3060 non-null   object 
     1   site                   3060 non-null   object 
     2   job_url                3060 non-null   object 
     3   job_url_direct         898 non-null    object 
     4   title                  3060 non-null   object 
     5   company                3060 non-null   object 
     6   location               2903 non-null   object 
     7   area                   3060 non-null   object 
     8   date_posted            2607 non-null   object 
     9   job_type               3060 non-null   object 
     10  salary_source          143 non-null    object 
     11  interval               143 non-null    object 
     12  min_amount             143 non-null    float64
     13  max_amount             143 non-null    float64
     14  currency               143 non-null    object 
     15  is_remote              3060 non-null   bool   
     16  job_level              3060 non-null   object 
     17  job_function           3059 non-null   object 
     18  listing_type           0 non-null      float64
     19  emails                 573 non-null    object 
     20  description            3060 non-null   object 
     21  company_industry       3060 non-null   object 
     22  company_url            3060 non-null   object 
     23  company_logo           3060 non-null   object 
     24  company_url_direct     0 non-null      float64
     25  company_addresses      0 non-null      float64
     26  company_num_employees  0 non-null      float64
     27  company_revenue        0 non-null      float64
     28  company_description    0 non-null      float64
     29  skills                 0 non-null      float64
     30  experience_range       0 non-null      float64
     31  company_rating         0 non-null      float64
     32  company_reviews_count  0 non-null      float64
     33  vacancy_count          0 non-null      float64
     34  work_from_home_type    0 non-null      float64
    dtypes: bool(1), float64(14), object(20)
    memory usage: 815.9+ KB

At first glance, the dataset has a lot of null values. However, on a closer look, a lot of the fields are not that important in the first palce. They're good to know if exists, but not necessary. Most of the fields that's expected to have values, like `title`, `company`, `description`, `company_industry` are all full.   

{{< expand title="Additional notes" >}}
A couple of columns that's expected to have full values, like `location` and `date_posted`, lack part of the values. Since I didn't build the scraper, I don't know why the values are lacking. For a serious analysis, this could be crucial. Since most of the values are present and we should already know the range based on the search term, it seems fine for me to leave it as is. Sometimes null values like this could reveal crucial problems in the system if this analysis is for business purposes, but that's not the case here.   
{{< /expand >}} 

The other thing is a few fields don't have all the values but only some, like `job_url_direct`, `emails`, `salary_source`, `interval`, `min_amount`, `max_amount` and `currency`. Not all of these fields in this notebook will be analyzed, but it's interesting to see, for example, 898 out of the 3060 jobs have direct link to their job postings and 573 out of the 3060 jobs have emails. Only 143 out of all jobs have salary ranges, which we'll do some light analysis on in the next section "Salary".

### Map of jobs counts of cities across the US

When scraping jobs, I tried my best to expand the search to represent major cities in most states and include cities that are not traditionally represented in similar analysis. For example, Milwaukee, WI, Salt Lake City, UT, Birmingham, AL and Jackson, MS are all included. Even if the analysis is not applied to any specific city or state, all codes will be available as expandable code blocks and all data will be available by city or as a whole so that people can conduct analysis themselves if they want to. To show the state of backend engineering jobs in each of the cities, the review will follow the format of dividing each attribute by its geographical locations in each main section. 

Below is the code to create a map of job counts by city. 

```python
# import libraries
import plotly.express as px
import plotly.io as pio
from geopy.geocoders import Nominatim
from geopy.extra.rate_limiter import RateLimiter
import time

# convert city, state to latitude, longitude
area_counts = jobs.groupby("area").size().reset_index(name="count")

geolocator = Nominatim(user_agent="job_mapper", timeout=10)
geocode = RateLimiter(geolocator.geocode, min_delay_seconds=1)

lat_list = []
lon_list = []

for area in area_counts["area"]:
    loc = geocode(f"{area}, USA")
    if loc:
        lat_list.append(loc.latitude)
        lon_list.append(loc.longitude)
    else:
        lat_list.append(None)
        lon_list.append(None)
    time.sleep(1)  # required to avoid rate-limit

area_counts["lat"] = lat_list
area_counts["lon"] = lon_list
```

This only needs to be run once. The following code plots the map of job counts by city in the US.

```python
# import saved file that map city, state to latitude, longitude
area_counts = pd.read_csv(root_dir+"area2latlon.csv")

fig = px.scatter_geo(
    area_counts,
    lat="lat",
    lon="lon",
    size="count",
    hover_name="area",
    hover_data={"count": True},
    size_max=50,
    opacity=0.7,
    color="count",
    color_continuous_scale="Viridis",
    projection="albers usa",
)

# Balanced background and black state lines
fig.update_layout(
    title="Jobs Counts By City Across The US",
    title_font_size=20,
    title_font_color="#555555",   # grey title
    geo=dict(
        scope="usa",
        showland=True,
        landcolor="#e0e0e0",        # medium-light gray
        lakecolor="#d0e8ff",        # pale blue lakes
        showlakes=True,
        showcountries=True,
        countrycolor="gray",
        showcoastlines=False,
        bgcolor="#f5f5f5",           # soft gray background
        showsubunits=True,
        subunitcolor="gray",         # black state lines
    ),
    paper_bgcolor="#f5f5f5",         # soft gray outside map
    font=dict(color="black"),
    margin={"r":0,"t":50,"l":0,"b":0},
    height=500,
    width=1000,
)

# Optional: add city labels
fig.update_traces(text=area_counts["area"], textposition="top center")

fig.show()
```

It looks like a lot of code, but it's mostly just styling.

{{< rawhtml >}}
<iframe src="/embeds/job_counts_by_city_us.html" width="710" height="410"></iframe>
{{< /rawhtml >}}

{{< expand title="Additional notes" >}}
It's black box why the scraper found exactly 140 jobs for all Boston, SF, San Jose, Seattle and Denver, but it's okay in my opinion because the numbers in general are correlated with our impression of which city has more tech jobs. 
{{< /expand>}}

There're more jobs in commonly-known tech hubs like San Francisco and San Jose (= 140). But there're also a lot of backend engineering jobs in big cities like Baltimore, Washington D.C. and Chicago. It might be expected that Southern state have fewer jobs, but it's good to see that these following cities also have a decent amount of jobs:

- Atlanta, GA
- Raleigh, NC
- Charlotte, NC
- Detroit, MI (80)
- Minneapolis, MN (75)
- Cincinnati, OH (70)
- Phoenix, AZ (70)
- Salt Lake City, UT (70)
- Hartford, CT (70)
- Pittsburgh, PA (69)

Especially, Atlanta, GA has 120 jobs, and 90 and 89 jobs were found at Raleigh and Charlotte from North Carolina respectively. For comparison, Hartford, CT also have 70 jobs and Hartford is one of the small cities that has a lot of very well-paid tech jobs right above the NY state.

{{< rawhtml >}}
<iframe src="/embeds/ratio_of_job_counts_by_city.html" width="710" height="510"></iframe>
{{< /rawhtml >}}

It seems from the US map of job counts by city that the majority of the jobs are from big tech cities. However, breaking it down in the second box/proportion graph, it's obvious that the number of jobs from big tech cities is clearly below the total number of jobs in the dataset. Moreover, there's a significant representation of cities from red states in the rest of the dataset, demonstrated by the city names in most of the boxes on the right side of the plot. 

## 💵 Salary

There're 143 jobs that have salary listed in total. Related fields are `salary_source`, `interval`, `min_amount` and `max_amount`. The field names don't tell us directly what the fields are, so the following blcok print the first value of each field. 

```python
salary_source = jobs.salary_source.dropna().tolist()
intervals = jobs.interval.dropna().tolist()
min_amount = jobs.min_amount.dropna().tolist()
max_amount = jobs.max_amount.dropna().tolist()
print("First value of each related field \n"+
f"salary_source: \"{salary_source[0]}\" \ninterval: {intervals[0]} \nmin_amount: {min_amount[0]} \nmax_amount: {max_amount[0]}")
```

    First value of each related field 
    salary_source: "description" 
    interval: yearly 
    min_amount: 90000.0 
    max_amount: 110000.0


From experience `salary_source` and `interval` would be the same for each of the 143 row. Just for scientific spirits, let's verify that.


```python
salary_source_unique = jobs.salary_source.dropna().unique().tolist()
intervals_unique = jobs.interval.dropna().unique().tolist()
print(f"Salary source(s): {salary_source_unique} \nIntervals: {intervals_unique}")
```

    Salary source(s): ['description'] 
    Intervals: ['yearly', 'hourly']


Turned out it's good we checked. When inspecting min salary amount and max salary amount, we'll check these seperately.


```python
min_amount_yearly = jobs[jobs.interval == 'yearly'].min_amount.dropna().tolist()
max_amount_yearly = jobs[jobs.interval == 'yearly'].max_amount.dropna().tolist()
min_amount_hourly = jobs[jobs.interval == 'hourly'].min_amount.dropna().tolist()
max_amount_hourly = jobs[jobs.interval == 'hourly'].max_amount.dropna().tolist()
print(f"# of yearly salary instances: {len(min_amount_yearly)} \n# of hourly salary instance(s): {len(min_amount_hourly)}")
```

    # of yearly salary instances: 140 
    # of hourly salary instance(s): 3



```python
for i in range(3):
    print(f"Hourly rate range {i+1}: ${min_amount_hourly[i]}/hr - ${max_amount_hourly[i]}/hr")
```

    Hourly rate range 1: $30.0/hr - $33.0/hr
    Hourly rate range 2: $75.0/hr - $80.0/hr
    Hourly rate range 3: $40.0/hr - $45.0/hr


We can translate hourly rates to yearly rates assuming the job works 40hr/week and 4 weeks/month.


```python
for i in range(3):
    min_amount_yearly.append(min_amount_hourly[i]*40*4*12)
    max_amount_yearly.append(max_amount_hourly[i]*40*4*12)
```

So then we get the numerical lower and higher end of minimal and maximal amount of salaries. 

```python
print(f"Lower end of minimal amount of salary: ${min(min_amount_yearly)/1000}k \n" +
      f"Higher end of minimal amount of salary: ${max(min_amount_yearly)/1000}k")
```

    Lower end of minimal amount of salary: $50.0k 
    Higher end of minimal amount of salary: $300.0k



```python
print(f"Lower end of maximal amount of salary: ${min(max_amount_yearly)/1000}k \n" +
      f"Higher end of maximal amount of salary: ${max(max_amount_yearly)/1000}k")
```

    Lower end of maximal amount of salary: $63.36k 
    Higher end of maximal amount of salary: $405.0k

It seems sort of arbitrary what these number actually means, so in the next section we look at the distribution for all salaries and then create maps of stats by city again.

### Distribution of Salary Range

First we load the libraries.

```python
import matplotlib.pyplot as plt
import logging
logging.getLogger("matplotlib.font_manager").disabled = True
```

The following create a styled graph of minimal and maximal salary distribution. 


```python
y_min = [1] * len(min_amount_yearly)
y_max = [2] * len(max_amount_yearly)

plt.xkcd()

plt.figure(figsize=(9, 4))
plt.scatter([x/1000 for x in min_amount_yearly], y_min, s=200, color='#FFB347', label='List1', marker='o', alpha=0.7)
plt.scatter([x/1000 for x in max_amount_yearly], y_max, s=200, color='#6A5ACD', label='List2', marker='o', alpha=0.7)

# Customize axes
plt.xticks(range(0, int(max(max_amount_yearly)/1000), 10), rotation=60)
plt.yticks([1,2], ['min salary', 'max salary'], rotation=45)
plt.xlabel('salary (k/yr)')
y_min = 0.5
y_max = 2.45
plt.ylim(y_min, y_max)
plt.title('Backend engineer salary range distribution')
plt.grid(axis='x', linestyle=':', linewidth=1.65, color='#333333', alpha=0.7)

plt.show()
```

![salary distribution](/images/salary-distribution.png)

It feels like the lower end of most salary ranges between \$50k - \$210k and the higher end of most salary ranges from \$65k to \$300k. By my own job browsing experience, this is pretty accurate. Compared to most tech jobs, this actually seems a bit low (quite low). If you search for data scientist or ML Eng jobs, the higher end could easily go up to around 600k to 800k. However, I hope this reiew can show **why in my opinion, backend engineering is a great first job for job seekers in this market and will likely open up further opportunities.**

**P.S.** If you're wondering why the graph is all doodles it's because of this magical command `plt.xkcd()`. You don't have to download extra packages because it's included in `matplotlib` but you do have to suppress the font manager warnings using `logging.getLogger("matplotlib.font_manager").disabled = True`.

### Median minimal and maximal yearly salary across US cities

The code for plotting the map is similar to the above and very long just because of styling, so it's skipped. But the data aggregation and manipulation code is shown below.

```python
# Converting hourly rates to yearly rates
min_amount_yearly = jobs[jobs.interval == 'yearly'].min_amount.dropna()
max_amount_yearly = jobs[jobs.interval == 'yearly'].max_amount.dropna()
min_amount_hourly = jobs[jobs.interval == 'hourly'].min_amount.dropna()
max_amount_hourly = jobs[jobs.interval == 'hourly'].max_amount.dropna()
min_amount_hourly = min_amount_hourly * 40 * 4 * 12
max_amount_hourly = max_amount_hourly * 40 * 4 * 12
min_amount_yearly = pd.concat([min_amount_yearly, min_amount_hourly], ignore_index=True)
max_amount_yearly = pd.concat([max_amount_yearly, max_amount_hourly], ignore_index=True)

# Data preparation: filtering cities that have jobs with salary
indices = jobs.min_amount.dropna().index.tolist()
areas_with_valid_rates = jobs.loc[indices].area.tolist()

# Data preparation: create the dataframe
data = {"area": areas_with_valid_rates,
       "min_amount": min_amount_yearly.tolist(),
       "max_amount": max_amount_yearly.tolist()}

df_salary_by_area = pd.DataFrame(data)

# Data aggregation: median minimal salary and job counts
df_area_min_salary_median = (
    df_salary_by_area.groupby("area")
    .agg(
        min_amount_median=("min_amount", "median"),
        count=("min_amount", "count")   # count rows per area
    )
    .reset_index()
)

# Data aggregation: median maximal salary and job counts
df_area_max_salary_median = (
    df_salary_by_area.groupby("area")
    .agg(
        max_amount_median=("max_amount", "median"),
        count=("max_amount", "count")   # same count (based on max_amount)
    )
    .reset_index()
)

area_counts = pd.read_csv(root_dir+"area2latlon.csv")

# --------Gets missing areas----------
df_area_min_salary_median = df_area_min_salary_median.merge(
    area_counts[['area', 'lat', 'lon']],
    on='area',
    how='left'
)

# Get sets of areas
all_areas = set(jobs["area"])
median_areas = set(df_area_min_salary_median["area"])

# Areas present in jobs but missing in median
missing_areas = list(all_areas - median_areas)

# Subset lat/lon for missing areas
missing_coords = area_counts[area_counts["area"].isin(missing_areas)][['area', 'lat', 'lon']]
```

The resulted maps are shown below.

{{< rawhtml >}}
<iframe src="/embeds/min_salary_map.html" width="710" height="410"></iframe>
{{< /rawhtml >}}

{{< rawhtml >}}
<iframe src="/embeds/max_salary_map.html" width="710" height="410"></iframe>
{{< /rawhtml >}}

The first thing to notice is a few cities you wouldn't normally expect to have high median salaries have really high median salary. Hovering on the bubbles, the count tells you how many job in that city have salary listed. Most of these cities only have 1 to 3 jobs that have salary labels, a very small sample size that wouldn't be very representative. So the median salary of these cities shown on the map can't be fully trusted. A hypothesis could be that only companies who can offer good salary listed the number, but this needs to be verified.

The couple other things to notice: San Diego, CA has 5 jobs with a high median salary range of \$172k/yr - \$258k/yr. Next, the following cities have shown (diabolically) low median salaries.

- Salt Lake City, UT (\$101k/yr - \$108.5k/yr)
- Milwaukee, WI (\$77k/yr - \$86k/yr)
- Detroit, MI (\$80k/yr - \$110k/yr)
- Indianapolis, IN (\$70k/yr - \$75k/yr)
- Birmingham, AL (\$90k/yr - \$110k/yr)

If you look through the bubbles, the median salaries in most cities on the map is \$130k/yr - \$195k/yr. So having salaries that range from around \$80k/yr to barely \$110k/yr shows that these cities might not have the most well-paid jobs. 

Lastly, the cities that don't have jobs with salary listed is a mix of couple big cities and major cities from red states. This could mean that not having listed salary is common practice but needs further verification which we'll not dive into in this analysis.

---

This concludes Part. I of my review of backend engineering job. Hope it's interesting in some ways. 