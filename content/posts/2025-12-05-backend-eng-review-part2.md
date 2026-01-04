---
title: "📝 A Review Of Backend Engineer Jobs, Part. II"
date: 2025-12-05T07:00:00+08:00
draft: false
---

## 📌 Recap

In Part. I, I introduced why I decided to do this analysis, performed exploratory data analysis (EDA) and an analysis of median salary ranges. As a reminder, the motivation for the analysis is backend engineering is a good entry-level job for people who wants to break into the tech industry. EDA revealed that there’re a lot of backend engineering jobs in big cities like Baltimore, Washington D.C. and Chicago. Salary analysis showed that cities like Detroit, Milwaukee, Indianapolis, Birmingham and Salt Lake City have really low salary range.

In this part, I delve deeper into other parts of the datasets that might be of interests, such as job levels, companies & industry, and remote jobs. 

## 🏅 Job Level

First, we import some ata scientist jobs collected from a few sampled big tech cities are used here in comparison to the backend engineering job dataset.

```python
jobs_ds = pd.read_csv(root_dir+"jobs_ds.csv")
print(f"The data scientist dataset has {jobs_ds.shape[0]} jobs. ")
```

The data scientist dataset has 834 jobs. 

Then we can plot a bar chart separated by levels for backend engineering and data science respectively to show the relative frequency of each level. 

{{< expand title="Code Block: Bar chart" >}}
```python
# Prepare data
levels_backend = jobs.job_level
levels_freq_backend = levels_backend.value_counts()

levels_ds = jobs_ds.job_level
levels_freq_ds = levels_ds.value_counts()

# Create subplots side by side
fig, axes = plt.subplots(1, 2, figsize=(12, 5))  # 1 row, 2 columns

# --- Backend Engineering Plot ---
levels_freq_backend.plot(kind='bar', ax=axes[0])
axes[0].set_xlabel("Job Levels")
axes[0].set_ylabel("Frequency")
axes[0].set_xticklabels(levels_freq_backend.index, rotation=45)
axes[0].grid(axis='y', linestyle=':', linewidth=1.65, color='#333333', alpha=0.7)

# Add numerical labels
for i, value in enumerate(levels_freq_backend):
    axes[0].text(i, value + 0.05, str(value), ha='center', va='bottom', fontsize=12)

axes[0].set_title("Frequently Demanded Job Levels (Backend Engineering)", fontsize=14)

# --- Data Scientist Plot ---
levels_freq_ds.plot(kind='bar', ax=axes[1], color='orange')  # optional color change
axes[1].set_xlabel("Job Levels")
axes[1].set_ylabel("Frequency")
axes[1].set_xticklabels(levels_freq_ds.index, rotation=45)
axes[1].grid(axis='y', linestyle=':', linewidth=1.65, color='#333333', alpha=0.7)

# Add numerical labels
for i, value in enumerate(levels_freq_ds):
    axes[1].text(i, value + 0.05, str(value), ha='center', va='bottom', fontsize=12)

axes[1].set_title("Frequently Demanded Job Levels (Data Scientist)", fontsize=14)

plt.tight_layout()
plt.show()
```
{{< /expand>}}

![job_levels](/images/job_levels.png)

In agreement with the common perception of the trend of job demands in the tech industry, there are more mid to senior level jobs than entry-level jobs. For the job title "Data Scientist", half of the jobs in the dataset are entry-level. In comparison, about 36% percent of backend engineering jobs are entry-level jobs, which I'd say is still pretty good. 

An extra consideration could be the definition of mid-senior level jobs for these two titles. Typically, people hiring for even entry-level data scientists would require a higher education degree like master's or PhD and proof of understanding of machine learning or deep learning concepts. However, backend engineering usually only requires a general understanding of programming, problem solving and maybe a framework like React or NextJS. It's much more difficult to get to mid-senior or even entry level as a data scientist compared to a backend engineer. The demand for entry-level AI engineer jobs has decreased by 10% in the past couple years, and people usually hire from the inside instead ([Is Data Science Dying? - Marina Wyss - AI & Machine Learning](https://www.youtube.com/watch?v=VbB9irtpoDc)).

{{< expand title="Code Block: Data Manipulation" >}}
```python
# find entry and mid-senior jobs and their counts
mid_senior_jobs = jobs[jobs["job_level"] == "mid-senior level"]
entry_jobs = jobs[jobs["job_level"] == "entry level"]

mid_senior_counts = mid_senior_jobs.groupby("area").size().reset_index(name="job_count")
entry_counts = entry_jobs.groupby("area").size().reset_index(name="job_count")

# Merge lat/lon
mid_senior_counts = mid_senior_counts.merge(area_counts[['area','lat','lon']], on='area', how='left')
entry_counts = entry_counts.merge(area_counts[['area','lat','lon']], on='area', how='left')
```
{{< /expand>}}


{{< expand title="Code Block: Map" >}}
```python
# Entry Level Map
fig2 = px.scatter_geo(
    entry_counts,
    lat="lat",
    lon="lon",
    size="job_count",
    color="job_count",
    hover_name="area",
    size_max=50,
    opacity=0.7,
    color_continuous_scale="Viridis",
    projection="albers usa",
    title="Entry Level Job Count by City"
)

fig2.update_layout(
    geo=dict(
        scope="usa",
        showland=True,
        landcolor="#e0e0e0",
        lakecolor="#d0e8ff",
        showlakes=True,
        showcountries=True,
        countrycolor="gray",
        showcoastlines=False,
        bgcolor="#f5f5f5",
        showsubunits=True,
        subunitcolor="black",
    ),
    paper_bgcolor="#f5f5f5",
    height=600,
    width=1000,
)

fig2.show()

# Mid-Senior Level Map
fig1 = px.scatter_geo(
    mid_senior_counts,
    lat="lat",
    lon="lon",
    size="job_count",
    color="job_count",
    hover_name="area",
    size_max=50,
    opacity=0.7,
    color_continuous_scale="Viridis",
    projection="albers usa",
    title="Mid-Senior Level Job Count by City"
)

fig1.update_layout(
    geo=dict(
        scope="usa",
        showland=True,
        landcolor="#e0e0e0",
        lakecolor="#d0e8ff",
        showlakes=True,
        showcountries=True,
        countrycolor="gray",
        showcoastlines=False,
        bgcolor="#f5f5f5",
        showsubunits=True,
        subunitcolor="black",
    ),
    paper_bgcolor="#f5f5f5",
    height=600,
    width=1000,
)

fig1.show()
```
{{< /expand>}}

{{< rawhtml >}}
<iframe src="/embeds/entry_job_map.html" width="710" height="410"></iframe>
{{< /rawhtml >}}

{{< rawhtml >}}
<iframe src="/embeds/mid_senior_job_map.html" width="710" height="410"></iframe>
{{< /rawhtml >}}

It seems like the following cities have more entry-level jobs, ranked from the most jobs to fewer jobs:

- Seattle, WA (38)
- Sacramento, CA (33)
- Denver, CO (32)
- San Jose, CA (30)
- San Francisco, CA (28)
- Baltimore, MD (28)
- Boston, MA (25)
- Washington D.C., D.C. (25)

And the following cities have more mid-senior level jobs (>90): New York, NY / Baltimore, MD / Washington D.C., D.C. / Boston, MA / Denver, CO. Atlanta and Chicago each have around 70 jobs, while cities in the West Coast actually have a bit less (around 60) mid-senior jobs.

## 🏢 Companies

```python
# View top 10 most mentioned companies
company_counts = jobs['company'].value_counts()

top_companies = company_counts.head(25)
print(top_companies)
```

    company
    Epic                         136
    Jobs via Dice                 65
    Google                        47
    PwC                           35
    ExecutivePlacements.com       34
    KPMG US                       34
    Intuit                        25
    GEICO                         25
    EY                            23
    Qualcomm                      22
    Pearson                       22
    Booz Allen Hamilton           21
    Meta                          21
    Tata Consultancy Services     19
    Uline                         18
    Mindrift                      18
    Adobe                         18
    Motion Recruitment            17
    Affirm                        17
    Capital One                   16
    Lockheed Martin               16
    Deloitte                      16
    Scribd, Inc.                  15
    Boeing                        15
    Raytheon                      15
    Name: count, dtype: int64

By getting the most mentioned companies in this dataset, we can see that Epic the electronic health record (EHR) system listed the most backend engineering jobs (136 jobs). While some suspicious job websites are up on the list, like Jobs via Dice, ExecutivePlacement.com and Motion Recruitment, most of the companies are well-known like Google, PwC, etc.

```python
# Check # of FAANG jobs
google = company_counts["Google"]
meta = company_counts["Meta"]
amazon = company_counts["Amazon"]
netflix = company_counts["Netflix"]
apple = 0
if "Apple" in company_counts.index: 
    apple = company_counts["Apple"]
print(f"# of FANNG jobs \nGoogle: {google}\nMeta: {meta} \nAmazon: {amazon} \nNetflix: {netflix} \nApple: {apple}")
```

    # of FANNG jobs 
    Google: 47
    Meta: 21 
    Amazon: 1 
    Netflix: 14 
    Apple: 0

A quick check people like to do is to look for FAANG company jobs. FAANG are five big tech companies recognized in the industry, standing for Facebook (now Meta), Amazon, Apple, Netflix, and Alphabet (Google) respectively. Here we have Google: 47 jobs, Meta: 21 jobs, Amazon: 1 job and Netflix: 14 jobs. Apple has no job listed, just because they don't post ANY job on ANY job boards and only on their own career website, so it's normal that our dataset doesn't contain any job from Apple. Our job source is LinkedIn.

```python
# Plot WordCloud of Companies
from wordcloud import WordCloud

# List of companies to remove
remove_companies = ["Jobs via Dice", "ExecutivePlacements.com", "Motion Recruitment"]

# Filter out these companies from the frequency dictionary
company_freq_filtered = {company: count for company, count in company_counts.items() if company not in remove_companies}

# Generate the word cloud from the filtered dictionary
wordcloud = WordCloud(
    width=800,
    height=400,
    background_color='white',
    colormap='viridis'
).generate_from_frequencies(company_freq_filtered)

# Plot
plt.figure(figsize=(15, 7))
plt.imshow(wordcloud, interpolation='bilinear')
plt.axis('off')
plt.title("Most Mentioned Companies", fontsize=24, font="DejaVu Sans")
plt.show()
```

![wordcloud](/images/company_wordcloud.png)

Some of the well-known companies that're on the list are: KPMG US, EY, Intuit, PwC, Qualcomm, Tata Consultancy Services, Booz Allen Hamilton, Affirm, Pearson, GEICO, and companies with less backend engineering jobs like Adobe, IBM, Lockheed Martin, Deloitte, NVIDIA and Boeing. 

## 🏭 Industry

```python
industry = jobs.company_industry

# List of companies to remove
remove_companies = ["Jobs via Dice", "ExecutivePlacements.com", "Motion Recruitment"]

# Filter the jobs dataframe to exclude these companies
jobs_filtered = jobs[~jobs['company'].isin(remove_companies)]

industry_counts = jobs_filtered['company_industry'].value_counts()

# Get top 10 most frequent industries
top_industries = industry_counts.head(7)
print(top_industries)
```

    company_industry
    Software Development                                         486
    IT Services and IT Consulting                                353
    Financial Services                                           217
    Technology, Information and Internet                         151
    Defense and Space Manufacturing                              117
    Insurance                                                     79
    Hospitals and Health Care and Information Technology          68

{{< expand title="Code Block: Bar chart" >}}
```python
# Convert top_industries to a dataframe for easier manipulation
top_industries_df = top_industries.reset_index()
top_industries_df.columns = ['industry', 'count']

# Rename the 7th industry (index 6) to "Hospitals and Health Care Software"
top_industries_df.loc[6, 'industry'] = "Hospitals and Health Care Software"

# Plot as bar chart
plt.figure(figsize=(10,6))
bars = plt.bar(top_industries_df['industry'], top_industries_df['count'])

# Add text labels on top of each bar
for bar in bars:
    height = bar.get_height()
    plt.text(
        bar.get_x() + bar.get_width()/2,  # x-position
        height + 0.5,                     # y-position slightly above bar
        str(int(height)),                  # label text
        ha='center', va='bottom', fontsize=12
    )

plt.xticks(fontsize=12)
plt.xlabel("Industry")
plt.ylabel("Number of Job Postings")
plt.title("Top 7 Most Frequent Industries")
plt.xticks(rotation=45, ha='right', fontsize=12)
plt.grid(axis='y', linestyle=':', linewidth=1.65, color='#333333', alpha=0.7)

plt.show()
```
{{< /expand>}}

![industry](/images/freq_industry.png)

Most jobs are from the software development, consulting or financial services industry. Other more specific industries are Defense and Space Manufacturing, Insurance and Healthcare Software. Some examples of software development industry companies could be Slack, Pearson and Scribd, Inc. Consulting companies are straightforward, like EY and PwC. Financial services companies include Affirm, Intuit and Capital One. 

## 🌐 Is Remote

```python
# Count remote jobs
is_remote = jobs_filtered.is_remote
remote_counts = jobs['is_remote'].value_counts()
print(remote_counts)
```

    is_remote
    False    2322
    True      738

It's clear that a bit less than a quater of the jobs is remote, which is a good amount. To better visualize this, let's do a pie chart. 

{{< expand title="Additional Notes" >}}
Pie chart is widely criticized and avoided in the visualization community. It's well-known that if you want to visualize the ratios of data, a pie chart won't give an accurate presentation [("5 alternatives to pie chart - Shelby Temple")](https://www.tableau.com/blog/5-unusual-alternatives-pie-charts). However, it's kinda perfect in this case because it's almost exactly a quater, so I thought it'd be funny to finally be able to use it once. 
{{< /expand>}}

{{< expand title="Code Block: Pie chart (horrific)" >}}
```python
# Count remote vs non-remote jobs
remote_counts = jobs_filtered['is_remote'].value_counts()
labels = ['Non-Remote', 'Remote']

# Custom function to show both count and percentage
def make_autopct(values):
    def my_autopct(pct):
        total = sum(values)
        count = int(round(pct*total/100.0))
        return f"{count} ({pct:.1f}%)"
    return my_autopct

plt.figure(figsize=(6,6))
plt.pie(
    remote_counts,
    labels=labels,
    autopct=make_autopct(remote_counts),
    colors=['#FF9999','#66B3FF'],
    startangle=90,
    shadow=True
)
plt.title("Proportion of Remote vs Non-Remote Jobs", fontsize=14)
plt.show()
```
{{< /expand>}}

![remote jobs](/images/remote_jobs.png)

The quater ratio is pretty clear based on this chart. The next step could be to visualize remote jobs by area.

```python
# Filter remote jobs
remote_jobs = jobs_filtered[jobs_filtered['is_remote'] == True]

# Group by area/city and count jobs
remote_counts = remote_jobs.groupby("area").size().reset_index(name="job_count")

# Merge lat/lon for plotting (assuming area_counts dataframe exists)
remote_counts = remote_counts.merge(area_counts[['area','lat','lon']], on='area', how='left')
```

{{< expand title="Code Block: Map" >}}
```python
fig = px.scatter_geo(
    remote_counts,
    lat="lat",
    lon="lon",
    size="job_count",
    color="job_count",
    hover_name="area",
    size_max=50,
    opacity=0.7,
    color_continuous_scale="Viridis",
    projection="albers usa",
    title="Remote Job Count by City"
)

# Map layout styling
fig.update_layout(
    geo=dict(
        scope="usa",
        showland=True,
        landcolor="#e0e0e0",
        lakecolor="#d0e8ff",
        showlakes=True,
        showcountries=True,
        countrycolor="gray",
        showcoastlines=False,
        bgcolor="#f5f5f5",
        showsubunits=True,
        subunitcolor="black",
    ),
    paper_bgcolor="#f5f5f5",
    height=600,
    width=1000,
)

fig.show()
```
{{< /expand>}}

{{< rawhtml >}}
<iframe src="/embeds/remote_job_map.html" width="710" height="510"></iframe>
{{< /rawhtml >}}

Separating remote jobs by city, the map shows that Denver (50), Baltimore (43), Washington DC (36), Boston, New York and Chicago. All of them have more than 30 remote jobs. Atlanta and Raleigh don't have as much but still a decent amount (29 jobs and 26 jobs, respectively).

It's interesting from all of the above analysis, traditionally well-regarded big tech cities like San Francisco and Seattle are seldom mentioned. Aside from the fact that those California cities have well-paid jobs, it seems like New York, Denver, Baltimore, Washington DC, Boston, Chicago, Atlanta and Raleigh have comparably well-paid jobs that are entry, mid-senior level and remote. One more signal to look for yourself could be which city have which industry. This is too detailed for this review, but the dataset of all jobs, as well as by city and the sampled data scientist dataset will be uploaded to Kaggle and available for download. [(data)](https://www.kaggle.com/datasets/tianyimasf/backend-engineer-jobs-us)

---

This concludes Part. II of my review of backend engineering job. 

