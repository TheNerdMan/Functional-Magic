# Goals
I'd like the 'built in' functions of this language to be agnostic. Meaning that when you "define" the spell its the *inputs* and the functions used that dictate how the spell will manifest.

I'd like the "complier" when running a function to output something like;
``` bash
$ functional-magic my-spell.mana
> The pages of your book turn mysticly # Flavour text to confirm running
> You say "Glacius Tempus!" in Draconic # Somantic
> Your Staff Channels magic # Focusing
> Drawing upon Fire # Magic type 
> You offer Sulfur and BatGuano # Material
> Your mind inputs the distance # Range/Shape/Size
> You set the recursion # Level
> It misses! # Roll is in the targets favour OR
    > It hits! # Roll is in your favour
> They take 10 fire damage # Pre rolled dice OR
    > They take 2d10 + 10 fire damage # Manual rolls for physical dice
> They are burned for 1 round # Lingering effects
```
or rather than a running one, describes what this spell you've created will do;
``` bash
$ functional-magic my-spell.mana
> When you cast this spell, you will need XYZ
> then say "magic word"
> then do "fancy hands"
> and a ice shard will impale your target.
> It will do 1d10 ice damage 
> and cost you 1 level 1 spell slot(s)
```

The "complier" could just be a simple;
``` bash
$ functional-magic my-spell.mana
> It seems like this spell will work # You wrote something fine OR
> The unit tests of the universe deem this borked # It doesn't compile
```

If I go with the above I want to purposly consume the compiler error. Keeping the mysteries of magic locked way.

# Defining the framework
## 1. Universal Spellcasting Functions
```
# Accepts FocusType as a parameter (staff, orb, amulet).
FOCUS(What: FocusType) -> "Your "+ What +" Channels magic" 
```

```
# Supports any spoken incantation and language.
INCANT(Phrase: String, Language: String) -> "You say \"" + Phrase + "\"" in " + Language
```

```
# Allows any energy type to be channeled (Fire, Necrotic, Arcane).
CHANNEL(Energy: MagicType) -> "Drawing upon " + Energy
```

```
# Accepts a list of material components, so crafting spells with unique materials is possible.
OFFER(Components: List<Component>) -> "Consumes " + Components
```

```
# Generic casting function, allowing custom spells to be defined dynamically.
CAST(SpellName: String, Level: Int, Effects: List<Effect>) -> "Unleashes " + SpellName + " at Level " + Level
```
## 2. Flexible Spell Effects
```
# Can work for ranged, melee, or magical attacks.
STRIKES(Target: Entity, Method: AttackType) -> Bool
```

```
# Works for any effect (Paralyze, Invisibility, Slow, etc.).
APPLY_EFFECT(Target: Entity, Effect: EffectType, Duration: Time)
```

```
# Allows custom damage types (Fire, Psychic, Poison, etc.).
## I'm not a fan of "DAMAGE"/"HEAL" as undead effects swap these
DAMAGE(Target: Entity, Amount: Int, Type: DamageType)
HEAL(Target: Entity, Amount: Int)
```

```
# Generic summoning function that can be used for elementals, spirits, or constructs.
SUMMON(Entity: Creature, Duration: Time, Location: Position)
DISMISS(Entity: Creature)
```

## 3. Core Mechanics for Interaction
```
# Allows any spell to use custom saving throws.
SAVE_THROW(Target: Entity, Type: SaveType, DC: Int) -> Bool
```
```
# Any effect can be maintained or broken dynamically.
CONCENTRATE(Duration: Time, Effect: EffectType) -> Bool
```
```
# Works on any effect, not just specific spells.
DISPEL(Target: EffectType, DC: Int) -> Bool
```       

## An example spell
```
DEFINE_SPELL("Frostbite", {
    RANGE: 60;
    DAMAGE: (Level) -> Level * 2d6;
    TYPE: Cold;
    DURATION: Instant;
    COMPONENTS: [ShavedIce];
    
    CAST(Level) -> {
        INCANT("Glacius Tempus!", Draconic);
        CHANNEL(Cold);
        OFFER([ShavedIce]);

        IF STRIKES(Target, Magic) {
            APPLY_EFFECT(Target, "Slowed", 1 Round);
            DAMAGE(Target, DAMAGE(Level), Cold);
        }
    }
});
```