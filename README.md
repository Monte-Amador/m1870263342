# m1870263342

# Debug Notes: m1870263342

These are just the preliminary tests, to try and isolate the problem and understand what's happening with the filter and search function on the Press Release page. I'll continue to update this as I investigate dive in more so that we're all on the same page for tomorrow's meeting.

- The [Press Release](https://www.airlines.org/news/) page initially loads all the custom post types correctly and in order, as expected. All custom posts are delivered chronologically by their following dates: 

```bash
Sept 21, 2021
Sept 09, 2021
August 10, 2021
July 28, 2021
July 14, 2021
July 12, 2021
July 07, 2021
June 21, 2021
June 04, 2021
```

Note that this is the default and expected output from the WordPress loop. The bug happens when trying either the filters or a custom search query I describe in detail below. However, what is strange is that when you put the filters back to their default status (e.g. All Months, All Years, All Topics) you would expect the same default output that the WordPress loop produces but instead we lose the 3 newest posts (Sept-August 2021). This mirrors the output of using the Year filter with a selected value of `2021` that I outline below.



## Tests Using the Month Filters:

- sorting by month "**September** and **August**" only brings up September/August articles from 2019 and earlier, nothing from 2021 nor 2020.
- sorting by month "**July**" brings up only one article from 2021 (July 07, 2021) but brings up the articles from 2020 and earlier
- sorting by month "**June**" brings up 2020 and earlier but nothing from 2021

## Tests Using the Year Filter:

- sorting by the 2021 results in posts starting from July 28, 2021 and then correctly displays the remaining posts in order. We are however specifically, we're missing the 3 most recent news items Sept 21, Sept 09 and August 10 2021 posts respectively.

## Test using the Search Query

- as soon as the search field is used the example articles will disappear, even when you clear the search query. There is no way to see the original page unless you reload the page.

