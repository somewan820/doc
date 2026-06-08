# 物品 JSON 格式规约

## 一、核心规则

物品 JSON 用于配置游戏内所有可展示、可购买或可结算属性的物品数据。

- 每个物品必须拥有唯一的 `ItemId`。
- `ItemId` 使用字符串格式，从 `"0001"` 开始递增。
- 所有字段名必须保持固定写法，区分大小写。
- `Icon` 暂时为空字符串 `""`，后续接入图标资源后再填写资源路径或资源名。
- `Rarity` 只能填写规约中定义的五种字符串之一。
- 数值字段统一使用整数，不使用小数。

## 二、完整 JSON 示例

```json
{
  "ItemId": "0001",
  "DisplayName": "青竹短剑",
  "Price": 120,
  "Icon": "",
  "Description": "以青竹与寒铁打磨而成的短剑，剑身轻巧，适合初入江湖者随身携带。",
  "Unlocked By Default": false,
  "Rarity": "Common",
  "Attack": 8,
  "Defense": 0,
  "MovementSpeed": 1
}
```

## 三、字段说明

| 字段 | 类型 | 必填 | 默认值 | 说明 |
| --- | --- | --- | --- | --- |
| `ItemId` | string | 是 | 无 | 物品唯一 ID，格式为四位数字字符串，从 `"0001"` 开始递增 |
| `DisplayName` | string | 是 | 无 | 物品显示名称 |
| `Price` | int | 是 | 无 | 物品价格，必须为整数 |
| `Icon` | string | 是 | `""` | 物品图标资源标识；当前为空字符串 |
| `Description` | string | 是 | 无 | 物品描述文本，用一段话说明物品外观、用途或背景 |
| `Unlocked By Default` | bool | 是 | `false` | 是否默认解锁 |
| `Rarity` | enum/string | 是 | 无 | 物品稀有度，只能填写指定枚举值 |
| `Attack` | int | 是 | 无 | 攻击属性加成，必须为整数 |
| `Defense` | int | 是 | 无 | 防御属性加成，必须为整数 |
| `MovementSpeed` | int | 是 | 无 | 移动速度属性加成，必须为整数 |

## 四、ItemId 命名规则

| 规则 | 示例 | 说明 |
| --- | --- | --- |
| 四位数字字符串 | `"0001"` | 第一个物品 ID |
| 递增编号 | `"0002"`、`"0003"` | 后续物品依次递增 |
| 不使用纯数字 | `0001` 不推荐 | JSON 中必须写成字符串 `"0001"` |

## 五、Rarity 枚举

| 枚举值 | 含义 |
| --- | --- |
| `Common` | 普通 |
| `Fine` | 精良 |
| `Superior` | 优秀 |
| `Epic` | 史诗 |
| `Immortal` | 仙品 |

## 六、字段填写要求

### DisplayName

- 使用玩家可见的物品名称。
- 名称应简短清晰，避免过长。

### Price

- 必须填写整数。
- 价格为 `0` 时表示免费或不可通过价格结算，具体含义由业务逻辑决定。

### Icon

- 当前统一填写为空字符串：

```json
"Icon": ""
```

### Description

- 使用一段完整的描述文本。
- 描述应说明物品特征、用途、来历或氛围感。
- 不建议只写过短文本，例如“武器”“装备”等。

### Unlocked By Default

- 默认填写：

```json
"Unlocked By Default": false
```

- 如果该物品在游戏初始阶段即可使用，则填写 `true`。

### Attack、Defense、MovementSpeed

- 三个字段都必须填写整数。
- 没有对应加成时填写 `0`。
- 字段含义如下：

| 字段 | 含义 | 示例 |
| --- | --- | --- |
| `Attack` | 攻击加成 | `8` |
| `Defense` | 防御加成 | `3` |
| `MovementSpeed` | 移动速度加成 | `1` |

## 七、多个物品配置示例

如果需要在同一个文件中配置多个物品，推荐使用数组结构：

```json
[
  {
    "ItemId": "0001",
    "DisplayName": "青竹短剑",
    "Price": 120,
    "Icon": "",
    "Description": "以青竹与寒铁打磨而成的短剑，剑身轻巧，适合初入江湖者随身携带。",
    "Unlocked By Default": false,
    "Rarity": "Common",
    "Attack": 8,
    "Defense": 0,
    "MovementSpeed": 1
  },
  {
    "ItemId": "0002",
    "DisplayName": "旧皮护甲",
    "Price": 180,
    "Icon": "",
    "Description": "由深色皮革缝制而成的护甲，虽然已有磨损，但仍能提供基础防护。",
    "Unlocked By Default": false,
    "Rarity": "Fine",
    "Attack": 0,
    "Defense": 6,
    "MovementSpeed": 0
  }
]
```
