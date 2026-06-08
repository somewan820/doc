**一、核心规则**

```
事件类型由 type 决定：
world    → 世界事件，最先判断
common   → 多人事件，其次判断
personal → 个人事件，最后判断

conditions 统一表示“全部满足才成立”：
- 空数组 [] = 无条件，默认成立
- 多个条件 = AND
- 不支持 OR / NOT / 嵌套 logic

事件处理结果由 outcomes 决定：
- outcomes 必须配置 priority
- priority 越高，越优先，越接近策划期望的真结局
```

**二、ID 命名规则**

| 类型         | 格式                  | 示例         | 说明         |
| ------------ | --------------------- | ------------ | ------------ |
| 个人事件     | A + 人物ID + 事件序号 | AP01E001     | 个人事件 ID  |
| 人物 ID      | P + 两位数字          | P01、P02     | 从 P01 开始  |
| 事件序号     | E + 三位数字          | E001         | 从 E001 开始 |
| 个人 outcome | 事件ID + O序号        | AP01E001O001 | 个人事件结果 |
| 多人事件     | C + 三位数字          | C001         | common 事件  |
| 世界事件     | W + 三位数字          | W001         | world 事件   |

**三、完整 JSON 示例**

```
{
  "version": "1.0.0",
  "events": [
    {
      "id": "W001",
      "type": "world",
      "title": "第七回合异变",
      "participants": [],
      "priority": 100,
      "once": true,
      "trigger": {
        "requirements": [],
        "conditions": [
          {
            "scope": "world",
            "attr": "round",
            "op": "==",
            "value": 7
          }
        ]
      },
      "outcomes": [
        {
          "id": "W001O001",
          "priority": 1,
          "conditions": [],
          "text": "第七回合开始时，世界局势发生了微妙变化。",
          "next": []
        }
      ]
    },
    {
      "id": "C001",
      "type": "common",
      "title": "共同探索遗迹",
      "participants": ["P01", "P02"],
      "priority": 80,
      "once": true,
      "trigger": {
        "requirements": [
          {
            "target": "P01",
            "eventId": "AP01E002"
          },
          {
            "target": "P02",
            "eventId": "AP02E004"
          }
        ],
        "conditions": [
          {
            "scope": "world",
            "attr": "round",
            "op": ">=",
            "value": 7
          },
          {
            "scope": "npc",
            "target": "P01",
            "attr": "attack",
            "op": ">=",
            "value": 50
          },
          {
            "scope": "npc",
            "target": "P02",
            "attr": "speed",
            "op": ">=",
            "value": 30
          }
        ]
      },
      "outcomes": [
        {
          "id": "C001O001",
          "priority": 100,
          "conditions": [],
          "text": "P01和P02一同前往城外，探索了那座被尘封许久的遗迹。",
          "next": [
            {
              "target": "P01",
              "nextId": "AP01E004"
            },
            {
              "target": "P02",
              "nextId": "AP02E005"
            }
          ]
        }
      ]
    },
    {
      "id": "AP01E001",
      "type": "personal",
      "title": "遗迹传闻",
      "participants": ["P01"],
      "priority": 10,
      "once": true,
      "trigger": {
        "requirements": [],
        "conditions": [
          {
            "scope": "world",
            "attr": "round",
            "op": ">=",
            "value": 7
          }
        ]
      },
      "outcomes": [
        {
          "id": "AP01E001O001",
          "priority": 100,
          "conditions": [
            {
              "scope": "npc",
              "target": "P01",
              "attr": "attack",
              "op": ">=",
              "value": 50
            },
            {
              "scope": "npc",
              "target": "P01",
              "attr": "defense",
              "op": ">=",
              "value": 40
            },
            {
              "scope": "npc",
              "target": "P01",
              "attr": "speed",
              "op": ">=",
              "value": 30
            }
          ],
          "text": "P01听闻遗迹传闻后，凭借出色的综合能力决定亲自前往调查。",
          "next": [
            {
              "target": "P01",
              "nextId": "AP01E002"
            }
          ]
        },
        {
          "id": "AP01E001O002",
          "priority": 1,
          "conditions": [],
          "text": "P01听闻遗迹传闻后，决定暂时继续观望。",
          "next": [
            {
              "target": "P01",
              "nextId": "AP01E003"
            }
          ]
        }
      ]
    }
  ]
}
```

**四、根字段**

| 字段    | 类型   | 必填 | 说明                                   |
| ------- | ------ | ---- | -------------------------------------- |
| version | string | 否   | 配置版本号                             |
| events  | array  | 是   | 所有事件配置，包含世界、多人、个人事件 |

**五、事件字段**

| 字段         | 类型     | 必填 | 说明                                   |
| ------------ | -------- | ---- | -------------------------------------- |
| id           | string   | 是   | 事件唯一 ID                            |
| type         | enum     | 是   | 事件类型：world、common、personal      |
| title        | string   | 否   | 策划和调试用标题                       |
| participants | string[] | 是   | 参与事件的 NPC；世界事件可为空数组     |
| priority     | int      | 是   | 同类型事件同时满足时，优先级高的先处理 |
| once         | bool     | 是   | 是否只能触发一次                       |
| trigger      | object   | 是   | 事件能否开始的判断                     |
| outcomes     | array    | 是   | 事件开始后可能进入的结果               |

**六、type 枚举**

| type     | 含义     | 调度方式                                                     |
| -------- | -------- | ------------------------------------------------------------ |
| world    | 世界事件 | 检查 trigger.conditions 是否满足                             |
| common   | 多人事件 | 检查多个 NPC 是否处于指定事件，并检查 trigger.conditions     |
| personal | 个人事件 | NPC 当前 currentEventId 等于事件 id 时，检查 trigger.conditions |

固定调度顺序：

```
world → common → personal
```

**七、trigger 字段**

| 字段         | 类型  | 必填 | 说明                           |
| ------------ | ----- | ---- | ------------------------------ |
| requirements | array | 是   | 事件进度要求，主要用于多人事件 |
| conditions   | array | 是   | 数值条件，默认 AND             |

个人事件通常：

```
"requirements": []
```

多人事件通常：

```
"requirements": [
  { "target": "P01", "eventId": "AP01E002" },
  { "target": "P02", "eventId": "AP02E004" }
]
```

**八、requirements 字段**

| 字段    | 类型   | 必填 | 说明                         |
| ------- | ------ | ---- | ---------------------------- |
| target  | string | 是   | 要检查的 NPC ID              |
| eventId | string | 是   | 该 NPC 当前必须处于的事件 ID |

含义示例：

```
{  "target": "P01",  "eventId": "AP01E002" }
```

表示：

```
P01.currentEventId 必须等于 AP01E002
```

**九、conditions 字段**

| 格式                   | 含义                     |
| ---------------------- | ------------------------ |
| []                     | 无条件，默认满足         |
| [condition]            | 单个条件满足             |
| [condition, condition] | 所有条件都满足，默认 AND |

示例：

```
"conditions": [
  {
    "scope": "npc",
    "target": "P01",
    "attr": "attack",
    "op": ">=",
    "value": 50
  },
  {
    "scope": "npc",
    "target": "P01",
    "attr": "defense",
    "op": ">=",
    "value": 40
  },
  {
    "scope": "npc",
    "target": "P01",
    "attr": "speed",
    "op": ">=",
    "value": 30
  }
]
```

含义：

```
P01.attack >= 50
且 P01.defense >= 40
且 P01.speed >= 30
```

**十、condition 单项字段**

| 字段   | 类型               | 必填   | 示例   | 说明                 |
| ------ | ------------------ | ------ | ------ | -------------------- |
| scope  | enum               | 是     | npc    | 判断对象范围         |
| target | string             | 视情况 | P01    | scope = npc 时必填   |
| attr   | string             | 是     | attack | 属性名，是字符串 key |
| op     | enum               | 是     | >=     | 比较符               |
| value  | number/string/bool | 是     | 50     | 比较值               |

**十一、scope 枚举**

| scope | 含义          | target 是否需要 | 示例             |
| ----- | ------------- | --------------- | ---------------- |
| npc   | 判断 NPC 属性 | 是              | P01.attack >= 50 |
| world | 判断世界状态  | 否              | round == 7       |

不使用 event scope。事件进度判断统一放在 requirements。

**十二、attr 推荐字段**

| attr    | scope | 含义       | 数据类型 |
| ------- | ----- | ---------- | -------- |
| round   | world | 当前回合数 | number   |
| attack  | npc   | 攻击       | number   |
| defense | npc   | 防御       | number   |
| speed   | npc   | 速度       | number   |

如果角色属性本身存在字典里：

```
{
  "attack": 60,
  "defense": 45,
  "speed": 32
}
```

那么 attr 就是字符串 key，调度器通过 npc.Attributes[attr] 取值。

**十三、op 枚举**

| op   | 含义     |
| ---- | -------- |
| >=   | 大于等于 |
| >    | 大于     |
| <=   | 小于等于 |
| <    | 小于     |
| ==   | 等于     |
| !=   | 不等于   |

**十四、outcome 字段**

| 字段       | 类型   | 必填 | 说明                            |
| ---------- | ------ | ---- | ------------------------------- |
| id         | string | 是   | outcome 唯一 ID                 |
| priority   | int    | 是   | 必填；越高越优先                |
| conditions | array  | 是   | 进入该 outcome 的条件，默认 AND |
| text       | string | 是   | 写入人物经历的文本              |
| next       | array  | 是   | 事件结束后更新 NPC 下一事件     |

默认 outcome 写法：

```
{
  "id": "AP01E001O999",
  "priority": 1,
  "conditions": [],
  "text": "P01决定暂时继续观望。",
  "next": [
    {
      "target": "P01",
      "nextId": "AP01E003"
    }
  ]
}
```

**十五、next 字段**

| 字段   | 类型   | 必填 | 说明                         |
| ------ | ------ | ---- | ---------------------------- |
| target | string | 是   | 要更新的 NPC                 |
| nextId | string | 是   | 该 NPC 下一回合进入的事件 ID |

**十六、调度规则**

```
每回合事件处理阶段：

1. 处理 world 事件
   - 按 priority 从高到低检查
   - trigger.conditions 全部满足则触发
   - 选择 priority 最高且 conditions 全部满足的 outcome

2. 处理 common 事件
   - 按 priority 从高到低检查
   - trigger.requirements 全部满足
   - trigger.conditions 全部满足
   - 选择 priority 最高且 conditions 全部满足的 outcome
   - 给 participants 中的 NPC 添加同一条经历文本
   - 被 common 处理过的 NPC，本轮不再处理 personal

3. 处理 personal 事件
   - 根据 NPC.currentEventId 找到事件
   - trigger.conditions 全部满足
   - 选择 priority 最高且 conditions 全部满足的 outcome
   - 给该 NPC 添加经历文本

4. 应用 next
   - outcome.next 写入 NPC.nextEventId
   - 回合事件处理结束后，nextEventId 覆盖 currentEventId
```

补充：

```
更新一下设定：
值	含义	是否参与判断
EVENT_FINISHED	个人线结束，但 NPC 仍可能被世界/多人事件拉回或参与判断	是
EVENT_INACTIVE	NPC 完全退场，不再参与任何事件条件判断	否

当回合若进行了多人事件，则不会再进行对应npc的单人事件
使用方式：

"next": []
如果某事件 outcome 没有给参与者配置 next，该参与者会自动进入 EVENT_FINISHED。

如果要让 NPC 完全退场，可以显式写：

"next": [
  {
    "target": "P01",
    "nextId": "EVENT_INACTIVE"
  }
]
完全退场后的NPC在当回合不会再来商店，如果想让NPC退场且当回合来商店收个尾，建议写EVENT_FINISHED
```

