---
layout: post
title: "Godot Architecture"
subtitle: "Keep your code out of your node"
date: 2025-12-03 08:00:00 +0100
categories: gamedev
---
I propose to have a flat node hierarchy, as much as possible. More importantly, I prefer to have as little logic in nodes. Preferably none at all. Let me explain why.

# Traditional Godot architecture

Godot is all about Nodes. It's the gateway to all of the engine's beautiful features. Most tutorials and examples will have you attach scripts to these Nodes (or Scenes) to build your game. So you might have a Player scene with a Player script that controls its underlying Node tree. This feels clean and modular. And it's easy to reason about.

Each Node behaves independently, and pretends the outside world doesn't exist. So there are then many ways to make Nodes interact with one another in ways that encourage clean code. This is a very sensible approach at first, but you may have already run into some issues.

Rather quickly, you will end up in situations where you are emitting signals up a tree, through multiple levels, until it reaches the place that needs it. Or you need to pass down a reference to some object multiple levels down this tree. Again, this feels clean and modular. But I would suggest that this arbitrary modularity is actually holding you back.

It becomes especially painful when two separate Nodes, say, a vehicle and a player, need to interact in complex ways. Some player code needs to remain functional, like inventory management and looking around. But jumping in a vehicle is not allowed. Instead, the car needs to brake. This can result in some gnarly code that goes into both Nodes. Of course there are ways around this. But those are patches you don't need if you shift your mental model.

Allow me to present an alternative approach.

# A different approach

Essentially there are three components to your game:
- State
- Logic
- View

We'll dive into more detail on each of those later, but in one sentence, this is the mental model:
>"Logic operates on State, and View represents that State."

If you're familiar with common software engineering patterns, you will probably know this as MVC, or Model-View-Controller. In a sense, that is what I'm suggesting. But games are a totally different beast than business software. So it's only similar in spirit.

## Lifting code out of the Node

The main thing about this approach is that you start out with all game logic in a single script. Sure, split it out later. But at first, that single file will control all Nodes and Scenes in your game in top-down way. So all of that code in your Player script? Put it in that top-level script instead.

Next, do the same thing for your state. That `player_node.health` you are manipulating? Not allowed. It lifes in that same top-level script. Even better: put all state in a separate object. So updating health becomes `state.player_health += 10` followed by `player_node.set_health_bar(state.player_health)`. Use this approach for *everything*.

At some point, you will end up with a gigantic top-level script. But you should be able to easily parse out all the patterns that appear in that file. Factor them out into separate scripts if you feel like it. But whatever you do, *do not* move the code into Nodes.

# What it looks like

Here's a glimpse of what this architecture might look like in code. Mind you that this is a really "zoomed-in" showcase of the architecture. It ignores all boilerplate a game might have, like a Main class and such.

## State

**In short:** We want one single source of truth: State. Essentially this is an in-memory save file.

All *persistent* game state goes into a single State class. To be more specific:
- Example data: all players and their positions, health, items, etc.
- In short: this contains all data that you want to be able to save/load
- Additionally, if your game is multiplayer, this is all data that is synced among clients
- Does *not* include ephemeral data that you do not care about losing after loading from save
- Does *not* include static data (resources), like the map representation, enemy stats, etc.

This is an example of what it might look like:
```gdscript
class_name State extends Resource

var player: Player # position, health, mana, xp, level
var items: Array[Item]
var current_map: String # example: "dark_citadel" or "friendly_town"
```

## Logic

**In short:** Logic operates on State. This would be your whole game if it had no audiovisual representation.

It contains top-down logic. It changes State, and then calls View to make sure all nodes align with said State. In essence, it looks like this:

```gdscript
class_name Logic extends RefCounted

var state: State
var view: View

func update():
	update_player(state)
	update_npc(state)
	update_monsters(state)
	
	view.update(state)

func update_player():
	state.player.position = lerp(...)
	
	if state.player.xp > 99:
		state.player.level += 1
		state.player.xp = 0
		view.play_level_up_animation()

func update_npc(): ...
func update_monsters(): ...
```

>Tip!
>Do not use `_physics_process()` or `_process()` for game logic. Just roll your own `update()` function so you have full control over when updates happen.

## View

**In short:** Relatively dumb representation layer that renders State.

This is likely a Scene where all your Nodes live. It can feed all engine hooks into your Logic class through signals. I would consider the View component optional, however, as it just becomes a barrier to access your Nodes. But I want to discuss it regardless, because it helps with adopting the mental model of this architecture.

It could look as simple as this:
```gdscript
func update(state: State):
	ui.xp = state.player.xp
	ui.level = state.player.level
```

And sometimes you might need some specific functions to call based on events:
```gdscript
func play_level_up_animation():
	ui.level_up_animation.play()
```

So, if you have a game with multiple game modes, for example, then this View component can be nice to have, as it separates all the presentation code from your Logic class.

# Considerations

With this approach, saving and loading becomes as trivial as this:

```gdscript
class_name Logic extends RefCounted
# Does not have to be in the Logic class, but it gets the point across

var state: State

func save_game():
	var file := FileAccess.open("user://save_game.dat", FileAccess.WRITE)
	var state_as_string: String = state.to_string()
	file.store_string(state_as_string)

func load_game():
	var file := FileAccess.open("user://save_game.dat", FileAccess.READ)
	var file_content: String = file.get_as_text()
	var new_state: State = State.from_string(file_content)
	
	state = new_state
	view.setup(state)
```

But this also raises the question: "What happens to my nodes when I swap out state like that?" Let's explore some options. First, we will need some way to build up the initial node representation. This is as simple as:

```gdscript
class_name View extends Node

func setup(state: State):
	var player: Node = preload("res://player.tscn").instantiate()
	player.position = state.player.position
	player.health = state.player.health
	player.level = state.player.level
	add_child(player)
	
	for monster in state.monsters:
		var monster: Node = create_monster(monster)
		add_child(monster)
```

# Key takeaways

- Make a clear distinction between logic (just data & functions), and representation (nodes)
- Only put code in nodes when it's about presentation, or to expose Godot's built-in features to your game's logic
- Treat your state as something you can fully replace at a moment's notice, e.g. load game