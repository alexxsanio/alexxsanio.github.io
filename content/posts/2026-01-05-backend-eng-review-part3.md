---
title: "📝 A Review Of Backend Engineer Jobs, Part. III"
date: 2026-01-05T07:00:00+08:00
draft: false
---

## 📌 Recap

Part. I & II of this analysis series covered EDA, salary, levels, companies and remote or not of the backend engineering job market. This part of the analysis extracts about sections, skills keywords, key phrases of required experiences and benefits of the job posting dataset using text analysis (natural language processing). 

## 🧾 Job Description

Each job description would have an about section, a skill section, a responsibility section and a benefit section. We write a suite of functions to heuristically extract these sections and key words and phrases. In the end, we build a dictionary of info from each job description.

{{< expand title="Code Block: Putting it all together" >}}
```python
# Utility function: extract job info
def extract_sections_for_description(description):
    # get sections as text strings
    about = extract_about_section(description)
    responsibility, requirements, benefits = extract_job_info(description)
    
    skills = []
    key_phrases_exp = []
    key_phrases_bg = []
    
    # extract skills and key responsibility phrases
    for r in responsibility:
        skills_r = list(extract_skills(r))
        skills.extend(skills_r)
        key_phrases_r_exp = extract_phrases(r, level_experience)
        if key_phrases_r_exp: key_phrases_exp.extend(key_phrases_r_exp)
        key_phrases_r_bg = extract_phrases(r, level_background)
        if key_phrases_r_bg: key_phrases_bg.extend(key_phrases_r_bg)

    # extract skills and key resquirements phrases
    for req in requirements:
        skills_r = list(extract_skills(req))
        skills.extend(skills_r)
        key_phrases_r_exp = extract_phrases(req, level_experience)
        if key_phrases_r_exp: key_phrases_exp.extend(key_phrases_r_exp)
        key_phrases_r_bg = extract_phrases(r, level_background)
        if key_phrases_r_bg: key_phrases_bg.extend(key_phrases_r_bg)

    # extract bullet points of benefits
    benefits = get_benefits(benefits)
    
    # return about sections, key words and phrases, bullet points
    return about, skills, key_phrases_exp, key_phrases_bg, benefits

# --- extract info from description ---
data = []

for desc in jobs.description:
    a, s, kph_exp, kph_bg, b = extract_sections_for_description(desc)
    res = {
        "about": a,
        "skills": s,
        "key_phrases_exp": kph_exp,
        "key_phrases_bg": kph_bg,
        "benefits": b,
    }
    data.append(res)

# Save data
with open("data.json", "w") as f:
    json.dump(data, f, indent=2)
```
{{< /expand >}} 

### About Section

For every about section extracted, it's converted to a document vector and then clustered using KMeans. Each vector with dimension 364 is then reduced to 3d vectors using t-SNE ([Introduction to t-SNE - datacamp](https://www.datacamp.com/tutorial/introduction-t-sne)). The 3d vectors are visualized in 3d space color-coded with their respective cluster and labeled with the original text when hovered. 

{{< rawhtml >}}
<iframe src="/embeds/about_clusters.html" width="710" height="610"></iframe>
{{< /rawhtml >}}

It doesn't feel like the clusters mean a lot. To further validate (or invalidate) this observation, we can sample the cluster of about sections and summarize each sample. The results are below:

Group 0 of about sections seem to be about introducing the mission of the company. 

{{< expand title="Summary: Group 0" >}}
 Akraya is an award-winning IT staffing firm consistently recognized for its commitment to excellence and a thriving work environment . At Ford Motor Company, we believe freedom of movement drives human progress . At Scribd (pronounced “scribbed”), our mission is to spark human curiosity . At Sherwin-Williams, our purpose is to inspire and improve the world by coloring and protecting what matters . At Recidiviz is creating safer, healthier communities by improving outcomes for people in the criminal justice system . At Gusto, we're on a mission to grow the small business economy . At Bridge, we offer "Culture Conversations Conversations" to help you gain insights into our culture . All your information will be kept confidential according to EEO guidelines .
 {{< /expand >}} 

Group 1 of about sections seem to be about introducing the position itself. 

{{< expand title="Summary: Group 1" >}}
 The Product Engineering Intern will join Lumafield’s Product team to prototype and ship experimental software that brings 3D data to life . The ByteDance Doubao (Seed) Team, is dedicated to pioneering advanced AI foundation models . The salary range listed and paragraph are only applicable to U.S.-based candidates . The position will be performed alongside a cohort of summer interns and is a paid position . We do not currently offer visa sponsorship or OPT eligibility . We encourage you to apply anyway – If you're excited about our technology, the opportunity, and are eager to learn more we’d love to hear from you! Etched  Ergo turns every customer conversation—across Zoom, email, Slack, and phone
{{< /expand >}} 

Group 2 of about sections seem to be different from Group 1 because Group 1 introduces more relaxed positions with a focus on benefits, while Group 2 of about sections introduces positions that has much higher and more specific requirements for the candidate, mentioning technical terms like "distributed computing", "large-scale system design" and "building a product experience" which are all requirements for higher level engineer positions. 

{{< expand title="Summary: Group 2" >}}
 The US base salary range for this full-time position is $113,000-$161,000 . The ERP Application Product Specialist will be responsible for managing our Vista and Vista Web systems . Netflix is looking for engineers who bring fresh ideas from all areas, including information retrieval, distributed computing,. large-scale system design, networking and data storage,. natural language processing, UI design and mobile . The SWT Team builds internal tooling that builds how our developers build, maintain, and test Netflix products . You’re not just writing backend code, you're building a product experience you believe in . You're passionate about observability, reliability, performance and ensuring these are understood by everyone on the team as it relates to our backend systems .
{{< /expand >}} 

Group 3 mentioned data engineering and have a large stress on benefits as well. 

{{< expand title="Summary: Group 3" >}}
 As a Data Engineer, you will monitor the measurement, audience and data products by developing Key Performance Indicators (KPIs), data pipelines, dashboards, and doing ad-hoc analysis to derive insights and opportunities . SRC offers a generous benefit package, including medical, dental, vision, 401(k) with a company match, life insurance, vacation and sick paid time off accruals starting at 10 days of vacation and 5 days of sick leave annually, 11 paid holidays, tuition reimbursement, and tuition reimbursement . Netflix has a unique culture and environment. We do not discriminate on the basis of race, religion, color, . For more information: http://www.dailymailonline.com/news/business//
{{< /expand >}} 

Group 4 orients more towards govenemental employers, boundaries and restrictions. 

{{< expand title="Summary: Group 4" >}}
 Medtronic will consider for employment qualified job applicants with arrest or conviction records in accordance with the Los Angeles County Fair Chance Ordinance for Employers and the California Fair Chance Act . Cintel, Inc. expressly prohibits any form of unlawful employee harassment or discrimination based on any of the characteristics mentioned above . Alaska is not a state, but a state or state, or state of the United States, with jurisdiction in Washington DC or Alaska . Puerto Rico is one of the U.S. Virgin Islands and Alaska is a state of origin in the state of Washington, D.C., Alaska, Alaska, New England, Delaware and Delaware . The state of Puerto Rico has jurisdiction in Puerto Rico, Alaska and Hawaii . The U.N. has jurisdiction for Puerto
{{< /expand >}} 

Similar to Group 4, Group 5 focuses on state, space and defense agencies.

{{< expand title="Summary: Group 5" >}}
 Capella Space is a pioneer in Synthetic Aperture Radar (SAR) satellite technology and space-based signal intelligence . Capella is charting the future of Earth observation . InVita Healthcare Technologies is a leading software provider for complex medical, forensics, and community care environments . The posting will remain active until the position is filled, or a qualified pool of candidates is identified . The South Carolina Law Enforcement Division (SLED) is a premier statewide law enforcement agency dedicated to serving and protecting the citizens of South Carolina . The position will remain open for up to five days, with the posting being posted for as many as five days . BAE Systems, Inc. is an equal opportunity employer and will consider all applications without regard to race, sex, age, color,
{{< /expand >}} 

Group 6 has a distinct focus on companies that have ambitious mission statements to better the wrold, using words like "the world", "of all".

{{< expand title="Summary: Group 6" >}}
 The group says they embrace responsibility to change the way the world is now and that it will be better for all in the future . They say they want the change to be more equitable, safe and for the future of all of the world . We want to lead change that makes our world safer, safer, more equitable and more for all, not just for the sake of all, but for the benefit of all . Our vision is a world with Zero Crashes, Zero Emissions and Zero Congestion and we embrace the responsibility to lead the change that will make our world better, safer and more equitable for all . We embrace responsibility for the change we want to make the world better and safer for all for all. We are committed to leading
{{< /expand >}} 

{{< expand title="Code Block: Summarization using Hugging Face Transformer and Pipeline" >}}
```python
# Library imports
from transformers import pipeline, AutoTokenizer
import random

# --- define models --- 
MODEL_NAME = "sshleifer/distilbart-cnn-12-6"

tokenizer = AutoTokenizer.from_pretrained(MODEL_NAME)

summarizer = pipeline(
    "summarization",
    model=MODEL_NAME,
    tokenizer=tokenizer
)

# --- utility functions ---
# Chunk texts for faster processing speed
def chunk_text_by_tokens(text, tokenizer, max_tokens=900):
    tokens = tokenizer.encode(text, add_special_tokens=False)
    return [
        tokenizer.decode(tokens[i:i + max_tokens], skip_special_tokens=True)
        for i in range(0, len(tokens), max_tokens)
    ]

# Summarize individual texts
def summarize_text_safe(
    text,
    tokenizer,
    summarizer,
    max_tokens=900,
    max_length=400,
    min_length=150
):
    # Step 1: chunk original text
    chunks = chunk_text_by_tokens(text, tokenizer, max_tokens)
    summaries = []

    for chunk in tqdm(chunks, desc="Summarizing chunks", leave=False):
        result = summarizer(
            chunk,
            max_length=max_length,
            min_length=min_length,
            do_sample=False
        )
        summaries.append(result[0]["summary_text"])

    # Step 2: if combined summaries are still too large, recurse
    combined = " ".join(summaries)
    combined_tokens = len(tokenizer.encode(combined, add_special_tokens=False))

    if combined_tokens > max_tokens:
        # 🔁 recursively summarize summaries
        return summarize_text_safe(
            combined,
            tokenizer,
            summarizer,
            max_tokens=max_tokens,
            max_length=max_length,
            min_length=min_length
        )

    # Step 3: final safe summary
    final = summarizer(
        combined,
        max_length=max_length,
        min_length=min_length,
        do_sample=False
    )

    return final[0]["summary_text"]

# Summarize each group of texts
def summarize_grouped_dict(grouped, sample_size=20):
    for label, texts in tqdm(grouped.items(), desc="Summarizing groups"):
        if not texts:
            continue

        sampled = random.sample(texts, min(sample_size, len(texts)))
        combined_text = " ".join(sampled)

        summary = summarize_text_safe(
            combined_text,
            tokenizer,
            summarizer
        )

        print(f"\n=== Group: {label} (sampled {len(sampled)} texts) ===")
        print(summary)

# --- create groups then summarize each group ---
grouped = group_by_labels(about, labels)
summarize_grouped_dict(grouped)
```
{{< /expand >}} 

### Skills 

Here's the top 30 mentioned skills ranked & labeled with their respective number of mentions. There're a couple noise in there ("We", "You") but the most demanded skills are expected like "Python", "CI/CD" and "AWS". Some more niche skills are interesting to see as well, for example, "API", "Git", "NoSQL", "Linux" and "GCP".

![skills](/images/skills.png)

## Key Phrases of Responsibility and Requirements Sections

{{< expand title="Code Block: Clustering and Visualization" >}}
```python
# Get all key phrases
key_phrases_exp = []
for d in data:
    if len(d["key_phrases_exp"]) != 0:
        key_phrases_exp.extend(d["key_phrases_exp"])

# Get embeddings
embeddings = model.encode(key_phrases_exp, normalize_embeddings=True)
n_clusters = 7  # adjust as needed

# Clustering
kmeans = KMeans(n_clusters=n_clusters, random_state=42)
labels = kmeans.fit_predict(embeddings)

# Reducing dimensionality of embeddings
tsne = TSNE(
    n_components=3,
    perplexity=min(5, len(embeddings) - 1),
    random_state=42,
    init="random"
)
embeddings_3d = tsne.fit_transform(embeddings)

# Create dataframe to store embeddings, labels and phrases
df = pd.DataFrame({
    "x": embeddings_3d[:, 0],
    "y": embeddings_3d[:, 1],
    "z": embeddings_3d[:, 2],
    "label": labels,
    "phrase": key_phrases_exp
})

MAX_PER_CLUSTER = 200

df_sampled = (
    df.groupby("label", group_keys=False)
      .apply(lambda g: g.sample(
          n=min(len(g), MAX_PER_CLUSTER),
          random_state=42
      ))
      .reset_index(drop=True)
)

# Visualizing the clusters of key phrases
fig = px.scatter_3d(
    df_sampled,
    x="x",
    y="y",
    z="z",
    color="label",
    hover_data=["phrase"],
    title="3D Clustering of Key Phrases (Sampled)"
)

fig.update_traces(marker=dict(size=5))
fig.update_layout(height=700)
fig.show()
fig.write_html("key_phrases_clusters.html")
```
{{< /expand >}} 

Looking at this visualization is much more interesting in that the clustering is much clearer. The dark, navy blue data points are phrases that stresses high degree of technicality but doesn't include specific terminologies, like "analyze complex technical problems" or "designing a significant system component". The blue purple data points stress about data like "dataset" and "data center". The pink purple points center around specific technical terms like "C#", "Scripting language" and "multi-threading". The purple-ish pink points don't really represent too much meanings, and the orange points are more descriptive like "understanding of" or "learn quickly". The orange yellow points revolves around cloud technologies, and the yellow points are more about the benefits of working in a company, like "different perspectives" and "embrace various identities" or more directly "work onsite 3 days/week".

{{< rawhtml >}}
<iframe src="/embeds/key_phrases_clusters.html" width="710" height="610"></iframe>
{{< /rawhtml >}}

### Benefits

Using similar pipeline as above, following is a visualization of benefits word vectors. This one is even simpler: the yellow points represent retirement plan specifics, the orange points represent further education assistance, the pink points represent health insurance specifics, the purple points specify paid time leaves and the dark blues represent everything else. 

{{< rawhtml >}}
<iframe src="/embeds/benefits_clusters.html" width="710" height="610"></iframe>
{{< /rawhtml >}}

---

This concludes the last part of the backend engineering job analysis. 


