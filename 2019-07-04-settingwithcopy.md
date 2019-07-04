---
title: "The SettingWithCopy Warning: Python Pandas"
date: 2019-07-04
---

This post is essentially just a little note for myself because I have encountered this warning a lot and I use Pandas quite a bit. Essentially I pulled all the information from this [DataQuest post](https://www.dataquest.io/blog/settingwithcopywarning/). I always find that writing things down helps me to retain knowledge, so I thought I'd make a post about it ! Hopefully this is helpful to someone 

The `SettingWithCopyWarning` is a *warning* not an error. This means that the code will still run.

## Chained Assignment
Pandas generates this warning when it detects *chained assignment* to understand chained assignment we need to understand a few basic concepts:

* Assignment: Operations that *set* a value of something
* Access: Operations that *return* the value of something
* Indexing: Referencing a subset of the data e.g. `df[1:5]`
* Chaining: Using multiple indexing operations e.g. `df[1:5][0]`

Chained assignment is the combination of chaining and assignment. So essentially this `SettingWithCopyWarning` pops up when we try to assign a value to some chained subset of the data.

Some actions in Pandas will return a **view** of your data whilst others will return a **copy**. Suppose I have a some DataFrame called `data`. If I *view* a subset of this data, I can call `data[1:5]`. Now this object `data[1:5]` is not a 'new' object, it is simply the subset which exists within `data`. If I want to create a copy of this subset I can call `data[1:5].copy()`. This distinction is important because any operation conducted on `data[1:5]` will also change `data`, however an operation on `data[1:5].copy()` will not.

## How to fix the problem

Here is an example of the error coming up when I was working on my honours thesis. I have a panel dataset named `panel` which I had read into the kernel as a `pd.DataFrame`. I wanted to work with only observations that were taken at the 'midline' period. From this, I wanted to loop through the DataFrame to calculate some village level statistics. My code looked like this:


{% highlight python %}
midline = panel[panel['survey_round'] == 'Midline']

midline['adopted'] = np.where(midline['s7_2_0'] == 'Yes', 1, 0)
midline_treated = midline[midline['treatment'] ==  1]

props = []
ttl_adopt = []

for village in villages_list:
    no_hh = midline_treated[midline_treated['villagecode'] == village]['treat_hh'].sum()
    n = len(midline_treated[midline_treated['villagecode'] == village])
    adopt = midline_treated[midline_treated['villagecode'] == village]['adopted'].sum()
    ttl_adopt.append(adopt/n)
    props.append(no_hh/n)

{% endhighlight %}

Now I recieved a `SettingWithCopyWarning` and the problem lied within the first line `midline = panel[panel['survey_round'] == 'Midline']`. The issue here is that Python is the DataFrame `midline` is a *view* of the original dataframe. That is any operation conducted on `midline` would also be conducted on the original `panel` DataFrame. The chained assignment problem then comes up when we try to assign a new column to midline: `midline['adopted'] = np.where(midline['s7_2_0'] == 'Yes', 1, 0)`. In practice, I get the output I want, but it is not good coding practice to not address these warnings because they are pretty serious ! The fix is quite simple...

All I need to do is change the first line to: `midline = panel[panel['survey_round'] == 'Midline'].copy()`. This line of code tells Python that the object `midline` is now a *copy* of the subset of the original `panel` DataFrame we are interested in.  