# BestPronounce SEO + 内容优化需求文档

**版本**: v1.0
**日期**: 2026-03-05
**优先级**: P0

---

## 问题背景

**核心问题：曝光高，点击率极低**

当前数据：
- `atorvastatin`: 322 曝光，0 点击
- `cómo se pronuncia información`: 613 曝光，1 点击
- `wegovy aussprache`: 68 曝光，2 点击

---

## 根因分析

### 1. Sitemap 结构问题

**现状：**
- 所有页面 priority = 1.0（无优先级差异）
- 所有页面 lastmod = 2025-10-09（内容过时）
- 缺少 sitemap index（只有一个文件）

**影响：**
- 搜索引擎无法判断页面重要性
- 抓取效率低
- 内容被认为过期

---

### 2. 内容问题（根本原因）

**用户搜索"发音"，但点击前得不到任何答案。**

从 sitemap 看，页面是 `/es/{word}` 格式，但搜索结果页不显示：
- 音标/拼音
- 发音音频
- 示例句子
- 语言标签

用户看到的是：只有 URL，没有描述，没有预览。

**为什么要点击？**

---

## 产品策略

**SEO + 内容 = 点击**

让用户在搜索结果页直接看到发音信息，降低决策成本。

---

## 功能需求

### SEO 优化

#### P0 - Sitemap 重构

**目标：** 添加优先级分级和更新时间

**实现：**

##### 1. Sitemap Index

创建 `sitemap-index.xml`：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<sitemapindex xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
  <sitemap>
    <loc>https://www.bestpronounce.com/sitemap-en.xml</loc>
    <lastmod>2026-03-05</lastmod>
  </sitemap>
  <sitemap>
    <loc>https://www.bestpronounce.com/sitemap-es.xml</loc>
    <lastmod>2026-03-05</lastmod>
  </sitemap>
  <sitemap>
    <loc>https://www.bestpronounce.com/sitemap-zh.xml</loc>
    <lastmod>2026-03-05</lastmod>
  </sitemap>
</sitemapindex>
```

##### 2. 单语言 Sitemap（Priority 分级）

```xml
<!-- 首页 -->
<url>
  <loc>https://www.bestpronounce.com/</loc>
  <lastmod>2026-03-05</lastmod>
  <priority>1.0</priority>
  <changefreq>daily</changefreq>
</url>

<!-- 高频词（搜索量 > 100） -->
<url>
  <loc>https://www.bestpronounce.com/es/atorvastatin</loc>
  <lastmod>2026-03-05</lastmod>
  <priority>0.8</priority>
  <changefreq>weekly</changefreq>
</url>

<!-- 普通词 -->
<url>
  <loc>https://www.bestpronounce.com/es/aaliyah</loc>
  <lastmod>2026-02-20</lastmod>
  <priority>0.5</priority>
  <changefreq>monthly</changefreq>
</url>
```

**Priority 分级规则：**
- 1.0: 首页
- 0.8: 高频词（搜索量 > 100）
- 0.6: 中频词（搜索量 10-100）
- 0.5: 普通词（搜索量 < 10）
- 0.3: 低质量页面

---

#### P1 - Meta 标签优化

**目标：** 动态生成有效的 meta 描述

**实现：**

每个词页面的动态 meta：

```html
<!-- 示例：atorvastatin -->
<title>How to Pronounce Atorvastatin - Audio Guide | BestPronounce</title>
<meta name="description" content="Learn how to pronounce atorvastatin correctly. Listen to native Spanish pronunciation, phonetic spelling: /a-tor-va-sta-tin/, and example sentences.">
<meta property="og:title" content="How to Pronounce Atorvastatin">
<meta property="og:description" content="Audio pronunciation guide with phonetic spelling and examples">
<meta property="og:type" content="website">
<meta property="og:image" content="https://www.bestpronounce.com/og/atorvastatin.png">
<meta property="og:url" content="https://www.bestpronounce.com/es/atorvastatin">
```

**规则：**
- Title: `{Word} Pronunciation - Audio Guide | BestPronounce`
- Description: 包含词、音标、语言、示例
- OG Image: 动态生成词的发音卡片

---

#### P2 - 结构化数据（Schema.org）

**目标：** 让搜索引擎理解内容类型

**实现：**

```html
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "HowTo",
  "name": "How to Pronounce Atorvastatin",
  "description": "Step-by-step guide to pronounce atorvastatin correctly in Spanish",
  "step": [
    {
      "@type": "HowToStep",
      "text": "Listen to the audio pronunciation",
      "url": "https://www.bestpronounce.com/es/atorvastatin"
    },
    {
      "@type": "HowToStep",
      "text": "Repeat after the speaker",
      "position": 2
    }
  ],
  "inLanguage": "es",
  "tool": [
    {
      "@type": "AudioObject",
      "name": "Atorvastatin Pronunciation",
      "contentUrl": "https://www.bestpronounce.com/audio/atorvastatin.mp3"
    }
  ]
}
</script>
```

---

### 内容优化

#### P0 - 首屏发音信息显示

**目标：** 用户点击进来后，立即看到发音信息

**实现：**

```
┌─────────────────────────────────────────┐
│  How to Pronounce: ATORVASTATIN         │
│                                         │
│  /a-tor-va-sta-tin/          ▶ Play     │
│                                         │
│  Example:                              │
│  "El médico recetó atorvastatin."      │
│                          ▶ Play         │
└─────────────────────────────────────────┘
```

**关键要素：**
1. 词名（大写）
2. 音标/拼音
3. 播放按钮（首屏可见）
4. 一个例句（带播放）
5. 语言标签（右上角）

---

#### P1 - 内容差异化

**目标：** 避免重复内容，提供有价值的上下文

**实现：**

针对不同词类型提供不同内容：

**1. 药物名**：
- 药物用途
- 发音技巧
- 相关词（同类药物）

**示例（atorvastatin）：**
```
Atorvastatin

/a-tor-va-sta-tin/  ▶ Play

About This Word:
Atorvastatin is a prescription medication used to lower cholesterol.
It belongs to a class of drugs called statins.

Pronunciation Tips:
- Stress the third syllable: a-TOR-va-sta-tin
- The "t" is soft, like in "water"
- The "z" is pronounced like an "s" sound

Example Sentences:
1. El médico recetó atorvastatin.
2. Toma atorvastatin antes de dormir.

Related Words:
- Cholesterol: /ko-les-te-rol/
- Statins: /es-ta-tins/
```

**2. 品牌名**：
- 品牌背景
- 发音来源
- 相关产品

**3. 食物名**：
- 食物描述
- 起源国家
- 相关菜谱

**4. 地名**：
- 地理位置
- 文化背景
- 著名景点

---

#### P2 - 多语言相关词推荐

**目标：** 提升页面内链权重

**实现：**

页面底部显示：
```
Related Pronunciations:
- Lipitor (atorvastatin brand name)
- Cholesterol
- Simvastatin
- Crestor
```

---

### 技术优化

#### P0 - 多语言 hreflang

**目标：** 告诉搜索引擎多语言关系

**实现：**

```html
<link rel="alternate" hreflang="en" href="https://www.bestpronounce.com/en/atorvastatin">
<link rel="alternate" hreflang="es" href="https://www.bestpronounce.com/es/atorvastatin">
<link rel="alternate" hreflang="zh" href="https://www.bestpronounce.com/zh/atorvastatin">
<link rel="alternate" hreflang="ja" href="https://www.bestpronounce.com/ja/atorvastatin">
<link rel="alternate" hreflang="x-default" href="https://www.bestpronounce.com/en/atorvastatin">
```

---

#### P1 - 页面性能优化

**目标：** 提升加载速度，影响 SEO

**实现：**

1. SSG/SSR 确保首屏快速渲染
2. 音频懒加载
3. 图片优化（WebP 格式）
4. 启用 CDN
5. 压缩资源

---

#### P2 - 面包屑导航

**目标：** 提升用户体验和 SEO 结构

**实现：**

```
Home > Spanish Pronunciation > Atorvastatin
```

```html
<nav aria-label="Breadcrumb">
  <ol vocab="https://schema.org/" typeof="BreadcrumbList">
    <li property="itemListElement" typeof="ListItem">
      <a property="item" href="https://www.bestpronounce.com/">
        <span property="name">Home</span>
      </a>
      <meta property="position" content="1">
    </li>
    <li property="itemListElement" typeof="ListItem">
      <a property="item" href="https://www.bestpronounce.com/es/">
        <span property="name">Spanish</span>
      </a>
      <meta property="position" content="2">
    </li>
    <li property="itemListElement" typeof="ListItem">
      <span property="name">Atorvastatin</span>
      <meta property="position" content="3">
    </li>
  </ol>
</nav>
```

---

## 数据追踪

### 关键指标

1. **CTR 趋势**
   - 曝光 vs 点击
   - 不同词类型的 CTR
   - 不同语言的 CTR

2. **页面性能**
   - 加载时间
   - 首次内容绘制（FCP）
   - 交互时间（TTI）

3. **用户行为**
   - 音频播放率
   - 页面停留时间
   - 跳出率
   - 深度滚动率

---

## 实施计划

### Week 1
- [ ] Sitemap 重构（priority 分级）
- [ ] Meta 标签优化
- [ ] 首屏发音信息显示

### Week 2
- [ ] 结构化数据添加
- [ ] hreflang 标签
- [ ] 内容差异化（药物/品牌/食物/地名）

### Week 3
- [ ] 面包屑导航
- [ ] 多语言相关词推荐
- [ ] 数据埋点

### Week 4
- [ ] 性能优化
- [ ] A/B 测试（不同首屏布局）
- [ ] 监控和调优

---

## 成功指标

### 短期（2周）
- 整体 CTR 从 <2% 提升到 10%
- 高频词 CTR > 15%

### 中期（1月）
- 整体 CTR > 15%
- 搜索排名提升 20%
- 页面停留时间 +30%

---

## 不做的功能

- ❌ 复杂的用户系统
- ❌ 收藏夹
- ❌ 社交分享（简单链接即可）
- ❌ 评分评论
- ❌ 语音搜索

---

## 风险与缓解

| 风险 | 概率 | 影响 | 缓解措施 |
|------|------|------|----------|
| 内容生成成本高 | 中 | 高 | AI 辅助生成 + 人工审核 |
| 音频 API 成本 | 中 | 中 | 缓存热门词音频 |
| SEO 效果延迟 | 高 | 中 | 监控数据，快速迭代 |

---

## 备注

**专注一件事：让用户最快听到发音。**

做少，但做到极致。