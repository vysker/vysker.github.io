---
layout: post
title: "Trucking Traders"
#subtitle: "Truck, trade, tussle"
date: 2025-03-07 08:00:00 +0100
categories: gamedev
---
Welcome to the first blog post!
This blog will mostly feature the game I've been working on.
But I'm rambly, so I'll probably also dive into other topics like code and (game) design.
We'll see where it goes.

First, I want to introduce my most recent project:
Trucking Traders (working title).

Trucking Traders is a multiplayer game where you compete to:
- **Trade** items between towns for the best prices
- **Battle** creatures to protect your cargo during transport
- **Win** by completing objectives for points, which double as "perks" when scored

The game's design draws a lot of inspiration from Battle Brothers for its economic simulation,
and Mechabellum for the combat.
Besides those titles, I also use a ton of modern boardgame design sensibilities,
which tend to cleverly interweave systems to create deep strategies from relatively simple rules.

The game is currently in pre-alpha, so it is still very rough.
Expect loads of placeholder art, debug-related visuals, messy UI, etc.

## Gameplay

Trucking Traders consists of three core systems: trade, combat, and objectives.

### Trade

Trading is the very core of the game.
Everything revolves around items and trading.
Different towns will have different item prices.
These price changes are caused by events, which occur periodically.
So you will constantly be traveling between towns to find good deals.

<video width="100%" muted controls>
    <source src="{{site.media-path}}/trade.mp4" type="video/mp4">
</video>
<small><i>This is what trading and navigating around looks like. The third town has an event that affects prices for turrets.</i></small>

Most items also have a use outside of trading.
For instance, fuel is used for increased movement speed, and scrap for repairs.
This makes each item intrinsically valuable, regardless of its monetary value.

### Combat

Once you've bought some goods, you'll want to transport them.
But on the road, you'll encounter creatures trying to ambush you.
Combat makes use of items in your cargo.
So if you're carrying a turret, you can deploy it.
Before combat starts, you place your units on a grid.
Once the battle starts, you have no control over it.
Units automatically attack the nearest target.
So the strategy is to visualize the battle before it starts.

<video width="100%" muted controls>
    <source src="{{site.media-path}}/combat.mp4" type="video/mp4">
</video>
<small><i>Example of unit placement and combat. The pink cube is an "ambusher", which triggers combat when touched.</i></small>

Any damage units take during combat will persist.
Since these units return to your cargo after combat, you'll have damaged goods.
Even worse, if your truck gets hit during combat, it damages all items in your cargo.
And damaged goods do not sell well.
So you are incentivized to either fight well or repair/heal your units cheaply.

### Objectives

You'll be doing all of this trading and battling to score points.
The first player to score 20 points wins the game.

You score points by completing objectives.
More difficult objectives yield bigger points.
And, perhaps more importantly, completed objectives give you rewards,
which are things like "your units deal 20% more damage" or "increase cargo space by 2 slots".

<img src="{{site.media-path}}/objectives.png" width="100%">
<small><i>Preview of objectives and rewards. There will be several categories to objectives for, each with their own rewards.</i></small>

You can score any objective at any time, and you get to pick the reward.
This adds a deep strategic layer to the game that allows for many ways to victory.
It is like a tech tree and victory condition rolled into one.

## Other features

There are several features to enhance the core systems.
These could be things like:
- Unlockable map shortcuts - get to certain towns quicker
- Faction reputation - work towards better prices and unique items with specific vendors
- Trade contracts - basically quests
- Bounty hunt contracts - battle (elite) creatures for rewards
