
## Solution 

### Approach & Analysis

The first step is to look at the initial query data:

We are given that the queries follow an exponential distribution i.e, small queries are much more likely to occur than large queries, and in fact, the largest query that we find is that only 20 or so nodes ever get queried more than twice (from the path_distribution.png). In fact, doing some easy analysis on the detailed_results gives us that the median node is around 6, and no queried node is more than 43. Of course, given enough number of queries, we will see larger numbers appear, but it would take an extraordinarily large number to see a query like 499 appear, since the $e^{\lambda t}$ term will be extremely small.

Another thing we notice, is that in path_distribution frequencies, the median is very front_loaded, i.e, in an ideal world, we sacrifice some of the efficiency of the best performing queries, in order to decrease the overall median.

### Optimization Strategy

Final optimization strategy: Rank queried nodes by how frequently their are queried. For all non-queried nodes (with value v), add an edge with 100% probability to the (v%3)th most queried node (so either the first, second, or third most frequent query). Send frequently queried nodes to queried nodes with rank + 3 (so the first most queried node should have a 100% edge to the fourth queried node etc)... Finally, the 3 least frequently queried nodes (but at least queried once), should cycle between each other creating a "dead-node pool" that kills in an infinite cycle.

### Implementation Details

I created a dictionary to map queried nodes with their frequencies, and used it to sort the nodes by their frequency of query. This allowed me to quickly implement the above optimization strategy. I also added a median finder, because I think that factors of the median finder give us the best results for how many chains we should use. 

### Results

SUCCESS RATE:
  Initial:   80.5% (161/200)
  Optimized: 98.0% (196/200)
  ✅ Improvement: 17.5%

PATH LENGTHS (successful queries only):
  Initial:   566.5 (161/200 queries)
  Optimized: 3.0 (196/200 queries)
  ✅ Improvement: 99.5%

COMBINED SCORE (success rate × path efficiency):
  Score: 612.12
  Higher is better, rewards both success and shorter paths


### Trade-offs & Limitations

I considered the tradeoff between success rate and path length. Increasing the number of chains that I considered would decrease path lengths but increase success rate. For 6 chains, the tradeoff was too much as the success rate actually regressed. 3 chains was a good balance, not having a perfect success-rate, but the median path length was only 3.

One limitation is that for more queries, I might actually get queries that fall outside the current queries that I support (which only go up to 43). I think that it wouldn't significantly impact the score of the performance, since unsupported queries would only decrease the success rate by a neglegible amount (since there is an extremely small < 0.1% chance of this happening because of the exponential nature of the queries). 

### Iteration Journey

[Briefly describe your iteration process - what approaches you tried, what you learned, and how your solution evolved]
My first implementation was simple. Since all the queries are small, send all large nodes to 0, and then cycle through the rest of the possible query nodes, taking an edge from 0 to 1, 1 to 2, etc ... This guarentees that at least we can get 100% success with a time bounded by the index of i within the possible query nodes. I.e, to reach the 6th most commonly queried node we would cycle from *random -> 0 -> 1 -> 2 -> 3 -> 4 -> 5. Our first result for our combined score is roughly 527

For my second iteration, I notice that the query distribution actually isn't monotonically increasing, i.e, 7 is more likely to be queried than 6, so I adjust the cycle accordingly. Our combined score now is now 540. 

Now we try to work on reducing the median time. We can do by utilizing the fact that we have 10 chances to send 0 -> 2 -> 4, and 1 -> 3 -> 5, and equal probability of sending a random node to 0 vs 1. In order for the median to be small, we actually want the process to fail when it doesn't align with the even/odd that we want, so we create "dead-nodes" that just cycle into each other, that the process ends up in if we don't get what we want immediately. This gives us a combined score of 596 which is better.

For my third iteration, I try instead of using even/odd to do mod 3 chains, to see if that also improves. Eventually, I expect that the rate of success for mod n chains is $(\frac{n-1}{n})^{10}$, and the median time is roughly $1 + 6/n$ (this is from the fact that 6 targets take up half of the queries). Doing the math (score calculation is $success-rate \times (\ln (566.5/median) + 1)$, it seems like n=3 should be an improvement, so we try it. However, implementing it, the graph has too many edges (since most of the nodes have 3 edges). We fix this by offloading the responsibility of choosing between 3 vertices to input randomness between the nodes that are never queried, to the random picking of the nodes themselves. Our score here is 621.49, which is a lot better than our naive first implementation.

Finally, we check the n = 6 case to see if that does better. This doesn't score better (515), since the tradeoff between success rate and optimized path length (now 2) is too much. The success rate was 75% while the path length was 2. 

---

* Be concise but thorough - aim for 500-1000 words total
* Include specific data and metrics where relevant
* Explain your reasoning, not just what you did