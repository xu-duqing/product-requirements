# BestPronounce 首页 UI 重新设计需求文档

**版本**: v1.0
**日期**: 2026-03-05
**优先级**: P0

---

## 问题背景

**当前首页问题：**
1. 视觉吸引力不足 - 纯文字，没有视觉元素
2. 空白过多 - 搜索框上下都是大面积空白
3. 热门词不突出 - 小标签样式，不吸引点击
4. 用户看不到发音信息 - 无法判断页面价值

**核心目标：**
- 填充空白，提升视觉吸引力
- 展示发音信息，让用户立即判断价值
- 提升热门词点击率

---

## 设计方案

### 推荐方案：卡片式热门词

#### 首屏布局

```
┌──────────────────────────────────────────────────────┐
│  BestPronounce         🔊 How to Pronounce Anything  │
└──────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────┐
│  Learn Perfect Pronunciation                          │
│  Search for any word, name, or phrase                │
│                                                       │
│  [ Search box                     ] [EN ▼] [ Search ] │
└──────────────────────────────────────────────────────┘

Popular Words

┌────────────┐  ┌────────────┐  ┌────────────┐
│ atorva-    │  │ how to     │  │ cerave     │
│ statin     │  │ pronounce  │  │ 怎么读     │
│            │  │            │  │            │
│ /a-tor-    │  │ /pla-      │  │ /se-reiv/  │
│  va-sta-   │  │  beɪk/     │  │            │
│   tin/     │  │            │  │            │
│            │  │            │  │            │
│     🔊     │  │     🔊     │  │     🔊     │
│   [ES]     │  │   [EN]     │  │   [ZH]     │
└────────────┘  └────────────┘  └────────────┘

┌────────────┐  ┌────────────┐  ┌────────────┐
│ acai       │  │ croissant  │  │ wegovy     │
│ /a-sai/    │  │ /kwa-san/  │  │ /we-go-vi/ │
│            │  │            │  │            │
│     🔊     │  │     🔊     │  │     🔊     │
│   [PT]     │  │   [FR]     │  │   [EN]     │
└────────────┘  └────────────┘  └────────────┘
```

---

## 组件设计

### 1. 热门词卡片 (WordCard)

**布局：**
```
┌────────────┐
│  word      │  ← 词名（大号，加粗）
│            │
│  /phonetic/│  ← 音标（中等，灰色）
│            │
│     🔊     │  ← 播放按钮（圆形，蓝色）
│   [LANG]   │  ← 语言标签（小圆点 + 代码）
└────────────┘
```

**尺寸：**
- 宽度：200px
- 高度：160px
- 圆角：12px
- 内边距：20px

**颜色：**
- 背景：白色
- 边框：1px solid #e2e8f0
- 词名：#1e293b（深灰）
- 音标：#64748b（中灰）
- 语言标签：#2563eb（蓝色）

**字体：**
- 词名：24px, 700, Inter
- 音标：16px, 400, monospace
- 语言标签：12px, 600, Inter

**交互：**
- 鼠标悬停：
  - 卡片放大：transform: scale(1.02)
  - 阴影增强：box-shadow: 0 8px 24px rgba(0, 0, 0, 0.12)
  - 边框颜色：#2563eb
- 点击卡片：跳转到详情页
- 点击播放按钮：播放发音（不跳转）

---

### 2. 语言标签

**样式：**
```
┌──────────┐
│   🔵 EN  │
└──────────┘
```

**布局：**
- 小圆点（8px）+ 语言代码
- 位于卡片右下角

**语言代码：**
- EN - English
- ZH - 中文
- ES - Español
- DE - Deutsch
- FR - Français
- PT - Português
- IT - Italiano
- JP - 日本語

---

### 3. 搜索框

**调整：**
- 提升位置，减少上方空白
- 缩小副标题字体
- 增加搜索框阴影

**尺寸：**
- 最大宽度：600px
- 高度：50px

---

## 响应式设计

### Desktop (> 1200px)
- 热门词：3列 x 2行（6个）
- 卡片宽度：200px

### Tablet (768px - 1200px)
- 热门词：2列 x 3行（6个）
- 卡片宽度：180px

### Mobile (< 768px)
- 热门词：1列 x 6行（6个）
- 卡片宽度：100%
- 搜索框：全宽

---

## 数据结构

### API 响应

```json
{
  "popular_words": [
    {
      "word": "atorvastatin",
      "phonetic": "a-tor-va-sta-tin",
      "language": "es",
      "audio_url": "/audio/atorvastatin-es.mp3",
      "search_count": 322
    },
    {
      "word": "how to pronounce",
      "phonetic": "pla-beik",
      "language": "en",
      "audio_url": "/audio/how-to-pronounce-en.mp3",
      "search_count": 29
    },
    {
      "word": "cerave",
      "phonetic": "se-reiv",
      "language": "zh",
      "audio_url": "/audio/cerave-zh.mp3",
      "search_count": 20
    }
  ]
}
```

---

## 功能需求

### P0 - 热门词卡片

**功能：**
1. 从 API 获取热门词列表
2. 显示音标（如果可用）
3. 显示语言标签
4. 显示播放按钮
5. 点击卡片 → 跳转详情页
6. 点击播放 → 播放发音（不跳转）

**实现：**

创建 `WordCard.vue` 组件：

```vue
<template>
  <div class="word-card" @click="goToDetail">
    <div class="word-title">{{ word.word }}</div>
    <div class="word-phonetic" v-if="word.phonetic">
      /{{ word.phonetic }}/
    </div>
    <div class="word-footer">
      <button
        class="play-btn"
        @click.stop="playAudio"
        :disabled="isPlaying || isLoading"
      >
        <span v-if="isLoading">⏳</span>
        <span v-else-if="isPlaying">⏸️</span>
        <span v-else>🔊</span>
      </button>
      <span class="language-tag">{{ word.language }}</span>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import { useRouter } from 'vue-router'

const props = defineProps<{
  word: {
    word: string
    phonetic?: string
    language: string
    audio_url?: string
  }
}>()

const router = useRouter()
const isPlaying = ref(false)
const isLoading = ref(false)

const goToDetail = () => {
  const langMap = {
    en: 'word',
    zh: 'wordZh',
    es: 'wordEs',
    de: 'word',
    fr: 'word',
    pt: 'word',
    it: 'word',
    jp: 'wordJp'
  }
  router.push({
    name: langMap[props.word.language] || 'word',
    params: { word: props.word.word }
  })
}

const playAudio = async () => {
  if (!props.word.audio_url || isPlaying.value || isLoading.value) return

  isLoading.value = true
  try {
    const audio = new Audio(props.word.audio_url)
    isPlaying.value = true
    await audio.play()
    audio.addEventListener('ended', () => {
      isPlaying.value = false
    })
  } catch (error) {
    console.error('Audio playback error:', error)
  } finally {
    isLoading.value = false
  }
}
</script>

<style scoped>
.word-card {
  background: white;
  border: 1px solid #e2e8f0;
  border-radius: 12px;
  padding: 20px;
  cursor: pointer;
  transition: all 0.3s ease;
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.word-card:hover {
  transform: scale(1.02);
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.12);
  border-color: #2563eb;
}

.word-title {
  font-size: 24px;
  font-weight: 700;
  color: #1e293b;
  text-align: center;
}

.word-phonetic {
  font-size: 16px;
  font-family: monospace;
  color: #64748b;
  text-align: center;
  flex-grow: 1;
  display: flex;
  align-items: center;
  justify-content: center;
}

.word-footer {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.play-btn {
  width: 36px;
  height: 36px;
  border-radius: 50%;
  border: none;
  background: #2563eb;
  color: white;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 18px;
  transition: all 0.3s ease;
}

.play-btn:hover:not(:disabled) {
  background: #1d4ed8;
  transform: scale(1.1);
}

.play-btn:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

.language-tag {
  font-size: 12px;
  font-weight: 600;
  color: #2563eb;
  background: #dbeafe;
  padding: 4px 8px;
  border-radius: 4px;
}
</style>
```

---

### P1 - 首屏布局调整

**调整：**

1. 缩小标题和副标题
2. 提升搜索框位置
3. 热门词卡片网格布局
4. 减少上下间距

**参考尺寸：**
- 标题：32px → 28px
- 副标题：16px → 14px
- 搜索框上方间距：40px → 20px
- 热门词上方间距：60px → 40px

---

### P2 - 分类热门词（可选）

**功能：**
- 按词类型分类（药物、食物、品牌、地名）
- 显示分类标签和图标
- 用户切换分类

**布局：**

```
💊 Medications  🍽️ Foods  🏷️ Brands  📍 Places

┌────────────┐  ┌────────────┐  ┌────────────┐
│ atorva-    │  │ lipitor    │  │ crestor    │
│ statin     │  │ ...        │  │ ...        │
└────────────┘  └────────────┘  └────────────┘
```

---

## 颜色方案

### 主色调
- Primary: #2563eb（蓝色）
- Hover: #1d4ed8（深蓝）

### 中性色
- Background: #f8fafc（浅灰）
- Card Background: #ffffff（白色）
- Border: #e2e8f0（浅灰）
- Text Primary: #1e293b（深灰）
- Text Secondary: #64748b（中灰）

### 语义色
- Success: #10b981（绿色）
- Warning: #f59e0b（橙色）
- Error: #ef4444（红色）

---

## 实施计划

### Week 1
- [ ] 创建 WordCard 组件
- [ ] API 接口扩展（热门词 + 音标 + 语言）
- [ ] 首屏布局调整
- [ ] 搜索框样式优化

### Week 2
- [ ] 响应式设计
- [ ] 动画效果（悬停、点击）
- [ ] 性能优化（音频预加载）
- [ ] A/B 测试（不同卡片样式）

---

## 成功指标

### 短期（1周）
- 首页视觉吸引力评分提升 30%
- 热门词点击率从 ~2% 提升到 10%

### 中期（2周）
- 整体 CTR > 15%
- 页面停留时间 +20%
- 返回率降低 15%

---

## 不做的功能

- ❌ 复杂的分类筛选
- ❌ 用户自定义热门词
- ❌ 热门词排序切换
- ❌ 更多热门词加载（分页）

---

## 风险与缓解

| 风险 | 概率 | 影响 | 缓解措施 |
|------|------|------|----------|
| 音标数据缺失 | 中 | 中 | 降级显示词名，不显示音标 |
| 音频 API 成本 | 中 | 中 | 只在点击时加载，缓存热门词 |
| 移动端显示问题 | 低 | 中 | 充分测试响应式设计 |

---

## 备注

**专注目标：填充空白，展示发音信息，提升点击率。**

做少，但做到极致。