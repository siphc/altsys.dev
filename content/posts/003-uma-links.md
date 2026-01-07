+++
title = "\"Performance Tuning\" a Gacha Game for No Reason Whatsoever"
date = "2026-01-07"
author = "siphc"
tags = ["random"]
description = "Numbers you say?"
+++

This is a completely unserious post, but if you would forgive me: I promise I have something interesting coming up.

I wanted to write this piece to (slightly) demonstrate how deep the rabbit hole of this innocuous-looking game can go, but this piece would also act as a fast and lightweight reference doc to me. Seriously, Hugo is great, and I would prefer navigating to my own site instead of bookmarking 10 others to find the same piece of information if I had just compiled them together prior.

Also I want to experiment with tables, graphs, and embedded math formulas on my website.

---

## Rerolling

The first step of any serious uma account is a good reroll, or creating new characters over and over until a desirable one is found. This game provides the perfect ingredients for players to do so:
- It takes two clicks to delete one's account;
- It is likewise very easy to create another;
- The game provides a generous amount of pulls upon account creation.

Of course, I wanted to have a good reroll, but not waste my life away seeking an impossibly small chance for *all* the traits that I desire. Let's enlist the help of probability theory!

The relation between # of pulls and probability of some end result can be modeled with a gamma distribution
$$
X \sim Gamma(\alpha, \lambda)
$$
...where $\alpha$ is the *shape parameter* and $\lambda$ is the *rate parameter*. The card we want is at 0.75%, and we want 5 copies of it to "max" it, so $\alpha = 5, \lambda = 0.0075$.

The CDF of such a distribution would look like the following:
$$
F(x) = \frac{1}{\Gamma(\alpha)} \gamma(\alpha, \lambda x)
$$
...where the gamma function $\Gamma(\alpha)$ evaluates to $(\alpha-1)!$ for positive integers, and the lower incomplete gamma function $\gamma(\alpha, \lambda x)$ is defined as:
$$
\gamma(\alpha, \lambda x) = \int_0^{\lambda x}t^{\alpha-1}e^{-t}dt
$$
...lovely. Thankfully we have the help of computer-aided graphical tools:
{{<embed "https://www.desmos.com/calculator/ncdfrvqt3t?embed">}}

But that's not the full story! Every 200 pulls, players can receive the card they desire for free. Updating the formula accordingly simply gives us
$$
\alpha_1 = 5 - \lfloor{\frac{x}{200}}\rfloor
$$
...substituting $\alpha_1$ for $\alpha$ in the original function gives us this lovely graph:
{{<embed "https://www.desmos.com/calculator/wxlzubqm8v?embed">}}

With 200 free pulls that every new character gets, there's a 6.6% chance that we achieve our primary goal of rerolling. Not bad! In future banners, we can also utilize the same graph to determine our chances of getting a card to max.

---

## The Links (What you're here for)	

This game is notorious for having layers of hidden complexity beneath its nondescriptive and often misleading tooltips. Players have been reverse engineering to gather data to fill in the gaps for years, and here's some of what we now have.

Note that my shortcode uses microlink's screenshot API, so the images might look jank.

{{< link-thumbnail url="https://docs.google.com/document/d/11X2P7pLuh-k9E7PhRiD20nDX22rNWtCpC1S4IMx_8pQ/edit?usp=sharing" caption="Reference Doc" >}}

{{< link-thumbnail url=https://euophrys.github.io/uma-tiers/#/global" caption="Support Card Tier List" >}}

## Spreadsheet Gaming:

{{< link-thumbnail url="https://docs.google.com/spreadsheets/d/1oB3eTvKqREtJDWJL0q80O_VjBcpOmRl5xE0z5fZKgFY/edit?usp=sharing" caption="Skills Spreadsheet" >}}

{{< link-thumbnail url="https://docs.google.com/spreadsheets/d/1mT_uH79lZwEth6qGvjBOvrRKxwfKixr6vfhgzoW1s-U/edit?usp=sharing" caption="Spark Procs" >}}

{{< link-thumbnail url="https://docs.google.com/spreadsheets/d/1kpPMEBnFOSkDWRLfHOHT2BjB38YD9Q7oMcuhF2j5V6M/edit?usp=sharing" caption="G1 Race Schedule" >}}

{{< link-thumbnail url="https://docs.google.com/spreadsheets/d/10a9nsrhECdsbvP0b7ZKWd1J95Du98kpTozzeJZvpIdY/edit?usp=sharing" caption="Individual Build Guide" >}}

{{< link-thumbnail url="https://docs.google.com/spreadsheets/d/14-b9KYeXCE-o43LZ_X8-pRqRo_Mnn_9_meAj3Bo6QOo/edit?usp=sharing" caption="Banner Release Estimation" >}}