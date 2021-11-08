## A4A Debug Version 2

[TOC]



## Debugging the featured post Ajax requests.

Since we last met, I've move forward to provide a solution to the featured posts exclusion that we all three went over together last Friday. As I dove into to codebase to discover exactly what part needed to be adjusted, I discovered more inconsistencies along the way and have documented them here in hopes they can provide the path for us to the solution.  

Both search filters on the news and blog pages operate similarly and are therefore having the same problems with the featured posts exclusion code. This makes sense as they are both written out to operate similarly with the exception of the month and date added filters on the news page.

Before I started working on the Featured Posts problem, I also wanted to find out if there were any patterns that would help point me to the right code block in the `custom-functions.php` file by running further tests on using the Ajax filter.

### Test 1: Initial Page Load Results

```bash
# Post Dates Loaded on Initial Page Load: Expected
Nov 06
Sept 21
Sept 09
Aug 10 
July 28
July 14
July 12 
July 07
June 21
```

After pressing the `View More` button:

```bash
# Post dates on 1st subsequent load more posts: Expected
June 04
May 20
May 20
May 18
May 14
May 11
May 05
May 03
April 27
```

And finally:

```bash
# Post dates on 2nd subsequent load more posts: Expected

April 02
March 30
March 22
March 10
February 19
February 02
February 02
January 29
January 22
```
### Test 1 Results: everything works the way we expect to see.

***

### Test 2: Understanding Ajax Filter Problem

Following the initial page load, we can deduce the expected results if you run a 'year' filter with a value of `2021` you expect to see posts as so:

```bash
Nov 06 #featured post
Sept 21 #featured post
Sept 09 #featured post
Aug 10 
July 28
July 14
July 12 
July 07
June 21
```

Instead, the first three posts (featured) are not displayed and therefore we get:

```bash
Aug 10
July 28
July 14
July 12 
July 07
June 21
June 04
May 20
May 20
```
This coincides with the meeting we held and points to the featured posts exclusion as the source of the bug. So to confirm this was the issue I continued loading more results to see how it would load:

Clicking the load more button again, we get the last 3 queried from the previous page load duplicated:

```bash
June 04 # duplicate from previous query
May 20 # duplicate from previous query
May 20 # duplicate from previous query
May 18 
May 14
May 11
May 05
May 03
April 27
```
For me, this was unexpected behavior. It seems the load more query doesn't count the last 3 posts from the previous query. My first inclination was that this was caused by something maybe in `the_offset` from the `custom-functions.php` file. However, *if you press load again the next sequence of 9 posts contain no duplicates from the last query:*

```bash
April 02
March 30
March 22
March 10
Feb 19
Feb 02
Feb 02
Jan 29
Jan 22
```
### Test 2 Results: 
It's as if the block of code (lines 922-938 in `custom-functions.php`) that is in charge of showin and excluding the featured posts doesn't exist when any filter is applied. This would mean the query is just grabbing the initial 9 posts. The problem, it seems, is that the original standard query that runs when the page isn't loading the ajax initially, or subsequent view more loads, but therefore doesn't pick up the featured posts.

So this follows in the comments from `custom-functions.php` as well, that same block of code gives us the correct output during page load and subsequent page loads but not for the filter. **Where is the filter block that works with featured images**?

***

## Featured Posts Exclusion Bug
Going through our team's initial findings of having the featured posts declared on their parent pages `/news/` and `/blog/` respectively, gave us some insight into how the Ajax function hides them while returning results during the initial page load. However I ran some tests in order to isolate the problematic code by first confirming that everything would work as expected if there I simply removed the featured posts from the News page altogether. 

Finally, to ensure these problems were only caused with the featured posts, I simply deleted all featured posts from the `/news/` parent page and returned to the filters/search on the news-events to see the results I was expecting.

### Deleting Featured Posts: The unexpected Results

```bash
# Post Dates Loaded on Initial Page Load: Expected
Nov 06
Sept 21
Sept 09
Aug 10 
July 28
July 14
July 12 
July 07
June 21
```

After removing the 3 featured posts from the News page's ACF rows, the first 3 posts that are being displayed in their larger cards format are no longer the 'featured posts' but are in fact the 3 most recent. The posts are being loaded from the `initial page load` Ajax request.

```bash
# Post Dates Loaded on 1st subsequent page load: Expected
June 04
May 20 
May 20 
May 18 
May 14
May 11
May 05
May 03
April 27
```

```bash
# Post Dates Loaded on 2nd subsequent page load: Not Expected
April 02 # duplicate from previous query
May 20 # duplicate from previous query
May 20 # duplicate from previous query
May 18 # duplicate from previous query
May 14 # duplicate from previous query
May 11 # duplicate from previous query
May 05 # duplicate from previous query
May 03 # duplicate from previous query
April 27 # duplicate from previous query
```

This continues in an endless loop of duplicate posts and we never actually get to anything earlier than April 02. 

***
