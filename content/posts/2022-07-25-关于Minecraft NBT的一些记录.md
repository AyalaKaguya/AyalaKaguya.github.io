+++
title = "关于Minecraft NBT的一些记录"
date = "2022-07-25T09:30:17+08:00"
author = "AyalaKaguya"
tags = [
    "Minecraft",
    "NBT",
    "游戏",
    "笔记"
]
categories = [
    "游戏",
]
keywords = [
    "Minecraft",
    "NBT",
    "游戏",
    "笔记"
]
+++

偶然看到电脑里存的一份记录着Minecraft NBT用法的文本文档，干脆拿过来当文章了（恼

## 都有些啥？

在一个大的json里套了一堆东西，如果写TypeScript表示那更好，但可惜不想写（懒

大部分都是从[minecraft fandom](https://minecraft.fandom.com/zh/wiki/%E6%95%99%E7%A8%8B/NBT%E5%91%BD%E4%BB%A4%E6%A0%87%E7%AD%BE)找的

### 附魔 Enchantments

```json
{
    Enchantments:[
        {id:"魔咒ID",lvl:#魔咒等级},
        {id:"魔咒ID",lvl:#魔咒等级}
    ]
}
```

类型: List

字段定义:

| 字段名 | 类型          | 意义       |
| ------ | ------------- | ---------- |
| id     | String        | 魔咒的id   |
| lvl    | uint8_t（雾） | 魔咒的等级 |

### 名称显示 display

```json
{
    display:{
        Name:"物品名称的JSON文本",
        color:"颜色代码",
        Lore:["第一行","第二行"]
    }
}
```

类型: Object

字段定义:

| 字段名 | 类型            | 意义               |
| ------ | --------------- | ------------------ |
| Name   | String          | 物品名称的JSON文本 |
| Color  | String with int | 颜色代码           |
| Lore   | List of String  | 物品描述           |

### 属性 AttributeModifiers

```json
{
    AttributeModifiers:[
        {AttributeName:"#",Name:"#",Amount:#,Operation:#,UUID:[I;#,#,#,#],Slot:#},
        {AttributeName:"#",Name:"#",Amount:#,Operation:#,UUID:[I;#,#,#,#],Slot:#}
    ]
}
```

类型: List

字段定义:

| 字段名        | 类型          | 意义                                                                                                                                                                                                                          |
| ------------- | ------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| AttributeName | String        | 决定属性修饰符所应用的属性。                                                                                                                                                                                                  |
| Name          | String        | 属性修饰符的名称，用于内部的区分和识别。                                                                                                                                                                                      |
| Amount        | int           | 决定属性修饰符的数值。                                                                                                                                                                                                        |
| Operation     | int           | 决定属性修饰符的运算模式。0为增加（X+Amount），1为按基加倍（X*(1+Amount)），2为加倍（X*(1+Amount)）。1和2的区别是，当存在多个Amount时，1会按照X*(1+Amount1+Amount2)的形式运算，而2会按照X*(1+Amount1)*(1+Amount2)的形式运算。 |
| UUID          | list object   | 属性修饰符的UUID，用于区分不同的属性修饰符。对于所有增加属性的装备而言，相同UUID取最高                                                                                                                                        |
| Slot          | int or String | 决定物品处在什么槽位时，属性修饰符才会生效。如果没有定义此项，属性修饰符将对所有槽位生效。                                                                                                                                    |

### 不可破坏 Unbreakable

```json
{Unbreakable:1b}
```

类型: Bool

0或者1，b写不写貌似无所谓

### 附魔书 StoredEnchantments

```json
{
    StoredEnchantments:[
        {id:"魔咒ID",lvl:#魔咒等级},
        {id:"魔咒ID",lvl:#魔咒等级}
    ]
}
```

类型: List

字段定义:

| 字段名 | 类型          | 意义       |
| ------ | ------------- | ---------- |
| id     | String        | 魔咒的id   |
| lvl    | uint8_t（雾） | 魔咒的等级 |

## 再详细说说

### 关于魔咒

| 魔咒名称   | 魔咒id                |
| ---------- | --------------------- |
| 水下速掘   | aqua_affinity         |
| 节肢杀手   | bane_of_arthropods    |
| 爆炸保护   | blast_protection      |
| 引雷       | channeling            |
| 绑定诅咒   | binding_curse         |
| 消失诅咒   | vanishing_curse       |
| 深海探索者 | depth_strider         |
| 效率       | efficiency            |
| 摔落保护   | feather_falling       |
| 火焰附加   | fire_aspect           |
| 火焰保护   | fire_protection       |
| 火矢       | flame                 |
| 时运       | fortune               |
| 冰霜行者   | frost_walker          |
| 穿刺       | impaling              |
| 无限       | infinity              |
| 击退       | knockback             |
| 抢夺       | looting               |
| 忠诚       | loyalty               |
| 海之眷顾   | luck_of_the_sea       |
| 饵钓       | lure                  |
| 经验修补   | mending               |
| 多重射击   | multishot             |
| 穿透       | piercing              |
| 力量       | power                 |
| 弹射物保护 | projectile_protection |
| 保护       | protection            |
| 冲击       | punch                 |
| 快速装填   | quick_charge          |
| 水下呼吸   | respiration           |
| 激流       | riptide               |
| 锋利       | sharpness             |
| 精准采集   | silk_touch            |
| 亡灵杀手   | smite                 |
| 灵魂疾行   | soul_speed            |
| 迅捷潜行   | swift_sneak           |
| 横扫之刃   | sweeping              |
| 荆棘       | thorns                |
| 耐久       | unbreaking            |

### 关于属性

| 属性名称                     | 效果                                                                                                                                 |
| ---------------------------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| generic.max_health           | 这个生物的最大生命值；亦或这个生物通过生命恢复最多可以恢复至的极限。你需要运用[Health:#]nbt改变生物的当前生命值                      |
| generic.follow_range         | 这个生物追踪玩家或者其他生物的最大范围，以方块数为单位。目标离开这个区域意味着它们将停止追踪。目前大多数生物这个值为16，而僵尸则有40 |
| generic.movement_speed       | 移动速度                                                                                                                             |
| generic.knockback_resistance | 这个生物的抗击退效果（包括攻击的击退、爆炸和弹射物冲击）的程度，1.0代表完全抵抗                                                      |
| generic.attack_damage        | 普通攻击造成的伤害，一点表示半个心形标志。此属性在友好生物中未找到                                                                   |
| generic.attack_knockback     | 这个生物的攻击击退力度，列表之外的生物中不具备该属性                                                                                 |
| generic.armor                | 盔甲的防御点数                                                                                                                       |
| generic.armor_toughness      | 盔甲韧性                                                                                                                             |
| generic.attack_speed         | 决定攻击力度的填充速度，值代表每秒可以进行全力攻击的次数                                                                             |
| generic.luck                 | 影响战利品表使用的quality和bonus_rolls（例如当打开箱子、运输矿车，钓鱼和杀怪）                                                       |

### 关于槽位

可以写成数字形式，也可以写成字符串的形式，不同的写法也有可能导致相同的结果。

| 槽位名   | 对应位置       |
| -------- | -------------- |
| mainhand | 拿在主手       |
| offhand  | 拿在副手       |
| head     | 装备在头部     |
| chest    | 装备在胸甲部位 |
| legs     | 装备在腿部     |
| feet     | 装备在脚上     |

## 举个栗子

以下指令均须在命令方块里执行

`/give @p minecraft:Command_Block`

### 迅捷步伐

具有高额击退，拿着的时候提供速度加成。

```json
/give @p minecraft:feather{display:{Name:'["迅捷步伐"]'},Enchantments:[{id:"knockback",lvl:16}],AttributeModifiers:[{AttributeName:"generic.movement_speed",Name:"generic.armor_toughness",Amount:2,Operation:1,UUID:[I;1,1,1,9],Slot:"mainhand"}]}
```

### 嗜杀癫狂

纯粹的高伤害，其实里面有附魔没生效。

```json
/give @p minecraft:stick{display:{Name:'["嗜杀癫狂"]'},Enchantments:[{id:"sharpness",lvl:255},{id:"bane_of_arthropods",lvl:255},{id:"smite",lvl:255}],AttributeModifiers:[{AttributeName:"generic.movement_speed",Name:"generic.armor_toughness",Amount:0.2,Operation:1,UUID:[I;1,1,1,9],Slot:"mainhand"}]}
```

### 巧取豪夺

中等伤害，较高等级的抢夺。

```json
/give @p minecraft:bone{display:{Name:'["巧取豪夺"]'},Enchantments:[{id:"sharpness",lvl:4},{id:"looting",lvl:32},{id:"silk_touch",lvl:1}],AttributeModifiers:[{AttributeName:"generic.movement_speed",Name:"generic.armor_toughness",Amount:0.1,Operation:1,UUID:[I;1,1,1,9],Slot:"mainhand"}]}
```

### 燃尽薪焱

低伤害，但是会附着火焰。~~差不多刚好暴击一头牛~~

```json
/give @p minecraft:blaze_rod{display:{Name:'["燃尽薪焱"]'},Enchantments:[{id:"sharpness",lvl:16},{id:"fire_aspect",lvl:255}],AttributeModifiers:[{AttributeName:"generic.movement_speed",Name:"generic.armor_toughness",Amount:0.1,Operation:1,UUID:[I;1,1,1,9],Slot:"mainhand"}]}
```

### 坚强立场

拿着的时候减速，但是提供高额护甲和盔甲韧性，难以被击退，惧怕魔法伤害和破防

```json
/give @p minecraft:nautilus_shell{display:{Name:'["坚强立场"]'},Enchantments:[{id:"protection",lvl:5}],AttributeModifiers:[{AttributeName:"generic.movement_speed",Name:"generic.movement_speed",Amount:-0.8,Operation:1,UUID:[I;1,1,114,9],Slot:"mainhand"},{AttributeName:"generic.knockback_resistance",Name:"generic.knockback_resistance",Amount:2,Operation:0,UUID:[I;1,1,114,8],Slot:"mainhand"},{AttributeName:"generic.armor_toughness",Name:"generic.armor_toughness",Amount:255,Operation:0,UUID:[I;1,1,114,7],Slot:"mainhand"},{AttributeName:"generic.armor",Name:"generic.armor",Amount:255,Operation:0,UUID:[I;1,1,114,6],Slot:"mainhand"}]}
```

### 两本附魔书

魔咒等级太高而且冲突会导致附魔书失去作用

所有的保护

```json
/give @p minecraft:enchanted_book{StoredEnchantments:[{id:"blast_protection",lvl:5},{id:"feather_falling",lvl:5},{id:"fire_protection",lvl:5},{id:"projectile_protection",lvl:5},{id:"protection",lvl:5},{id:"unbreaking",lvl:3},{id:"depth_strider",lvl:3},{id:"respiration",lvl:3},{id:"thorns",lvl:4},{id:"soul_speed",lvl:4}]}
```

所有的伤害

```json
/give @p minecraft:enchanted_book{StoredEnchantments:[{id:"looting",lvl:3},{id:"sharpness",lvl:255},{id:"bane_of_arthropods",lvl:255},{id:"smite",lvl:255},{id:"fire_aspect",lvl:1}]}
```

### 一些盔甲

```json
/give @p minecraft:chainmail_boots{display:{Name:'["天使的足履"]'},Enchantments:[{id:"blast_protection",lvl:5},{id:"binding_curse",lvl:1},{id:"depth_strider",lvl:3},{id:"feather_falling",lvl:255},{id:"fire_protection",lvl:2},{id:"mending",lvl:2},{id:"projectile_protection",lvl:3},{id:"protection",lvl:5},{id:"respiration",lvl:4},{id:"swift_sneak",lvl:5},{id:"soul_speed",lvl:3},{id:"thorns",lvl:5},{id:"unbreaking",lvl:3}],Unbreakable:1b,AttributeModifiers:[{AttributeName:"generic.max_health",Name:"generic.max_health",Amount:10,Operation:0,UUID:[I;1,1,3,1],Slot:"feet"},{AttributeName:"generic.max_health",Name:"generic.max_health",Amount:0.1,Operation:1,UUID:[I;1,1,3,2],Slot:"feet"},{AttributeName:"generic.knockback_resistance",Name:"generic.max_health",Amount:0.2,Operation:1,UUID:[I;1,1,3,3],Slot:"feet"},{AttributeName:"generic.movement_speed",Name:"generic.max_health",Amount:0.4,Operation:1,UUID:[I;1,1,3,5],Slot:"feet"},{AttributeName:"generic.attack_speed",Name:"generic.max_health",Amount:514,Operation:0,UUID:[I;1,1,3,7],Slot:"feet"},{AttributeName:"generic.armor",Operation:0,Amount:10d,Slot:"feet",Name:"Armor",UUID:[I;1,1,3,4]}]}
```

```json
/give @p minecraft:elytra{display:{Name:'["天使的羽翼"]'},Enchantments:[{id:"blast_protection",lvl:5},{id:"binding_curse",lvl:1},{id:"depth_strider",lvl:3},{id:"feather_falling",lvl:255},{id:"fire_protection",lvl:2},{id:"mending",lvl:2},{id:"projectile_protection",lvl:3},{id:"protection",lvl:5},{id:"respiration",lvl:4},{id:"swift_sneak",lvl:5},{id:"soul_speed",lvl:3},{id:"thorns",lvl:5},{id:"unbreaking",lvl:3}],Unbreakable:1b,AttributeModifiers:[{AttributeName:"generic.max_health",Name:"generic.max_health",Amount:10,Operation:0,UUID:[I;1,1,2,1],Slot:"chest"},{AttributeName:"generic.max_health",Name:"generic.max_health",Amount:0.1,Operation:1,UUID:[I;1,1,2,2],Slot:"chest"},{AttributeName:"generic.knockback_resistance",Name:"generic.max_health",Amount:0.2,Operation:1,UUID:[I;1,1,2,3],Slot:"chest"},{AttributeName:"generic.movement_speed",Name:"generic.max_health",Amount:0.4,Operation:1,UUID:[I;1,1,2,5],Slot:"chest"},{AttributeName:"generic.attack_speed",Name:"generic.max_health",Amount:514,Operation:0,UUID:[I;1,1,2,7],Slot:"chest"},{AttributeName:"generic.armor",Operation:0,Amount:10d,Slot:"chest",Name:"Armor",UUID:[I;1,1,2,4]}]}

```
