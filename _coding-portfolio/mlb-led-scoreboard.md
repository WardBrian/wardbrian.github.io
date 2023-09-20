---
title: "MLB LED Scoreboard"
excerpt: "Live MLB scores on a Raspberry Pi"
collection: coding-portfolio
---

# MLB LED Scoreboard

The [mlb-led-scoreboard](https://github.com/MLB-LED-Scoreboard/mlb-led-scoreboard) project was
something I stumbled on in 2021. I quickly ordered the parts to build my own, and I started
pulling apart the code to see how it worked, how I could improve it, and so on.

I ended up re-writing large portions of the project. At the time, MLB was changing their
APIs, and the project was using the deprecated API. I also wanted to display more information,
including the current pitcher and batter. This extra information revealed some performance concerns
in the original code which lead to visual hitching while writing the names to the display, so I also
split the code into two threads, one for the display and one for the network requests.

After the original API was turned off in 2022, my fork was merged into the original project, and
I was added as a maintainer.

![MLB LED Scoreboard](/images/code/mlb-led-scoreboard.jpg)
