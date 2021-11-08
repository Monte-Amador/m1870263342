## A4A Debug Version 2

[TOC]



## Debugging the Ajax requests on the News Page

### Problem Context

Since we last met, I've moved forward to provide a solution to the featured posts exclusion that all three of us went over together last Friday. As I dove into to codebase to discover exactly what part needed to be adjusted, I discovered more inconsistencies along the way and have documented them here in hopes they can provide the path for us to the solution.  

Both search filters on the news and blog pages operate similarly and are therefore having the same problems with the featured posts exclusion code. This makes sense as they are both written out to operate similarly with the exception of the month and date added filters on the news page.

Test 1 works as expected all the way through the final page load.

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
### Test 1 Results: Expected Results

***

### Test 2: Isolating Ajax Filter

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

#### Inconsistent Output

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
For me, this was unexpected behavior that helped shed a little light into the query Ajax is delivering.. It seems the load more query doesn't count the last 3 posts from the previous query. My first inclination was that this was caused by something maybe in `the_offset` from the `custom-functions.php` file. However, *if you press load again the next sequence of 9 posts contain no duplicates from the last query:*

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

Here is the block of code (lines 922-938 in `custom-functions.php`) that seems to only work on initial page load and subsequent load-mores with no filtering - as stated in the code's comments. It's as if the  that handles the showing and excluding of the featured posts doesn't exist when any filter is applied.

```php
$the_offset = intval(9);

  if ( ! empty( $featured_ids ) ) {
    //only on initial page load
    if ( true === $initial_load ) {
      $IDS            = $featured_ids;
      $posts_per_page = intval( 9 - $total_featured );
      $post__not_in   = $exclude_featured_ids;
    } else {
      //only on subsequent load-mores w/ no filtering
          $initial_posts_per_page = intval(9 - $total_featured);
          $the_offset     = intval( $initial_posts_per_page + ( ($paged - 2) * $original_posts_per_page ) );
      if ( empty( $query ) && empty( $topic ) ) {
        if ( $paged > 1 ) {
          $post__not_in = $exclude_featured_ids;
        }
      }
    }
  }
```

If a user supplies a filter (e.g. year) and the ajax query loads the results, it seems that is not being treated the same as an initial page load. **Is there a filter function similar to the initial page load and subsequent load-mores function that also includes/works with featured images**? If this is true and If there isn't a filter function that handles the featured posts, then I think this code block is where the solution would need to be implemented. By adding a conditional for when the user uses a filter.

This is the solution path I'm on at this moment. If after considering this you think it's the right path, I can set out to take a stab at writing the conditional and inserting it but more than anything I wanted to communicate what I've tested and come to understand. 
