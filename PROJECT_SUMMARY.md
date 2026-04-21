# 贪吃蛇游戏项目总结

> 本项目由 Kimi Code AI 辅助开发完成，记录完整开发过程供学习参考。

---

## 一、项目概述

| 项目 | 内容 |
|------|------|
| 项目名称 | Snake Game（贪吃蛇） |
| 开发目标 | 编写一个可运行的贪吃蛇游戏，最终通过微信链接分享给朋友游玩 |
| 最终成果 | 纯 HTML+JavaScript 网页版游戏，部署在 GitHub Pages |
| 在线地址 | https://mynameispang.github.io/snake-game/snake.html |

---

## 二、开发完整过程

### 第一阶段：桌面版原型（Pygame）

**初始思路**
- 使用 Python + Pygame 快速实现一个可玩的贪吃蛇游戏
- 先验证核心玩法，再考虑如何分享给朋友

**代码结构**
- `Snake` 类：管理蛇的身体、移动方向、碰撞检测
- `Food` 类：管理食物生成和绘制
- `Game` 类：主循环（事件处理 → 更新逻辑 → 绘制画面）

**遇到的问题 1：字体加载崩溃**
- **现象**：`pygame.font.SysFont("simhei", 72)` 在 Windows 上报错 `TypeError: expected str, bytes or os.PathLike object, not int`
- **原因**：Pygame 扫描系统字体时遇到非字符串类型的注册表值，这是 Pygame 在 Windows 上的已知 bug
- **解决**：改用 `pygame.font.Font(None, 72)`，使用 Pygame 内置默认字体，不依赖系统字体

---

### 第二阶段：网页版转换（Pygbag）

**需求变化**
- 朋友无法直接运行 Python 文件，需要改成"点击链接就能玩"
- 选择 Pygbag 工具，将 Pygame 程序编译为浏览器可运行的 WebAssembly

**技术调整**
1. 主循环改为异步：`async def run()` + `await asyncio.sleep(0)`
2. 移除 `sys.exit()`，浏览器环境不允许直接退出进程
3. 添加屏幕虚拟方向键（手机触摸操作）
4. 窗口缩小到 640×480，更适合浏览器

**遇到的问题 2：CDN 加载超时**
- **现象**：浏览器打开后一直卡在 "Downloading..."
- **原因**：Pygbag 需要从国外 CDN (`pygame-web.github.io`) 下载 Python WebAssembly 运行时，国内网络访问极慢或失败
- **尝试解决**：
  - 等待加载 → 超时
  - 开代理/VPN → 不稳定
  - 本地缓存 CDN 文件 → 过于复杂
- **最终决策**：放弃 Pygbag 方案，改用纯 HTML + JavaScript 重写

---

### 第三阶段：纯网页版重写（HTML5 + JavaScript）

**重新设计**
- 完全抛弃 Pygame，使用 HTML5 Canvas + 原生 JavaScript
- 零外部依赖，一个 `.html` 文件搞定一切
- 加载秒开，微信内置浏览器也能正常运行

**核心代码结构**
```
snake.html
├── HTML 结构
│   ├── canvas 游戏画布
│   ├── 分数/最高分/速度显示
│   ├── 游戏结束遮罩层
│   └── 虚拟方向键（手机）
├── CSS 样式
│   ├── 深色主题配色
│   ├── 响应式布局（手机/电脑自适应）
│   └── 按钮触摸反馈
└── JavaScript 游戏逻辑
    ├── resetGame()      初始化/重置
    ├── placeFood()      生成食物（先收集空格再随机）
    ├── changeDirection() 改变方向（防反向）
    ├── update()          每帧更新（移动、碰撞、吃食物）
    ├── draw()            绘制画面（网格、蛇、食物、眼睛）
    └── gameLoop()        主循环（requestAnimationFrame）
```

**遇到的问题 3：食物生成失败**
- **现象**：吃到食物后没有新食物，有时开局也没有食物
- **原因**：`placeFood()` 使用 `while` 循环随机生成坐标，当蛇很长时可能一直随机到蛇身上，导致无限循环
- **解决**：改为"先遍历所有格子收集空格，再从中随机选一个"，保证食物一定生成

**遇到的问题 4：手机边界看不清**
- **现象**：手机屏幕纵向比例下，游戏面板左右边界不明显
- **解决**：
  - 加粗边框颜色（`#333` → `#666`）
  - 增加发光阴影效果
  - 添加 **Landscape / Portrait 布局切换按钮**，手机竖屏时可用更窄更高的面板（400×600）

**遇到的问题 5：触摸控制冲突**
- **现象**：手机滑动和点击虚拟按钮同时触发，方向混乱
- **解决**：
  - 虚拟按钮使用 `touchstart` + `preventDefault()` 阻止默认行为
  - 滑动控制增加最小距离阈值（20px），防止误触
  - 游戏结束点击屏幕重新开始

---

### 第四阶段：部署到 GitHub Pages

**步骤**
1. 在 `snake-game` 文件夹初始化 Git 仓库
2. 创建 GitHub 仓库 `mynameispang/snake-game`
3. `git push` 推送代码
4. Settings → Pages → 开启 Deploy from branch (main)
5. 获得链接：`https://mynameispang.github.io/snake-game/snake.html`

**遇到的问题 6：Git 仓库位置混乱**
- **现象**：`.git` 仓库在 `F:\KimiCode` 根目录，项目文件却散落在各处
- **解决**：
  - 将 `.git` 移入 `snake-game` 文件夹
  - 所有相关文件（`snake.html`, `snake_game.py`, `web/`）统一放入
  - 清理根目录残留文件夹

---

## 三、最终技术栈

| 层级 | 技术 | 说明 |
|------|------|------|
| 语言 | HTML5 + CSS3 + JavaScript | 纯前端，零依赖 |
| 渲染 | HTML5 Canvas 2D API | 绘制游戏画面 |
| 动画 | requestAnimationFrame | 流畅的游戏循环 |
| 存储 | localStorage | 本地保存最高分 |
| 部署 | GitHub Pages | 免费静态网站托管 |
| 版本控制 | Git | 代码版本管理 |

---

## 四、项目架构设计

### 4.1 游戏状态管理

```javascript
// 核心状态变量
let snake = [];           // 蛇身体坐标数组
let direction = {};       // 当前移动方向
let nextDirection = {};   // 下一帧要转的方向（防反向）
let food = {};            // 食物坐标
let score = 0;            // 当前分数
let highScore = 0;        // 最高分（localStorage）
let speed = 8;            // 游戏速度（帧/秒）
let gameOver = false;     // 游戏结束标志
```

### 4.2 游戏循环架构

```
requestAnimationFrame(gameLoop)
    ↓
gameLoop(timestamp)
    ↓
计算时间差 delta → 累积到 accumulator
    ↓
while (accumulator >= 1000/speed)
    ↓
    update()  // 更新游戏逻辑（移动、碰撞、吃食物）
    ↓
draw()  // 绘制一帧画面
```

**设计要点**：
- 使用 `accumulator` 累积时间，保证游戏速度稳定，不受帧率波动影响
- 速度可变：`speed = min(8 + score//50, 25)`，分数越高速度越快

### 4.3 输入处理架构

| 输入方式 | 事件 | 处理逻辑 |
|----------|------|----------|
| 键盘方向键 | `keydown` | `changeDirection(dx, dy)` |
| 键盘 WASD | `keydown` | 同上 |
| 空格重启 | `keydown` | `resetGame()` |
| 虚拟按钮点击 | `click` / `touchstart` | `changeDirection()` |
| 屏幕滑动 | `touchstart` + `touchend` | 计算滑动方向 → `changeDirection()` |
| 游戏结束点击 | `click` / `touchstart` | `resetGame()` |

**防反向机制**：
```javascript
if (dx !== -direction.x || dy !== -direction.y) {
    nextDirection = { x: dx, y: dy };  // 不能直接反向
}
```

### 4.4 食物生成算法

**旧算法（有 bug）**：
```javascript
// 随机生成，可能无限循环
do {
    food = { x: random(), y: random() };
} while (food 在蛇身上);
```

**新算法（正确）**：
```javascript
// 先收集所有空格，再随机选一个
const emptyCells = [];
for (每个格子) {
    if (格子不在蛇身上) emptyCells.push(格子);
}
const idx = Math.floor(Math.random() * emptyCells.length);
food = emptyCells[idx];
```

---

## 五、关键设计决策

| 决策 | 选择 | 理由 |
|------|------|------|
| 桌面版 → 网页版 | 纯 HTML+JS 重写 | Pygbag 国内网络不可用，HTML 零依赖更可靠 |
| 单文件 vs 多文件 | 单 HTML 文件 | 部署简单，GitHub Pages 直接托管 |
| 手机布局 | Landscape + Portrait 切换 | 不同握持方式下都能获得最佳体验 |
| 最高分存储 | localStorage | 无需服务器，纯前端实现 |
| 蛇眼睛绘制 | 根据方向动态计算 | 增加游戏趣味性 |

---

## 六、项目文件结构

```
snake-game/
├── snake.html          ← 最终网页版游戏（主文件，唯一必需文件）
└── PROJECT_SUMMARY.md  ← 本总结文档
```

> 注：早期开发过程中还创建了 `snake_game.py`（Pygame 桌面版）和 `web/`（Pygbag 打包文件），最终发布时已清理，只保留网页版核心文件。

---

## 七、学习收获

### 7.1 技术层面
1. **Canvas 绘图**：掌握 `fillRect`、`strokeRect`、`arc` 等基本绘图 API
2. **游戏循环**：理解 `requestAnimationFrame` + 固定时间步长的游戏循环模式
3. **触摸事件**：区分 `click`、`touchstart`、`touchend`，处理移动端输入
4. **响应式设计**：使用 CSS Media Query 和 JavaScript 动态调整布局
5. **Git 部署**：从本地开发到 GitHub Pages 的完整工作流

### 7.2 问题解决思路
1. **遇到 bug 先定位**：看报错信息 → 查相关代码 → 最小化复现
2. **方案走不通及时换**：Pygbag 网络问题无法解决，果断换技术路线
3. **用户反馈驱动优化**：根据实际手机测试反馈，逐步改进体验
4. **代码组织**：项目独立文件夹、Git 版本控制、文档记录

### 7.3 可改进方向
- [ ] 添加音效（吃食物、游戏结束）
- [ ] 添加暂停功能
- [ ] 添加多种难度级别
- [ ] 添加排行榜（需要后端服务器）
- [ ] PWA 支持（离线游玩、添加到主屏幕）

---

## 八、核心代码速查

### 8.1 游戏主循环
```javascript
function gameLoop(timestamp) {
    const delta = timestamp - lastTime;
    accumulator += delta;
    const interval = 1000 / speed;
    while (accumulator >= interval) {
        update();
        accumulator -= interval;
    }
    draw();
    requestAnimationFrame(gameLoop);
}
```

### 8.2 碰撞检测
```javascript
// 墙壁碰撞
if (head.x < 0 || head.x >= COLS || head.y < 0 || head.y >= ROWS) {
    endGame();
}
// 自身碰撞
if (snake.some(s => s.x === head.x && s.y === head.y)) {
    endGame();
}
```

### 8.3 食物生成
```javascript
function placeFood() {
    const emptyCells = [];
    for (let x = 0; x < COLS; x++) {
        for (let y = 0; y < ROWS; y++) {
            if (!snake.some(s => s.x === x && s.y === y)) {
                emptyCells.push({ x, y });
            }
        }
    }
    const idx = Math.floor(Math.random() * emptyCells.length);
    food = emptyCells[idx];
}
```

---

> 文档版本：v1.0
> 创建时间：2026-04-21
> 作者：Kimi Code AI 辅助开发
