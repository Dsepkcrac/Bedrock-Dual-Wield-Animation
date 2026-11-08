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