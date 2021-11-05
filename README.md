# Debug Notes: m1870263342

These are just the preliminary tests, to try and isolate the problem and understand what's happening with the filter and search function on the Press Release page. I'll continue to update this as I investigate dive in more so that we're all on the same page for tomorrow's meeting.

- The [Press Release](https://www.airlines.org/news/) page initially loads all the custom post types correctly and in order, as expected. All custom posts are delivered chronologically by their following dates before the ajax call: 

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
Note that this is the default and expected output from the WordPress loop. The bug happens when trying either the filters or a custom search query I describe in detail below. However, what is strange is that when you put the filters back to their default status (e.g. All Months, All Years, All Topics) you would expect the same default output that the WordPress loop produces but instead we lose the 3 newest posts (Sept-August 2021) and then the rest of the posts show up. This mirrors the same output of using the Year filter with a selected value of `2021` that I outline below.

## Tests Using the Month Filters:

- sorting by month "**September** and **August**" only brings up September/August articles from 2019 and earlier, nothing from 2021 nor 2020.
- sorting by month "**July**" brings up only one article from 2021 (July 07, 2021) but brings up the articles from 2020 and earlier
- sorting by month "**June**" brings up 2020 and earlier but nothing from 2021

## Tests Using the Year Filter:

- Sorting by the 2021 results in posts starting from **July 28, 2021** and then correctly displays the remaining posts in order. We are however specifically, we're missing the 3 most recent news items **Sept 21**, **Sept 09** and **August 10** **2021** posts respectively.

## Test using the Search Query

- as soon as the search field is used the example articles will disappear, even when you clear the search query. There is no way to see the original page unless you reload the page.



## What I've Investigated So Far

I went through the various News custom post types and added "Document Type" and "Topics" to posts that weren't showing up and changed their meta values in the back end to match the posts that were showing up, but this didn't give any joy, nothing changed. So I went through the flow of how the content gets displayed on the News page via the codebase and am now sitting on 2 main files and 2 companion files where I think the problem may lie. However, I'm not familiar with the way `wp-admin/admin-ajax.php` in `wp-settings.php` operates so that's why I'm checking in with the both of you to make sure I'm on the right path.

There are two main files I am focusing on: 

1.  `theme.js` (located: `wp-content/themes/airlines2020/js/theme.js`) 
2.  `custom-functions.php` (located: `wp-content/themes/airlines2020/inc/custom-functions.php` ). 
   In particular, the `custom-functions.php` has the WordPress queries for Month & Year as well as Topic further down. It also works with the `$initial_load` which looks like its sets `offset` as an object to the `$args` array? Is this offset what is possibly throwing off the defaults after you clear the search and filters?

There are two others that I'm checking out too:

1. `ajax-response-news-posts-initial.php` (located: `wp-content/themes/airlines2020/global-templates/ajax-parts/ajax-response-news-posts-initial.php` ) displays ajax response from NEWS post queries and is built using a specific order for the masonry grid.

2. `ajax-response-news-posts-paged.php` (located: `wp-content/themes/airlines2020/global-templates/ajax-parts/ajax-response-news-posts-paged.php`) displays ajax response from NEWS if there is more than 1 paged content.

I'm wondering if there's just an array object that needs to be adjusted in the `$args` variable that gets passed to the `WP_Query` in the `custom-functions.php`file. I thought maybe it was the `$offset` being declared on line 1159 so I tried to comment that out just to see but that too brought no joy. 

Hopefully this will help and I can get a little insight to sort out the issue. This has been a good codebase walkabout mixed in with some debugging context so it's been helpful to see the way this theme was built. But this was from an inherited theme from another developer is that correct?

Anyways, I'm looking forward to meeting up and going through the glitch.

