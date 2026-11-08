# 基岩版双手持物视觉动画教程 (TwoHandHold Addon)

[![Minecraft Bedrock](https://img.shields.io/badge/Minecraft-Bedrock-green)](https://www.minecraft.net/)
[![Format Version](https://img.shields.io/badge/format__version-1.20.50-blue)]()

欢迎来到本教程！这是一个为 Minecraft 基岩版（Bedrock Edition）制作的附加包（Addon）开源示例项目。

本项目将教你如何通过修改**玩家自身的动画**（Player Animation）和**动画控制器**（Animation Controller），实现**第三人称视角下的“双手持物”视觉效果**（例如双手持枪、双手握剑等）。而不是仅仅修改物品的模型。

---

## ✨ 效果展示

*(建议：在仓库里新建一个 `images` 文件夹，上传你的游戏截图命名为 `demo.png`，然后把下面这行括号里的文字替换掉)*

<!-- ![双手持握效果](images/demo.png) -->
> *此处应有第三人称双手持物效果图*

---

## 📥 快速测试与安装

如果你想直接进游戏体验效果：
1. 点击上方目录进入 `releases` 文件夹。
2. 下载 `TwoHandHold.mcaddon` 文件。
3. 双击下载的文件，Minecraft 会自动导入资源包（RP）和行为包（BP）。
4. 创建新世界，**务必开启“假日创作者功能”等实验性选项**，并在创造模式物品栏找到“双手持握测试物”。

---

## 🧠 核心实现思路解析

在基岩版中，实现双手持物**不是去修改物品的模型**，而是**修改玩家自身的骨骼动画**。

主要分为三个步骤：
1. **定义物品**：定义一把专属物品，作为触发动作的“开关”。
2. **编写动画**：编写玩家手臂动画，让左右手在第三人称下摆出特定姿势。
3. **逻辑挂载**：利用动画控制器和 Molang，让玩家在拿着这把物品时，覆盖掉原版默认的动作。

---

## 📂 代码详细拆解

### 第一步：定义触发物品 (行为包 BP)

在 `BP/items/two_hand_item.json` 中，我们定义了一个名为 `demo:gun` 的物品。当玩家手持这个物品时，会触发我们的自定义动画。

```json
{
  "format_version": "1.20.50",
  "minecraft:item": {
    "description": {
      "identifier": "demo:gun",
      "menu_category": { "category": "items" }
    },
    "components": {
      "minecraft:icon": { "texture": "two_hand_item" },
      "minecraft:display_name": { "value": "双手持握测试物" },
      "minecraft:max_stack_size": 1
    }
  }
}
💡 提示：记得在资源包的 textures/item_texture.json 中注册 two_hand_item 贴图，否则游戏内会显示紫黑方块。

第二步：制作双手持握动画 (资源包 RP)
在 RP/animations/player_armor.animation.json 中，我们为玩家的 rightArm（右手）和 leftArm（左手）设定了旋转角度。

json
{
	"format_version": "1.8.0",
	"animations": {
		"animation.player.gun": {
			"loop": true,
			"bones": {
				"rightArm": { "rotation": ["-90 - this", 0, 0] },
				"leftArm": { "rotation": [-77.7968, 51.8659, 9.654] }
			}
		}
	}
}
⚠️ 注意：骨骼名称极其敏感，必须严格使用 rightArm 和 leftArm。你可以使用 Blockbench 直观地调整这些旋转数值并导出。

第三步：挂载动画控制器 (资源包 RP)
在 RP/animation_controllers/player.animation_controllers.json 中，我们新建了一个独立的控制器 controller.animation.player.gun。
它的逻辑非常纯粹：只要玩家手里拿着短名为 gun 的物品，就执行动画。

json
"controller.animation.player.gun": {
  "initial_state": "default",
  "states": {
    "default": {
      "animations": [
        {
          "gun": "query.get_equipped_item_name == 'gun'"
        }
      ]
    }
  }
}
第四步：将控制器注入玩家主逻辑 (资源包 RP)
这是最关键的一步！在 RP/entity/player.entity.json 中，我们修改了原版的玩家实体定义。

1. 在 animations 列表中，注册我们新建的动画和控制器：

json
"gun": "animation.player.gun",
"gun_controller": "controller.animation.player.gun"
2. 在 third_person 状态（第三人称）下，通过 Molang 让它在拿枪时生效：

json
{
  "gun_controller": "query.get_equipped_item_name == 'gun'"
}
因为我们是在 third_person 状态的动画列表最末尾追加了 gun_controller，所以它会覆盖原版默认的 holding 动画。

⚠️ 新人避坑指南
严格区分大小写与英文：所有文件路径、动画名、骨骼名请永远使用纯英文小写，不能使用中文。

副手触发的冲突：目前的逻辑 query.get_equipped_item_name == 'gun' 没有指定是哪只手。如果你想让逻辑更严谨（比如只有主手拿枪时才触发），可以修改为 query.get_equipped_item_name('main_hand') == 'gun'。

动画优先级：如果发现动作没生效，请检查你的控制器是否放到了正确的状态（third_person）下，以及 player.entity.json 的层级嵌套是否正确。

📜 开源协议
本项目基于 MIT License 开源，你可以自由修改、分发并用于自己的作品中。

如果这个教程帮到了你，欢迎点个 Star ⭐！
