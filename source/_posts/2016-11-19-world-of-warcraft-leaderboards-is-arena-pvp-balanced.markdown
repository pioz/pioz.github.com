---
layout: post
title: "World of Warcraft leaderboards: is arena PvP balanced?"
date: 2016-11-19 04:04:39 +0100
comments: true
categories: [games, World of Warcraft]
---

I've created a little script in Ruby to grab some data from the World of
Warcraft 2v2 and 3v3 PvP leaderboards. On [Github](https://gist.github.com/pioz/479f6be0a53900f554b8d6a15828e040)
you can find the source code of the script:

{% gist 479f6be0a53900f554b8d6a15828e040 %}

This script parse these leaderboards pages
https://worldofwarcraft.com/en-gb/game/pvp/leaderboards/2v2
and count for every class the number of players. Then generate a pie graph to
see which classes are most frequent in the leaderboards.

If you run the script with the command `ruby wowpvpstats.rb` after few seconds
you can find two png images like these:

![2v2 graph](/images/posts/stats_2v2.png)

and

![3v3 graph](/images/posts/stats_3v3.png)

Now you can easily see which classes are more present in the leaderboards:
*druids*, *shamans* and *monks* dominate the leaderboard while *mages* and
*hunters* suck. So, what do you think? Is this PvP balanced?
