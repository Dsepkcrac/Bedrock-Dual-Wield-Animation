# 基岩版双手持物视觉动画教程（TwoHandHold）

[![Minecraft Bedrock](https://img.shields.io/badge/Minecraft-Bedrock%20Edition-green)](https://www.minecraft.net/)
[![format_version](https://img.shields.io/badge/format__version-1.20.50-blue)](https://wiki.bedrock.dev/)

一个面向 Minecraft 基岩版（Bedrock Edition）的附加包（Addon）示例项目，用于讲解如何通过覆盖**玩家动画**（Player Animation）与**动画控制器**（Animation Controller），在第三人称视角下实现“双手持物”的视觉效果，例如双手持枪、双手握剑。

本项目的思路是修改**玩家自身的骨骼动画**，而不是修改物品模型。

## 目录

- [效果展示](#效果展示)
- [快速开始](#快速开始)
- [实现原理](#实现原理)
- [目录结构](#目录结构)
- [分步实现](#分步实现)
  - [步骤一：定义触发物品](#步骤一定义触发物品)
  - [步骤二：编写玩家动画](#步骤二编写玩家动画)
  - [步骤三：编写动画控制器](#步骤三编写动画控制器)
  - [步骤四：将动画挂载到玩家](#步骤四将动画挂载到玩家)
  - [步骤五：配置资源包与行为包清单](#步骤五配置资源包与行为包清单)
- [常见问题](#常见问题)
- [参考资料](#参考资料)
- [许可证](#许可证)

## 效果展示

![双手持物效果示意](images/demo.png)

*图 1：第三人称视角下的双手持枪姿势。*

## 快速开始

### 环境要求

- Minecraft 基岩版 **1.20.50** 及以上版本。
- 创建世界时开启**“假日创作者功能”（Holiday Creator Features）**等实验性选项；自定义物品与玩家动画挂载依赖该开关。

### 安装步骤

1. 打开仓库的 `releases` 目录，下载 `TwoHandHold.mcaddon` 文件。
2. 双击该文件，Minecraft 会自动导入资源包（RP）与行为包（BP）。
3. 新建世界，在“资源包”和“行为包”页面中分别启用导入的两个包。
4. 进入游戏，在创造模式物品栏中找到“双手持握测试物”并手持，切换到第三人称视角即可看到效果。

## 实现原理

实现双手持物**不需要修改物品模型**，而是覆盖玩家自身的骨骼动画。整个流程分为三个环节：

| 环节 | 文件位置 | 作用 |
| --- | --- | --- |
| 定义物品 | `BP/items/` | 提供一把专属物品，作为触发动画的“开关” |
| 编写动画 | `RP/animations/` | 让左右手臂在第三人称下摆出持物姿势 |
| 动画控制器 | `RP/animation_controllers/` | 结合 Molang 判断手持物品，替换原版默认动作 |

三者的关系是：动画控制器依据 Molang 查询结果切换状态，状态中播放自定义动画，最终覆盖玩家手臂的默认姿态。

## 目录结构

```text
TwoHandHold/
├── BP/                                  # 行为包
│   ├── manifest.json
│   └── items/
│       └── two_hand_item.json
└── RP/                                  # 资源包
    ├── manifest.json
    ├── animations/
    │   └── two_hand_hold.animation.json
    ├── animation_controllers/
    │   └── two_hand_hold.animation_controllers.json
    ├── entity/
    │   └── player.entity.json
    ├── textures/
    │   └── items/
    │       └── two_hand_item.png
    └── texts/
        └── zh_CN.lang
```

## 分步实现

### 步骤一：定义触发物品

在 `BP/items/two_hand_item.json` 中定义名为 `demo:gun` 的自定义物品。玩家手持该物品时，即触发后续的自定义动画。

```json
{
  "format_version": "1.20.50",
  "minecraft:item": {
    "description": {
      "identifier": "demo:gun",
      "menu_category": {
        "category": "items"
      }
    },
    "components": {
      "minecraft:icon": {
        "texture": "two_hand_item"
      },
      "minecraft:display_name": {
        "value": "双手持握测试物"
      },
      "minecraft:max_stack_size": 1
    }
  }
}
```

几点说明：

- `minecraft:icon` 中的 `two_hand_item` 是纹理短名，需在 `RP/textures/item_texture.json` 中映射到 `textures/items/two_hand_item`。
- 物品标识符必须带命名空间（如 `demo:gun`），且不能与既有的原版物品重名。
- `menu_category` 决定物品出现在创造模式物品栏的哪个分类中。

### 步骤二：编写玩家动画

在 `RP/animations/two_hand_hold.animation.json` 中为玩家双臂编写持物姿势。手臂骨骼名为 `rightArm` 与 `leftArm`，旋转角度采用角度制，顺序为 `[x, y, z]`。

```json
{
  "format_version": "1.8.0",
  "animations": {
    "animation.player.two_hand_hold": {
      "loop": true,
      "animation_length": 1.0,
      "bones": {
        "rightArm": {
          "rotation": [-75, -15, 0]
        },
        "leftArm": {
          "rotation": [-75, 15, 0]
        }
      }
    }
  }
}
```

要点：

- 动画仅调整手臂旋转，躯干与头部保持原版动作，观感更自然。
- 具体角度需按武器模型的实际朝向微调；必要时可同时对 `rightItem`、`leftItem` 骨骼做位移或旋转补偿。
- 文件名建议与动画名保持一致，便于后续维护。

### 步骤三：编写动画控制器

在 `RP/animation_controllers/two_hand_hold.animation_controllers.json` 中，根据玩家手持物品切换动画状态。

```json
{
  "format_version": "1.10.0",
  "animation_controllers": {
    "controller.animation.player.two_hand_hold": {
      "initial_state": "default",
      "states": {
        "default": {
          "transitions": [
            {
              "hold": "query.get_equipped_item_name == 'gun'"
            }
          ]
        },
        "hold": {
          "animations": [
            "two_hand_hold"
          ],
          "transitions": [
            {
              "default": "query.get_equipped_item_name != 'gun'"
            }
          ]
        }
      }
    }
  }
}
```

要点：

- `query.get_equipped_item_name` 返回不带命名空间的物品短名，因此判断 `gun` 而非 `demo:gun`。
- 状态内 `animations` 数组中的 `two_hand_hold` 是短名，需在客户端实体文件中映射到动画全名。
- 动画控制器会持续覆盖原版手臂动画，无需额外设置动画优先级。

### 步骤四：将动画挂载到玩家

在 `RP/entity/player.entity.json` 中登记动画与控制器，并通过 `scripts.animate` 启用控制器。

```json
{
  "format_version": "1.10.0",
  "minecraft:client_entity": {
    "description": {
      "identifier": "minecraft:player",
      "animations": {
        "two_hand_hold": "animation.player.two_hand_hold",
        "two_hand_hold_controller": "controller.animation.player.two_hand_hold"
      },
      "scripts": {
        "animate": [
          "two_hand_hold_controller"
        ]
      }
    }
  }
}
```

覆盖原版 `player.entity.json` 时，应保留文件原有的其余字段，仅追加 `animations` 与 `scripts` 内容，避免玩家渲染出现异常。

### 步骤五：配置资源包与行为包清单

两个包各需一份 `manifest.json`，其中 `header.uuid` 与 `modules[].uuid` 不能重复，且资源包需通过 `dependencies` 声明对行为包的依赖：

```json
{
  "format_version": 2,
  "header": {
    "name": "TwoHandHold BP",
    "description": "双手持物示例项目（行为包）",
    "uuid": "00000000-0000-0000-0000-000000000000",
    "version": [1, 0, 0],
    "min_engine_version": [1, 20, 50]
  },
  "modules": [
    {
      "type": "data",
      "uuid": "00000000-0000-0000-0000-000000000001",
      "version": [1, 0, 0]
    }
  ]
}
```

示例中的 UUID 必须替换为实际生成的值（任意 UUID v4 生成器均可）。

## 常见问题

**Q1：为什么只有第三人称才有双手持物效果？**

玩家骨骼动画作用于第三人称模型。第一人称手臂由独立的渲染逻辑控制，需另行制作第一人称动画或调整相机，才能在第一人称下看到相同姿势。

**Q2：为什么不用挂件（attachable）/ 物品模型来实现？**

挂件只能改变物品自身的外观，无法改变手臂姿态。双手持物的关键在于让两只手同时指向持物点，因此必须覆盖玩家动画。

**Q3：动画完全不生效，如何排查？**

按以下顺序检查：

1. 两个包的 `manifest.json` 是否被游戏正确识别，UUID 是否重复或格式错误。
2. 世界中是否开启了“假日创作者功能”等实验性选项。
3. `player.entity.json` 中的动画短名与控制器短名是否一致，`scripts.animate` 是否引用了控制器。
4. Molang 判断的字符串是否与 `query.get_equipped_item_name` 的返回值一致（不带命名空间）。
5. 是否被其他更高层级的动画或模型覆盖。

**Q4：手持长剑时朝向不对怎么办？**

先调整 `rightItem` / `leftItem` 骨骼的旋转，再微调手臂角度。建议在游戏中逐帧对照武器握持点，避免仅凭经验值盲调。

## 参考资料

- [Microsoft Learn：Minecraft Creator 文档](https://learn.microsoft.com/minecraft/creator/)
- [Bedrock Wiki：附加包开发指南](https://wiki.bedrock.dev/)
- [Blockbench：模型与动画编辑器](https://www.blockbench.net/)

## 许可证

本项目暂未声明开源许可证。如需公开分发，建议补充 `LICENSE` 文件（例如 MIT 许可证），并同步更新页面顶部的徽章信息。
