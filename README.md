# 《清明上河图》数字长卷与北宋百业图鉴 (Qingming Scroll Interactive)

专为 iPhone 移动端打造的高帧率《清明上河图》互动长卷单页应用。支持《汴京夜宴》昼夜双态光影切换、视口动态瓦片裁剪渲染，以及北宋“酒楼茶肆、医疗香药、交通运输、手工作坊”四大百业图鉴与长卷锚点的双向联动。

## 核心技术架构

- **60fps 视口裁剪引擎**：`<canvas>` 物理尺寸严格锁定屏幕视口，基于一维包围盒（1D AABB）剔除屏幕外元素，并将长卷切分为 `512px` 宽的动态瓦片缓存池，确保 iOS Safari 显存占用恒定在 3 屏以内。
- **昼夜双态光影着色**：读取设备本地时间（支持手动切换）驱动离屏 Lightmap 缓冲区。采用 `multiply`（正片叠底）叠加青黛夜色遮罩，结合 `destination-out` 局部还原与 `lighter` 三段式暖橙径向泛光，模拟烛火摇曳与汴河水面碎金倒影。
- **百分比相对坐标契约**：所有建筑锚点与光源均采用 `{ x: 0~100, y: 0~100 }` 相对百分比坐标绑定长卷世界坐标系，确保在任意手机屏幕尺寸与横竖屏切换下零偏移。

## 目录结构

```text
├── public/
│   ├── data/
│   │   └── metadata.json       # 核心数据契约（瓦片信息、分类配置、POI坐标与光源）
│   └── tiles/                  # 自动化管线生成的 512px 切片 (tile_0.webp ~ tile_N.webp)
├── scripts/
│   └── build_assets.py         # 素材抓取、色温归一化、切片与元数据清洗脚本
├── src/
│   ├── engine/                 # Canvas 视口裁剪、惯性滚动与昼夜光影渲染器
│   ├── ui/                     # 移动端分类筛选栏、图鉴卡片与双向联动控制器
│   └── main.js                 # 应用入口
├── tests/
│   └── mobile.spec.js          # Playwright iPhone 视口自动化验收测试
└── index.html
```

## 核心数据契约 (`public/data/metadata.json`)

所有模块开发必须严格遵循以下 JSON Schema：

```json
{
  "scrollMeta": {
    "totalWidth": 24000,
    "totalHeight": 1200,
    "tileSize": 512,
    "tileCount": 47
  },
  "categories": [
    { "id": "food", "name": "酒楼茶肆", "color": "#C83E37" },
    { "id": "medical", "name": "医疗香药", "color": "#2E7D6B" },
    { "id": "transport", "name": "交通运输", "color": "#2B5F8E" },
    { "id": "craft", "name": "手工作坊", "color": "#D48817" }
  ],
  "pois": [
    {
      "id": "poi_sunyang",
      "title": "孙羊正店",
      "categoryId": "food",
      "section": "城门街肆段",
      "coords": { "x": 82.5, "y": 46.0 },
      "lights": [
        { "x": 82.1, "y": 44.5, "radius": 90, "isWaterfront": false }
      ],
      "summary": "北宋东京七十二正店之一，拥有官方酿酒特许权，门口扎有彩楼欢门。"
    }
  ]
}
```

## AI 协作开发规范 (Devin Workflow)

1. 按 Issue 编号顺序或模块边界独立创建分支（如 `feat/asset-pipeline`、`feat/canvas-engine`、`feat/mobile-ui`）。
2. 严禁修改 `metadata.json` 的字段命名；若需扩充 POI 条目，必须通过校验脚本。
3. 发起 PR 前必须确保 `npx playwright test` 在移动端视口下全部通过。
