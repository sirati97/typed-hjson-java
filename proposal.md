Entry Syntax
==
Type:Name: Value Literal

Native types
==
| Literal | Meaning | Example | Comment |
| ------- | ------- | ------- | ------- |
| `I`       |  Integer
| `F`       |  Fraction/Floating point
| `S`       |  String
| `R`       |  Range
| `D`       |  Date&Time | | same as TOML, can specify format
| `T`       |  Type Decl
| `L`       |  Literal Decl
| `#`       |  Enum Type Decl postfix | |Enum types can contain data!
| `?`       |  Nullability postfix |`I?:nullableInt: ?` | ? is null literal
| `[]`      |  array postfix, optional | `I[]:numbers: 1` = `I[]:numbers: [1]` = `I:numbers: [1]` | if given interpret one value singleton array


Value Literals
==
Range
--
| Literal | Example | Comment |
| ------- | ------- | ------- |
| Number..Number | `1..2`  |(1 & 2 inclusive)
| [Number..Number] | `[1..2]`| (1 & 2 inclusive)
| Number..Number[ | `[1..2[`| (1 inclusive, 2 exclusive)
| Number..Number[ | `1..2[`| (1 inclusive, 2 exclusive)
| [Number..Number[ | `[1..2[`| (1 inclusive, 2 exclusive)
| ]Number..Number[ | `[1..2[`| (1 & 2 exclusive)

Fraction / Decimals
--
| Literal | Example | Comment |
| ------- | ------- | ------- |
| `Int/Int` | `1/3` |
| `Int Int/Int` |  `1 1/3` | equal to 1 + 1/3
| Fix point decimal number | `1.3` | |
| scientific notation | `1.23e-5` | |
| decimal starting with point | `.43` | equal to 0.43 |

Generated Comment Literal
--
`###` starts a comment that will be copied into the configuration file

Enum Literal
--
[Name: Value Literal, Name: Value Literal]

Ref Value Literal
--
$Top level key|enum type [.field.field]

Type Decl Literal
--
```yaml
T:Name:  [supertype, supertype] {
 L:LiternalName: Literal Decl Literal #see below
 Type:FieldName! #manditory field
 Type:FieldName!: [array of allowed values]! #manditory field
 Type:FieldName!: [array of default values]? #manditory field, only useful when writing presets for config files
 Type:FieldName!: [array of allowed values]![array of default values]?
 S:FieldName!: '''REGEX''' #manditory string field with regex verifier
 Type:FieldName: Default Value #optional field
}
```

`T?:Name:  [supertype, supertype]` #abstract type

`T#:Name:  [type option 1, type option 2, [supertype, supertype] option 3, ...]` #Of any type of allowed


Single Value Class Literal
--

If there is only one non-optional variable, we can assign that type without having to be verbose

```yaml
T:Block: {
  S:id!
}

#This is the same
Block:Stone: minecraft:stone

#as this
Block:Stone: {
  S:id: "minecraft:stone"
}
```

Literal Decl Literal
--

```yaml
T:Ressource: {
 L:Full: $Category\$Namespace:$Path
 L:NoCategory: $Namespace:$Path {
    S:Category: ""
 }
 L:MinecraftRes: $Category\$Path {
    S:Category: ""
 }
 L:MinecraftBlock: $Path [NoCategory] {
    S:Namespace: Minecraft
 }
 S:Namespace!: '''[a-z][1-90a-z_]*''' 
 S:Path!: '''[a-z][1-90a-z_]*''' 
 S:Category!: '''[a-z][1-90a-z_]*''' 
}

Ressource:Stone Block: stone
Ressource:Stone Block Texture: block\stone
Ressource:Lead Ore: extra_ores:lead_ore
Ressource:Lead Ore Texture: block\extra_ores:lead_ore
```

Example
==

Preset
--
Type definitions, and defaults for config file

```yaml
##start type definitions

T:Block: [] {
  Ressource:id!
}

T:PropBase: {
  S:name!: '''[a-z][1-90a-z_]*''' 
}

T:Bool Prop: [PropBase] {
  B:value!
}

T:Int Prop: [PropBase] {
  I:value!
}

T:Str Prop: [PropBase] {
  S:value!: '''[a-z][1-90a-z_]*''' 
}

T#:Prop: [Bool Prop, Int Prop, Str Prop] {
  L:ctor: $name: $value
}

T:BlockState: {
  Block:Block!
  Prop[]:Properties: []
}


Block:Stone: stone //normal minecraft stone, here stone is a string that uses the L:MinecraftBlock of Ressource


//the below is for generating a comment in the config file part
###Define a list of the stone types to be used for ores.
BlockState#:StoneTypes! [stone: $Stone]? //this config must define the enum of type BlockState with name StoneTypes


T:Ore {
  BlockState:BaseOre!
  StoneTypes#[]:Embeddable In!
}

###List ores to be generated in the overworld here.
Ore[]:ActiveOverworldOres!

```


Config file
--
```yaml

##start user config
##everything above here should be put into a seperate file, or leaded from memory

Block:ModStone: quark:realistic_stone

//generated comment below
#Define a list of the stone types to be used for ores. DEFAULT: [stone: $Stone], ALLOWED: ANY
BlockState#:StoneTypes: [
    stone: $Stone  #Single Value Class Literal
    marble: "terrafirmcraft:marble"  #Single Value Class Literal, then defined literal
    granite: {
        Block: $Stone  #omitted type
        Properties: [variant:granite] #using literal of Prop, resolves to Str Prop
    }
    dark marble: {
        Block: $marble.Block #cross reference
        Properties: [is_dark:true] #using literal of Prop, resolves to Bool Prop
    }
    green slate: { #completely spelled out
        Block:Block: $ModStone,  #reference to global
        Prop[]:Properties: [
            S:Name: "variant" #omitting {}, comma here illegal as incomplete content definition
            S:Value: "slate", #thus comma needed
            is_molten:false,  #using literal, technically not needing comma, but good practise
            { #not omitting {}
                S:Name: "colour", #comma optional
                S:Value: "green", #comma allowed doesnt matter
            }, #comma allowed doesnt matter
        ]
    }
]

Ore:Iron { #the colon before {} brackets can be ommitted
    BaseOre: iron #string -> Block -> Blockstate
    Embeddable In: [stone, granite, green slate] #here we do not need $ as we are defining which enum members to use
}


Ore:Quartz {
    BaseOre: {
        Block: extra_ores:crystal_ore
        Properties: type:mountain_quartz #the [] can be ommitted
    }
    Embeddable In: [granite, marble, dark marble] 
}

#List ores to be generated in the overworld here. DEFAULT: NONE, ALLOWED: ANY
Ore[]:ActiveOverworldOres: [$Iron, $Quartz] #here we need the $ as we are not dealing with an enum
```


It would be allowed that the user defined their own enum, to avoid having to write $
```yaml
Ore#:Ores [ #the colon before [] brackets can be ommitted
    Iron: {
        BaseOre: iron #string -> Block -> Blockstate
        Embeddable In: [stone, granite, green slate] #here we do not need $ as we are defining which enum members to use
    }
    Quartz {
        BaseOre: {
            Block: extra_ores:crystal_ore
            Properties: type:mountain_quartz #the [] can be ommitted
        }
        Embeddable In: [granite, marble, dark marble] 
    }
    Unused {
        BaseOre: iron #string -> Block -> Blockstate
        Embeddable In: marble
    }
]

#List ores to be generated in the overworld here. DEFAULT: []
Ores[]:ActiveOverworldOres: [Iron, Quartz] #here we use the enum members!
//mind that Ore[] had to be changed to Ores[] for this to work
//otherwise we would have had to use [$Ores.Iron, $Ores.Quartz]
//but the below might be even better
Ore[]:ActiveOverworldOres: $Ores #here we use all members, implicit conversion to array

```
