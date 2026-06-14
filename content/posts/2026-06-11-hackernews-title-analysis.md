---
title: "Hacker News Title Analysis"
date: 2026-06-11T07:00:00+08:00
draft: true
---


# Intro

[Hacker News](https://news.ycombinator.com) is a forum popular within the startup community where people share their opinions and products. I like the venue because it frequently surfaces fun and niche insights about the trends within the community. I encountered this dataset on kaggle and thought it'd be interesting to analyze the titles. Unfortunately, this dataset doesn't countain anything about the real post.  


# EDA


```python
df = pd.read_csv("/kaggle/input/datasets/elvisbui/hacker-news-top-stories-2020-2026/hacker_news_top_stories_2020_2026.csv")
df.head()
```

|       id | title                              | url                                                 | domain            | author     |   score |   num_comments | created_at                |   created_at_i | story_type   |   year |
|---------:|:-----------------------------------|:----------------------------------------------------|:------------------|:-----------|--------:|---------------:|:--------------------------|---------------:|:-------------|-------:|
| 21927248 | Rewriting m4vgalib in Rust         | http://cliffle.com/blog/m4vga-in-rust/              | cliffle.com       | zimmerfrei |     372 |             44 | 2020-01-01 08:38:20+00:00 |     1577867900 | story        |   2020 |
| 21927322 | What Makes a Successful Founder?   | https://www.basisset.ventures/founder-superpowers   | basisset.ventures | yarapavan  |     109 |             46 | 2020-01-01 09:07:52+00:00 |     1577869672 | story        |   2020 |
| 21927427 | I'm not feeling the async pressure | https://lucumr.pocoo.org/2020/1/1/async-pressure/   | lucumr.pocoo.org  | pauloxnet  |     337 |            100 | 2020-01-01 10:00:04+00:00 |     1577872804 | story        |   2020 |
| 21927473 | Hard deadlines are not user-first  | http://jatins.gitlab.io/me/why-deadline/            | jatins.gitlab.io  | jatins     |     467 |            246 | 2020-01-01 10:22:42+00:00 |     1577874162 | story        |   2020 |
| 21927773 | What Happened in the 2010s         | https://avc.com/2019/12/what-happened-in-the-2010s/ | avc.com           | notlukesky |     167 |             90 | 2020-01-01 12:26:04+00:00 |     1577881564 | story        |   2020 |


We can see that besides the title there're also score, number of comments, created at and other attribute. But here we'll ignore them for now.

#