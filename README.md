# Cold Waters Enhanced

Based on work by Skwabie.

## Features

- All the stuff Skwabie already fixed
- Torpedoes can have a maximum depth. Default is set to 1000 feet.
- Vessels may have a noise-per-knot to reduce or increase the rate at which they accrue noise as they move faster.
- Sonar noise per knot affects target signature display.
- Towed sonar arrays may have custom speed parameters for optimum tow speed and never exceed speed.
- Baffles respect optimum tow/never exceed speed.
- Torpedoes may report contacts (in vanilla, torpedoes only provide TMA data for existing contacts)
- "Advanced" torpedoes may report TMA data while in passive homing mode.
- Passive and semi-active homing missiles may require that the launch platform remain on the surface to provide guidance.

The torpedo depth thing is a (very) little more lax here in CWE than in Epic Mod; I guess I wanted to cut the manufacturers some slack. I mean, just because the warranty is void doesn't mean it stops working *immediately,* right? :)

This mod determines "advanced" torpedoes the same way the original Skwabie mod did: anything with a sensor cone greater than or equal to 55 degrees.

### Wide aperture arrays

Skwabie's wide aperture array feature adds three optional parameters to sonar. These do NOT exist on any of the default sensors, so you'll need to edit them in yourself if you want to use them (or else use overrides created by someone else).

Sample:
```
SonarWAARange=15000
SonarWAABaseSol=5
SonarWAATMARate=10
```

The wide aperture array increases the rate at which TMA data is accrued on any target within range. Range is determined by `SonarWAARange` and is expressed in yards.

The base increase in TMA accrual due to the wide aperture array is determined by `SonarWAABaseSol` and applies to any target in range. The `SonarWAATMARate` boost is applied only on targets as their bearing changes. (I hope I'm using "bearing" correctly.)

```
// USN AN/BQQ-10 sonar with wide aperture array config:
SonarModel=usn_an_bqq_10
SonarType=ACTIVE/PASSIVE
SonarFrequencies=L,M
SonarActiveSensitivity=33
SonarPassiveSensitivity=45
SonarBaffle=130
SonarNoisePerKnot=1
SonarOutput=280
SonarWAATMARate=10
SonarWAARange=15000
SonarWAABaseSol=5
```

### Torpedo maximum depth

This feature adds a single config item to all weapons (but it is only checked for torpedoes and decoys, so...) which can be used to trigger the destruction of the weapon if it descends below a certain depth. The main inspiration here is the reality that some submarines can dive deeper than some weapons can function. I heard somewhere that the original Mk48 torpedo (not the ADCAP found in-game) wasn't able to dive deep enough to engage boats of the Alfa-class, but I can't find a source. Whatever.

MaxDepth for all weapons is set, by default, to 1000 feet. To change the max depth (most modern torpedoes have a maximum depth of at least 500 meters, or about 1600 feet), add a line like this to the torpedo's configuration in weapons.txt: `MaxDepth=1640`. This should appear before the "Model" section for the weapon, as in the following example (which is for the UK's Spearfish, taken from PBS).

```
WeaponObjectReference=uk_sf
WeaponSprite=hud/default/uk_sf_sprite.png
WeaponType=TORPEDO
Warhead=560
SurfaceLaunched=FALSE
WireGuided=TRUE
WireBreakOnLaunchProbability=0.05
WireBreakSpeedThreshold=20
RangeInYards=54000
RunSpeed=60
ActiveRunSpeed=80
TurnRate=24
SensorAngles=80,25
SensorRange=4000
WeaponNoiseValues=160,230
MaxPitchAngle=30
HomeSettings=PASSIVE,ACTIVE
AttackSettings=STRAIGHT,LEFT,RIGHT
DepthSettings=LEVEL,SHALLOW,DEEP
MinCameraDistance=0.5
ResupplyTime=15
MaxDepth=1640
[Model]
...
```

Obviously, world navies do NOT publish these numbers. The United States, for example, will only admit that the latest Mk48 has a maximum depth "greater than 1200 feet." How very helpful. Anyway, feel free to fudge the numbers to your liking. I think this adds a fun gameplay element where you play chicken with a torpedo. "My boat may survive going down to 1800 feet, but that UGST might not!"

Good luck pulling that off in the China campaign. >.>

### Self noise per knot

I am not a submariner, but I feel it is possible that, given two different hull shapes, one of these shapes may become louder at speed than the other. Perhaps it has holes in funny places, or maybe the hull starts rattling as the water flows over it. I don't know. It's just a theory. Anyway, to support this concept, I have added a noise-per-knot modifier.

This value already existed in-game, but it was a global modifier found in the config.txt and not used for anything (the value was always set to 1 regardless of the setting in config.txt). I'm still ignoring that value (called `TargetNoisePerKnot`), but I've introduced a vessel-specific noise per knot setting called `SelfNoisePerKnot`.

As the default for this modifier was `1.0`, making this number larger than `1.0` will make your sub louder as it moves faster through the water. Making it smaller will make it quieter. This primarily affects the detection chances of enemy ships, but it will also affect your ability to detect them, because self noise degrades your sonar data.

```
// Acoustics config of the theoretical "Improved Virginia" from PBS
// with SelfNoisePerKnot modifier added. Naturally, this makes the
// improved Virginia even more OP :p
[Acoustics & Sensors]
SelfNoise=104
SelfNoisePerKnot=0.95
ActiveSonarReflection=25
ActiveSonarModel=usn_an_bqq_10
PassiveSonarModel=usn_an_bqq_10
TowedArrayModel=usn_tb_29
AnechoicCoating=TRUE
RADAR=usn_bps_15
RADARSignature=SMALL
TowedArrayPosition=-0.075,0.0,-0.619
```

### Sonar noise per knot

Imagine a modern sonar with some kind of learning AI to help filter out pointless noise on your sonar scope. That's the intention behind the change to `SonarNoisePerKnot`, which is a pre-existing config item for sonar sensors that was not, as far as I can tell, ever used in the base game. I've added code to make use of this value when drawing your target's signature on the sonar display.

This works similarly to `SelfNoisePerKnot` except that it really only effects the display itself, potentially making it easier for players to manually identify targets when traveling at speeds greater than five knots.

Existing sonar sensors already have this value in their configuration sections; if you want to see less "noise" on the scope as your submarine travels faster, you can change that value to be lower than 1. If you want your life to be harder, change it to be *higher* instead.

Additionally, note that `SonarNoisePerKnot` has no effect when applied to an Active or Towed sonar; the noise on your sonar scope is derived *only* from the `SonarNoisePerKnot` on your vessel's passive sonar.

### Configurable towed array speeds

Two configuration items have been added to towed arrays:

- OptimumTowSpeed
- NeverExceedSpeed

The poorly-named `OptimumTowSpeed` is the *maximum* speed at which the towed array can achieve optimal operation and past which its performance begins to degrade. The default optimal speed is five knots. If the optimal speed is set higher than five knots, optimal performance will be maintained anywhere from five knots up to the optimal speed.

The humorously-named `NeverExceedSpeed` is the speed at which the towed array becomes inactive, theoretically because the game "reels in" the towed array for you beyond a certain speed. By default, this is set to fifteen knots, which mimics the operation of the original, hard-coded values in-game.

The intent behind these configuration values was to allow for the idea of a near-future sci-fi "high speed towed array," which functions (albeit not well) at speeds greater than ten knots, which was where the game's existing towed arrays basically stopped working. The current implementation causes array performance to degrade linearly as vessel speed changes from the optimum to the never exceed speed.

```
// Fictional high-speed towed array
SonarModel=usn_tb_45
SonarType=TOWED
SonarFrequencies=VL,L
SonarActiveSensitivity=0
SonarPassiveSensitivity=94
SonarBaffle=FALSE
SonarNoisePerKnot=0.4
SonarOutput=0
OptimumTowSpeed=10
NeverExceedSpeed=25
```

The fictional array above becomes less effective than the standard passive sensors at about 15 knots.

Lastly, a target is no longer considered to have entered your "baffles" based on the hard-coded speed of 10 knots (which, in the base game, deactivates your towed array for the purpose of baffles checking). For the purpose of detecting a target in the baffles of your submarine, the average of the optimal and never exceed speed is used. For the fictional towed array above, that would be about 17 knots, meaning that your ship has no "baffles" at 15 knots but is blind to the rear at 20.

### Torpedo changes

In the base game, active torpedoes may appear to have a "flashlight" effect when they ping a target. For a long time, I thought that my sonar was simply picking up the return from the torpedo's pinger, and that's probably what the developers intended this mechanic to simulate, but that's not actually how it works in the game's code.

In fact, when an active torpedo *on a wire* pings a target, that target's TMA data is set to maximum for the player, as though the torpedo were reporting the target's position via the wire. (There is no effect for a torpedo not on a wire.) The thing is, modern, advanced torpedoes can do that with or without active sensors enabled. To represent this, "smart" torpedoes now provide TMA data even without the active seeker enabled.

Furthermore, I noticed an issue wherein torpedoes were able to provide TMA data for targets that don't appear on the sensors for your ship. This probably was a gameplay decision on the part of the developers rather than a bug, but I have changed the behavior so that torpedoes are able to trigger new contacts.

Lastly, Skwabie's implementation of swim-through for advanced torpedoes launched by the player had them perform the swim-through operation *followed by* the countermeasure homing operation. This resulted in a serious degradation of torpedo performance for the player. This issue has been fixed.

### Passive/semi-active homing missiles

Missiles with the flag `RequireGuidance=TRUE` will be destroyed if the launch platform submerges or lowers its radar mast. This is intended to support older cruise missiles that lacked an ability to locate a target without assistance.

## Potential problems

### Anti-submarine missiles

I'm not totally convinced that anti-submarine missiles work correctly. I remember I stopped using them because it seemed to me that enemies did not react to these. Shooting fish in a barrel isn't that fun. That said, it's possible I just wasn't using Skwabie's mod back then. I don't remember.

### Torpedo tracking

Torpedoes are freaking annoying in this version. Combine "drive-thru" on any modern torpedo with the fact that they now climb or descend directly toward targets rather than potentially passing above or below and you have a situation where evasion using knuckles or countermeasures is... "Challenging." I'm not sure this makes the game more fun.

### Russians less trigger happy

There was always something satisfying about launching a salvo of Tomahawks at an invasion fleet and initiating a crash dive just before the water filled with a dozen missile-delivered Soviet torpedoes. This no longer happens, because now the Reds want to know where you are before they shoot. Presumably, the Amerikanskis have the same problem. Admittedly, this shoot first, ask questions later attitude represented (mostly) a big waste of tax dollars, but I still think the game was more fun before the bad guys knew the definition of "overkill."

I *think* this relates to the following note from Skwabie's change log: **AI will only counter fire if launch detection.** Let me know what you think.

## Skwabie's original features

- WirebreakSpeedThreshold in weapons.txt now works. Wire break chance only affected by speed.
- Angled tube jams at ~10% chance (speed / 300) above 30.5 knots.
- Snake search pattern changed to much narrower. Default is 10deg/sec for 9 seconds turns. Changed to 2 deg/sec turn for 20 seconds.
- Torpedo vertical homing algorithm changed to pure pursuit.
- Torpedo run deep depth set to 200ft above sea floor, or 1000ft, whichever is shallower.
- "Advanced" torpedoes (sensor angle >=55 deg) reject wrecks, whales and oilrigs, and have a routine to evade if the target it locks on to is destroyed by another torpedo: stop homing - dive to run deep depth – snake search. Exception: countermeasure homing when target destroyed or evaded, upon which it will only circle search.
- AI launched torpedoes with sensor range above 2700yards will use drive through on jam. Below 2700 – drive around jam. By default AI torp drivethrough/drive around is based on chance. Player launched torpedoes: drive through jam on passive homing, drive around jam on active homing.
- AI launched torpedoes: sensor angles < 45 deg – snake; sensor angles between 45 and 55 deg – chance for either snake or straight; sensor angles > 55 deg – straight search.
- Player ASW missile torpedo will dive to depth of closest enemy at drop point and trigger AI reaction, and can also target surface ships.
- Sonar ping lines fade much slower so can get a solid bearing out of it.
- Max number of helos increased to 8 and min number ~ half of TF total. More chance for 8 if total helo count above 8. Initial patrol radius increased.
- Fixed wing aircraft is much less sticky.
- Enemy torpedoes can hit surface ships again.
- Mission start spawn range increased, by decreasing sonar contact strength requirement.
- AI detection range against player missile is largest at missile launch. Missiles have size factor 0.4 within launch +7 sec (booster is bright and lotta smoke), 0.2 outside launch +7 sec, 0.25 beyond waypoint. – Don Kay search radar range 50k yards, so launch detection 20kyds, missile detection 10k, active missile 15k. AI will only counter fire if launch detection. Largely decreased radar detection points on player by detecting player missiles.
- HeloMADMultiplier available in config.txt to make helicopter MAD detection a boost.
- Wide Aperture Array sonar properties available for sonar, configurable in sensors.txt.

## Version history

- 0.1.0-alpha.1: Fix swim-through torpedoes, support semi-active homing cruise missiles.
- 0.1.0-alpha: Initial release.

## Credits

- Skwabie (for most things)
- The Epic Mod guys (for torpedo maximum depth)
