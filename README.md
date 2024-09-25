<img src="https://github.com/OffworldIndustries/SquidBots/blob/main/SquidBots_Logo_Square.png" width="250">

# SquidBots
Welcome to SquidBots! SquidBots is a plug-in for Offworld games, distributed with Squad and the Squad Editor. Anyone can add to this documentation (just make a pull request).

## History

SquidBots started off as a tool for making Squad videos. The existing workflow of pre-recording your Squad character in PIE and saving out the animation was too time consuming, so I set out to automate some of the processes. The systems created let you entirely keyframe a soldier:

[Squad Animation System](https://www.youtube.com/watch?v=Hsl5ra6ZXPk)

[Keyframed Killhouse](https://www.youtube.com/watch?v=bg5m2shm6u4)

[Turrets Demo](https://www.youtube.com/watch?v=_tUPQV_SIwk)

These allowed for the creation of videos like:

[Welcome to Squad](https://www.youtube.com/watch?v=jOxBIEK172I)

[The Second Emu War](https://www.youtube.com/watch?v=wms8ZEtVQhg)

The systems became extensive enough that I started messing around with having systems to automaticallly keyframe soldiers: The first test of which was the [Squad Tower Defence mod](https://steamcommunity.com/sharedfiles/filedetails/?id=2845442529). Next, I experimented with using behaviour trees and pathfinding to make complex solider AI. The final hurdle was redesigning the system to be replicated (supporting multiplayer), and with those hurdles completed came the first release of the [SquidBots mod](https://steamcommunity.com/sharedfiles/filedetails/?id=2891780963). SquidBots is designed to be somewhat game-agnostic, and had been ported with ease to [Post Scriptum (now Squad44)](https://steamcommunity.com/sharedfiles/filedetails/?id=2909251852).

SquidBots was acquired by Offworld, and has now been integrated into Squad's base game.

# General Usage

SquidBots is designed to work in networked games (client), and offline (standalone). When testing in editor, it is recommended to play in standalone.  

![PIE Settings](https://github.com/user-attachments/assets/aaf5d2f4-d416-454c-91be-17d9a9e955e9)

## Editor Hanging?

There is currently a bug in the SquadEditor that fully automatic sounds can lock up your editor forever without crashing. If you have no need for authentic full auto sounds, it is recommended you disable `Can Use Full Auto Sounds in Editor` in the Editor Preferences. This will replace full auto sounds with semi auto sounds.  

![SQSquidBotsEditorSettings](https://github.com/user-attachments/assets/f929aefe-e777-4422-a18d-5037174a9957)

## Performance

SquidBots has not yet been optimised for performance. It’s almost all written in Blueprint, and the origins of the mod comes from wanting a high fidelity animation system for cinematics \- the bots are doing lots of potentially unnecessary stuff for general gameplay. The current limitation to bot counts is client performance, and I’d recommend avoiding more than 70 bots existing on a client for now. On a server, the limits are a bit higher, somewhere between 100-200.

When playing in a real game, with a client connected to a server, SquidBots benefits from the AI logic being processed by the server, and the animation logic being handled by the clients. But when playing offline, or testing in editor, it’s all having to be processed by one machine, worsening performance.

When playing offline, \~50 bots would give you acceptable performance (50 bots concentrated around the player can feel more full than 100 players scattered around a 8x8km map).

And when testing in editor, there’s lots of engine debug systems running in the background, again slowing down performance. I’d recommend lowering your bot counts in editor to whatever your pc can handle (for one of my PCs, that’s around 20, for another it’s around 40). (If testing seed layers, use [`AdminSetSeedTargetPlayerCount 10`](https://github.com/OffworldIndustries/SquidBots/blob/main/README.md#seeding-bot-counts))

# Navmeshes

## Navmesh Generation

For bots to move around, SquidBots requires the gameplay layer to have a navmesh. Due to an engine bug, the navmesh must be generated in the gameplay layer, not in the Geo Level (which annoyingly means duplicate navmeshes across layers). Hopefully this will be fixed in Squad in the future, were we to add navmeshes to all the vanilla geo levels.

To build a navmesh, drag-and-drop a `NavMeshBoundsVolume` into your world.

![NavMeshBoundsVolume](https://github.com/user-attachments/assets/092901f9-9a60-4f05-a8d7-353b2ee21e8d)

I believe the `NavMeshBoundsVolume` can be placed in the Geo Level, but then you must manually generate the NavMesh in the gameplay layer.

You can then scale your `NavMeshBoundsVolume` to the size of your playspace.

![NavMeshBounds In World](https://github.com/user-attachments/assets/8920274b-eaf9-4a01-9f6c-a7414d0911f0)

Bots are currently unaffected by Squad Map boundaries, so make sure to only have this cover the area you want bots to run around in.  
With editor default settings, the navmesh should then be automatically generated. For more manual control of this, you can disable `Update Navigation Automatically` in the editor preferences. To then manually generate the navmesh, type `RebuildNavigation` in the editor console (`~` key).

![UpdateNavigationAutomatically](https://github.com/user-attachments/assets/121a5eff-e85d-41ed-9cac-b089e17dda03)

![RebuildNavigation](https://github.com/user-attachments/assets/8591381b-209b-4be7-8e8f-f9984142ea05)

Note that the automated navmesh generation is asynchronous, so doesn’t lock up the editor and can be cancelled, but `RebuildNavigation` is synchronous, locking up the editor and non cancellable.

Once the navmesh has been generated, you can hit P in the editor viewport and the navmesh will be displayed.  

![NavMesh](https://github.com/user-attachments/assets/2b2edad6-bf29-4a62-ae20-3e58047b7c88)

If this isn’t being generated, I’d recommend looking into the [Unreal Engine documentation](https://dev.epicgames.com/documentation/en-us/unreal-engine/navigation-system-in-unreal-engine) as Squad is using the Engine’s navigation system out of the box.

When generating automatically, it’s advised that you disable the Navmesh Visualizer (hit P again), as generating with the visualizer enabled drastically worsens performance and slows down generation.

## Navmesh Tuning

Squad has project defaults defined for its navmeshes, tuned for the best results on the existing \~25 squad maps. If you want to manually tune these (which I wouldn’t recommend as it’s a rabbit hole), modify the navmesh settings in the generated `RecastNavMesh_Default`.  

![RecastNavMesh](https://github.com/user-attachments/assets/4fee79be-e6c5-46d2-9f19-8a4d3a42fcba)

You can also modify these in the Project Settings, but note that these apply globally, and for modders will be wiped whenever the SDK updates.

![NavigationMeshSettings](https://github.com/user-attachments/assets/c9a7f1c8-61fd-4d2f-a2b8-a30e60e11bed)

There are WorldSettings related to AI (`Enable AISystem`, `NavigationSystemConfig`), however they are overridden to use Squad’s defaults, unless `Enable Custom Navigation System Config` is enabled. So I would recommend not touching this.  

![WorldSettings](https://github.com/user-attachments/assets/20d0b5c3-44a1-4b97-be22-a6296c002876)

## Troubleshooting Navmeshes

### Your Navmesh isn’t showing up?

Oftentimes you might need to reload your world, or potentially restart your editor. Sometimes your navmesh can be entirely corrupted, so you may need to delete your existing `RecastNavMesh_Default`, then force `RebuildNavigation` again. Sometimes that doesn't even work, so you may need to move your `NavMeshBoundsVolume`, or delete and replace your `NavMeshBoundsVolume`.

### Your Navmesh isn’t generating on the landscape?

Ensure that `Used for Navigation` is enabled on the landscape.

![Landscape](https://github.com/user-attachments/assets/7032739a-7187-41cf-a887-44d016426d16)

## Team Restriction Zones

Areas of the navmesh can be marked as restricted to a team. Placing a `NavModifierVolume` in the world, or adding a `NavModifier` component to an existing actor, you can use the `Team1RestrictedZone` or `Team2RestrictedZone`. The restriction zones work that only Bots with Team X can use that section of the navmesh for pathing. (Red \= Team 1, Blue \= Team 2). In vanilla Squad, the `Gameplay_RestrictedTeamZone` used for seeding already has this navmodifier, so seeding layers restrict bots from entering the enemy team’s restricted zone.

![RestrictionZones](https://github.com/user-attachments/assets/357bb426-9097-4a0e-9fa5-8ede7400e446)

# Adding bots to Seeding Layers

Once the navmesh is sorted out, getting bots in seeding is very simple. Assuming you have a functional seed layer already, place an `SQSeedingBotSpawner` in the world.

![SQSeedingBotSpawner](https://github.com/user-attachments/assets/2a9cdfd5-71fa-4f07-956f-eb2441a4adf6)

You’ll want to place one for each team, near their team’s player spawn point. Ensure that they are within the navmesh, otherwise the bots will be immediately killed on spawn due to stuck detection. For seeding, you’ll only need to modify the `TeamNum` setting. Seeding bot spawners rely on the gamemode being set up correctly. If set up incorrectly, there’ll be logs explaining how to amend the layer.

![SeedingBotSpawnerDetails](https://github.com/user-attachments/assets/42c77a5b-2b83-40ec-9fd9-ed3311a29a16)

## Seeding Bot Counts

Seed bot counts are determined by the `SeedTargetPlayerCount` (in the `BP_GameStateSquad_Seed`). This is read from the `ServerConfig/CustomOptions.cfg`. Here’s what is explained there:

```
// Target player count for seeding. Bots will be used to fill the server up to this player count.  
// Teams will be balanced so that both teams have an equal number of players+bots.
// With SeedTargetPlayerCount=40, this is what you'll see:
// On an empty-ish server            // On a more full server
// Team 1    3 players    17 bots        // Team 1    16 players    4 bots
// Team 2    5 players    15 bots        // Team 2    18 players    2 bots
// Bots will all be killed (and none will spawn) when the seed match is live.
#SeedTargetPlayerCount=40
```

Note that the `SpawnerMaxBotCount` is unused for `SeedingBotSpawners`, it just uses the `SeedTargetPlayerCount`. This can be set at runtime using `AdminSetSeedTargetPlayerCount X`.

Multiple seeding bot spawners can be placed per team, but the team will still be limited to the (target player count / 2\) number of bots.

# Squad currently only supports Static Navmeshes

This is significant as Squad’s worlds are quite dynamic, with deployables, vehicles and wrecks affecting the game world and blocking once-available paths. The static navmeshes means that bots don’t know how to path around these objects, and will try to walk into them and get stuck. As a short term workaround (and legacy from the mod), bots stuck walking into deployables will automatically passively dig them down, until the deployable is destroyed or the bot is able to get unstuck. This is important to note in the seeding gamemode, as placing the seed bot spawners near fobs / habs can have gameplay consequences, due to the bots potentially getting stuck and digging these down.

# SquidBot Fundamentals
## Configurations
SquidBots officially supports:
* Singleplayer (standalone)
* Multiplayer (replicated clients)
* Editor scripting (level sequences)

## Game-agnosticism
On paper, SquidBot should be game agnostic and not depend on Squad functionality. This was an advantage of SquidBots as a mod not depending on a Squad base class, and we'd like to keep it that way. In practice, lots of Squad code and systems are used by SquidBots in Blueprint. No new code is being written depending on Squad (the plugin's C++ modules don't depend on Squad for example), so slowly as systems are refactored, the system should move towards being entirely game agnostic. Squad can depend on SquidBots, but SquidBot cannot depend on Squad.

### `UODKSquidBotsProjectInterface`

Some functionality SquidBots is looking for must be implemented for use in a project. Squad's implementation of this is `USQSquidBotsProjectImplementation`. Setting the `SquidBotsProjectImplementationClass` in the `ODKSquidBotsSettings` will assign the project its implementation class. SquidBots will manage the lifetime of a singleton of this class, then talk to it through the `UODKSquidBotsProjectInterface`.

## SquidBot Class Hierachy
* ACharacter: UE4 Engine base class, note that SquidBots does not use any SQ parent class.
    * ODKAvatar: C++ Parent class, very minimal implementation details. Adds key functionality like Health, Seeded Random support. Slowly, more stable functionality and features needing to be optimised will be moved into this.
        * SkeletalMeshPawn: A redundant legacy parent class that should be removed.
            * KillableSkeletalMeshPawn: Adds custom logic for damaging. Historically was added for non-humanoid bots (Emus lol) but should be removed and damage logic refactored away.
                * Avatar: The base class for any animation work. This was originally where 99% of the mod was developed, and contains most logic for the solider and their weapons.
                    * SquadSoldier: Adds support for SquadRole parsing (not very project agnostic), and adds support for entering vehicles. Any cinematic work should be using SquadSoldiers as they contain no AI logic.
                        * SquidBot: Adds all AI logic for soldier bots. SquadSoldiers can only be keyframed, but SquidBots can do complex behaviours and processing including scanning the world around them and automatically hunting down targets, following orders etc.
                            * SQSquidBot: Squad implementation of SquidBot. Any Squad-specific details should be contained to SQSquidBot. For example, SQSquidBot adds any system interactions for the Squad map and nametag systems, and interactions with water. The optimal setup would be everything above this point would be project agnostic and squad then adds its defaults and overrides here.

## Determinism and Seeds
When SquidBots are used for creating cinematics, it's pivotal for creators to get the exact same results whenever they play their sequence. With that in mind, a requirement of SquidBots' systems is all actions should be deterministic and repeatable. SquidBots does use randoms, but a seeded random system. The SquidBot has a `Seed` variable defined. Whenever a random is needed for initialisation of a bot, the seed is used for random number generation. Whenever a continuous series of random numbers is needed, `UODKSequencerHelpers::GetRandomSeedAtFrame` should be used. This will hook up to any existing level sequences to ensure that the seed is the same at that frame every time the sequence is played.

> Tip: SquidBots have a function on them to force randomise the seed. Selecting multiple bots and running this event will give them all different random seeds.
>
> ![Force Randomise Seed](https://github.com/user-attachments/assets/07a76a68-45ed-4071-ad85-e42045f58eb6)

## Items
Weapons and Items (all inheriting from `GenericItem`) are designed to exist both owned by a bot and by themselves. `SQWeapon` does not support this functionality, and this can be used to make simple weapon sentries or test weapon firing functionality without a bot

## Event Construct
SquidBots and Items both use a function called `Event Construct` to do most of their initialisation logic. This gets run at either construction time or beginplay, depending on the when the bot is spawned.

## Teams
SquidBots are not constrained to just Team 1 or Team 2. SquidBots have a team number that can be set to any integer. Bots will view any other entities that are not on their team as their enemy.

# Spawning Bots
Bots can be preplaced in the editor. Drag-and-drop a `SQSquidBBot` into the world.

![Place SQSquidBots](https://github.com/user-attachments/assets/3a02e62e-8ada-47b6-bc51-7a5bf54045fe)

There are a **lot** of settings configurable on the SquidBot details view, but all you need to search for to get a spawned bot in the scene is the `SquadRole`

![SquadRole](https://github.com/user-attachments/assets/ba3acbc4-fe9b-4702-a1c8-28f66e1cb622)

Set the `BP_SQFactionSetup` and `BP_SQRoleSettings` to the faction and role you want the bot to use. Optionally, you can use a random role (enable `Use Random Role`, which will choose a random role from the faction setups you provide in the `RandomRoleSettings.FactionSetups` array. The random role system also lets you filter for roles with specific tags or using specific weapons. Tags work that for a role to pass the filter, all of its tags must be in the `RandomRoleSettings.Tags` array. You may want to also set the `Seed` and the `TeamNum`.

## Using Spawners
TODO: Improve spawner systems and documentation.

Currently the only spawner that works out of the box for SquidBots is the [`SQSeedingBotSpawner`](https://github.com/OffworldIndustries/SquidBots/edit/main/README.md#adding-bots-to-seeding-layers).

## Manually Spawning Bots at Runtime
As a designer, the 2 most important API calls you have are spawning bots and giving them orders. Once you get to grips with those systems, the world is your oyster. To Spawn a bot, use the Blueprint SpawnActor node, and select a SQSquidBot class. Parameters will be explained elsewhere in this document, so I would recommend at this point usign `ctrl+f`.
![SpawnActor SQSquidBot](https://github.com/user-attachments/assets/039c15e5-3d73-40d5-9c1d-a8a228fcef4f)

# Order System
## Giving Orders

## Creating Orders

# Bot Behaviour
## Senses
### Sight

### Sound

### Communication

## Patrol (Follow Order)

## Search

## Combat

## Reload

## Flee to Cover

# Bots for Cinematics
