# Lesson 2 - Revive corals
### Script: [Revive corals](https://github.com/gnembon/scarpet/blob/master/programs/survival/revive_coral.sc)


In this tutorial lesson, we will look at the `revive_coral.sc` script by Ghoulboy.
When this script is loaded, right-clicking on a dead coral with a water bottle will revive the coral and, if the player is in survival/adventure mode, deplete the bottle.
Much like the previous script, it is very simple, this time using a simple event to allow the player to perform a basic interaction with the world.

## Event information
The basic underlyings of events where covered in the previous lesson, and the event used is the same: `__on_player_right_clicks_block`.
The arguments of this event are also the same as in the previous lesson: `player`, `item_tuple`, `hand`, `block`, `face`, and `hitvec`.

## Script
The script itself is very simple, consisting of just the one event declaration.
In it, there is an `if` statement, which checks for the conditions of a water bottle right-clicking on a dead coral block.

## `if` statement
The condition of the `if` statement consists of 3 parts. The first, `item_tuple:0=='potion'`, checks if the item is a potion item, since all potions (including water bottles) have the id `potion`.

The second condition checks whether or not the block being targetted is a dead coral.
The check, `block ~ '^dead_\\w+_coral_block$'`, makes use of the `~` operator.
This is an operator used in many ways throughout the scarpet programming language.
In this case, it is being used to check for the regex (regular expression) string `'^dead_\\w+_coral_block$'`, which checks whether the block id starts with the string `'dead_'` and ends with `'_coral_block'`.
It would have been possible to create a list of all the ids of dead coral blocks and check against those, but this is roughly as fast.
More information on regular expressions can be found in the Extra Materials section below.

The third condition, `item_tuple:2:'Potion' == 'minecraft:water'`, checks whether or not the item the player is holding, by now confirmed to have the item id of `'potion'`, is a water bottle.
This is done by checking the nbt of the item, which is the third item in the `item_tuple` list.
Accessing the nbt via `item_tuple:2`, we then look further within the nbt at the `'Potion` tag, which is used to store the type of potion, and check to see if it is `minecraft:water`.

## Reving the coral

Once the conditions have been met, and we have confirmed that the player has right-clicked on a dead coral with a water bottle, we can begin the task of reviving that coral.

Line 7 stores the position of the block in a variable `position`, for future use, and line 9 sets the block in that position, currently a dead coral, to the live version of the block.

Note how instead of mapping each dead coral block id to its live version, we simply remove the `dead_` prefix from the block id, and in this case the `block` variable is interpreted as a string, allowing for string manipulation like this to work.

Line 10 contains another `if` statement, this time checking whether or not the player is in creative mode, by using the `~` operator with the `player` entity to check for the `'gamemode'` property.
This is done because we do not want to deplete the water bottle for creative mode players, only for survival and adventure mode players.

Within the statement checking for creative mode, we have another `if` statement at line 12, this time checking in the item count (stored as the second item in the `item_tuple` list) is greater than 1.
In regular vanilla survival, this should never be the case, since water bottles are unstackable items, and should only stack to a count of 1.
However, as the comment graciously left by gnembon in line 13 points out, the player might have used commands to acquire a stack of more than 1 water bottle.
In this case, we first deplete the stack count by 1 in line 14, and then spawn an empty glass bottle on the ground besides the player, with a pickup delay of 0 ticks, using the `spawn()` function, which will be covered in more detail later on.
This is done to simulate roughly what happens when the player fills a bucket from a stack of multiple buckets, and so that the empty bottle goes into the correct place in the player's inventory.

In the case that the player does not have water bottle stacked beyond 1 (as is the case in regular vanilla survival gameplay), we replace the item in the player's hand from a water bottle to an empty glass bottle with a call to the `inventory_set()` function.
The specific call used is `inventory_set(player,if(hand=='mainhand',query(player,'selected_slot'),40),1,'glass_bottle')`.
The first argument of this function specifies the player to target.
The second is an `if` statement, and if the hand used was the main hand, it searches for the slot containing the water bottle, which is the player's currently selected slot.
If not, and if the player right-clicked with a water bottle from the offhand, the `if` statement returns 40, which is the index of the offhand stlot within the inventory.
The third argument to the `inventory_set()` function is the count of the items to be placed, in this case 1, since we want to place a single empty bottle in that slot.
The last argument is the id of the item to be placed in the slot, in this case `glass_bottle`.

**NB: in calls to `inventory_set`, as well as with certain other functions in scarpet, the `'minecraft:'` prefix is not required in front of item and block ids. Check what is required by your function before using. Modded items and blocks always need their prefixes in front of them, however.**

## Correcting for vanilla behaviour

Once we have set the empty glass bottle, it might seem like it is over, but unfortunately not yet.
Now that we have depleted the water bottle  set the revived coral block in its position using the `set()` function, the newly placed block will not be updated properly.
This is because coral blocks do not die as a result of random ticks, but as a result of block ticks scheduled on themselves randomly.
These block ticks are scheduled every 60-100 ticks, and check whether or not the block is surrounded by water, in which case it dies, and if it is alive, it schedules another block tick 60-100 ticks in the future.
In order to re-enable this behaviour, we have line 21, which uses a call to the `schedule()` function: `schedule(60+rand(40), _(outer(position)) -> block_tick(position))`.
This is a function which takes as its first argument a delay in ticks, which in this case is `60 + rand(40)`.
`rand(40)` is a function which returns a random number (a double, not an integer) evenly distributed between 0 and its argument, in this case 40.
It can be used with more argument, for that see the function in the scarpet docs.
When used in `schedule()` however, any trailing decimals are ignored, so `60 + rand(40)` is interpreted as an integer.

The second argument to `schedule()` is either the name of a function to be called after the delay, or in this case, a lambda function with no arguments.
This is done so that the function itself becomes a variable which can be called separately later on, after the specified delay, and allowing the game to progress as usual.
More information on lambda functions can be found in the Extra Materials section below.

The lambda function itself is simple: `_(outer(position)) -> block_tick(position)`.
Although it may seem that this lambda function does, in fact, have an argument, the `outer()` function simply takes the `position` variable declared earlier on in line 7, and makes it accessible within the lambda function.
This is becasue otherwise, seen as though the lambda function is running separately, it does not have access to the local variables in the scope of the function that called it.
In scarpet, all variables are restricted to their scope, like in any programming language, except for global variables, which are denoted by the prefix `global_`.
These can be accessed anywhere throughout the script, and are useful for storing important information, or flags which need to be visible anywhere within the script.
More information about function and variable scopes can be found in the Extra Materials section below.

The lambda function calls the `block_tick()` function at the position of the newly placed coral block.
This will allow it to check its surroundings for water blocks, as well as start up its own cycle of checking, without further need of programmer intervention.


## Conclusion
Thank you again for reading this scarpet tutorial lesson.
Hopefully, this lesson has explained how a simple script can, in an inobtrusive way, allow to easily code in a simple modification to gameplay.
This lesson should also show how sometimes scarpet scripts have to take various considerations into account, such as the possibility of stacked water bottle.

Also to be noted is the impact of scarpet functions on regular behaviour, and the need to, in some cases, artificially start vanilla behaviour back up.
With blocks, this can usually be acheived with a well-timed block tick, such as in this case, but each case is individual and will likely require a different solution.

## Extra Materials:
 - Video link: 
 - Regular expressions: 
 - Lambda functions: 
 - Function and variable scopes: 
