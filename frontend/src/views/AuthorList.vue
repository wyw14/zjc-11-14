<template>
  <div class="author-list">
    <div class="container">
      <section class="hero card">
        <div class="hero-left">
          <h1>👨‍💻 作者广场</h1>
          <p class="hero-desc">
            查看所有参与创作的作者，了解他们的贡献和精彩作品。
          </p>
        </div>
        <div class="hero-stats">
          <div class="stat-item">
            <span class="stat-num">{{ authors.length }}</span>
            <span class="stat-label">作者总数</span>
          </div>
          <div class="stat-item">
            <span class="stat-num">{{ totalEntries }}</span>
            <span class="stat-label">总段落数</span>
          </div>
          <div class="stat-item">
            <span class="stat-num">{{ totalChars }}</span>
            <span class="stat-label">总字数</span>
          </div>
        </div>
      </section>

      <section class="list-section">
        <div class="section-header">
          <h2>📋 作者排行榜</h2>
          <span class="sort-tip">按累计字数排序</span>
        </div>

        <div v-if="loading" class="loading card">正在加载作者列表...</div>

        <div v-else-if="authors.length === 0" class="empty card">
          <div class="empty-icon">✍️</div>
          <p>还没有作者，快去创建第一个故事吧！</p>
          <router-link to="/" class="btn-primary" style="margin-top: 16px; display: inline-block;">
            去创作
          </router-link>
        </div>

        <div v-else class="author-grid">
          <router-link
            v-for="(author, index) in authors"
            :key="author.name"
            :to="{ name: 'author', query: { name: author.name } }"
            class="author-card"
          >
            <div class="author-rank" :class="getRankClass(index)">
              {{ index < 3 ? getRankIcon(index) : index + 1 }}
            </div>
            <div class="author-avatar" :style="{ background: avatarColor(author.name) }">
              {{ author.name.slice(0, 1) }}
            </div>
            <div class="author-info">
              <h3 class="author-name">{{ author.name }}</h3>
              <div class="author-stats">
                <span class="stat-mini">
                  <span class="stat-mini-icon">📚</span>
                  {{ author.storyCount }} 故事
                </span>
                <span class="stat-mini">
                  <span class="stat-mini-icon">✏️</span>
                  {{ author.entryCount }} 段
                </span>
                <span class="stat-mini">
                  <span class="stat-mini-icon">📝</span>
                  {{ author.totalChars }} 字
                </span>
              </div>
              <div class="author-time">
                最近活跃 {{ formatTime(author.lastActiveAt) }}
              </div>
            </div>
            <div class="author-arrow">→</div>
          </router-link>
        </div>
      </section>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'
import api from '../api.js'
import { formatTime } from '../utils.js'

const authors = ref([])
const loading = ref(false)

const avatarColors = [
  '#6366f1', '#ec4899', '#10b981', '#f59e0b', '#3b82f6',
  '#8b5cf6', '#ef4444', '#14b8a6', '#f97316', '#06b6d4'
]

const totalEntries = computed(() =>
  authors.value.reduce((sum, a) => sum + a.entryCount, 0)
)

const totalChars = computed(() =>
  authors.value.reduce((sum, a) => sum + a.totalChars, 0)
)

function avatarColor(name) {
  let hash = 0
  for (let i = 0; i < name.length; i++) {
    hash = (hash * 31 + name.charCodeAt(i)) >>> 0
  }
  return avatarColors[hash % avatarColors.length]
}

function getRankClass(index) {
  if (index === 0) return 'rank-gold'
  if (index === 1) return 'rank-silver'
  if (index === 2) return 'rank-bronze'
  return ''
}

function getRankIcon(index) {
  if (index === 0) return '🥇'
  if (index === 1) return '🥈'
  if (index === 2) return '🥉'
  return index + 1
}

async function loadData() {
  loading.value = true
  try {
    authors.value = await api.getAuthors()
  } catch (e) {
    console.error(e)
  } finally {
    loading.value = false
  }
}

onMounted(loadData)
</script>

<style scoped>
.hero {
  display: flex;
  gap: 40px;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 32px;
  background: linear-gradient(135deg, #eef2ff 0%, #fdf2f8 100%);
  border-color: #e0e7ff;
}

.hero-left {
  flex: 1;
}

.hero h1 {
  font-size: 28px;
  margin-bottom: 12px;
  color: var(--text);
}

.hero-desc {
  color: var(--text-muted);
  font-size: 15px;
  max-width: 500px;
}

.hero-stats {
  display: flex;
  gap: 24px;
  flex-shrink: 0;
}

.stat-item {
  text-align: center;
  padding: 16px 24px;
  background: var(--surface);
  border-radius: var(--radius);
  box-shadow: var(--shadow-sm);
  min-width: 90px;
}

.stat-num {
  display: block;
  font-size: 28px;
  font-weight: 700;
  background: linear-gradient(135deg, var(--primary), var(--accent));
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.stat-label {
  font-size: 12px;
  color: var(--text-muted);
}

.section-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 16px;
}

.section-header h2 {
  font-size: 20px;
}

.sort-tip {
  font-size: 13px;
  color: var(--text-muted);
}

.author-grid {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.author-card {
  display: flex;
  align-items: center;
  gap: 16px;
  padding: 20px;
  background: var(--surface);
  border: 1px solid var(--border);
  border-radius: var(--radius);
  transition: all 0.25s;
  position: relative;
}

.author-card:hover {
  transform: translateX(4px);
  box-shadow: var(--shadow-lg);
  border-color: var(--primary-light);
}

.author-rank {
  width: 36px;
  height: 36px;
  border-radius: 50%;
  background: var(--surface-alt);
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: 700;
  font-size: 16px;
  color: var(--text-muted);
  flex-shrink: 0;
}

.author-rank.rank-gold {
  background: linear-gradient(135deg, #fbbf24, #f59e0b);
  color: white;
  font-size: 20px;
}

.author-rank.rank-silver {
  background: linear-gradient(135deg, #9ca3af, #6b7280);
  color: white;
  font-size: 20px;
}

.author-rank.rank-bronze {
  background: linear-gradient(135deg, #f97316, #ea580c);
  color: white;
  font-size: 20px;
}

.author-avatar {
  width: 56px;
  height: 56px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  color: white;
  font-size: 22px;
  font-weight: 700;
  flex-shrink: 0;
  box-shadow: var(--shadow-sm);
}

.author-info {
  flex: 1;
  min-width: 0;
}

.author-name {
  font-size: 17px;
  font-weight: 600;
  color: var(--text);
  margin-bottom: 6px;
}

.author-stats {
  display: flex;
  flex-wrap: wrap;
  gap: 12px;
  margin-bottom: 4px;
}

.stat-mini {
  display: flex;
  align-items: center;
  gap: 4px;
  font-size: 13px;
  color: var(--text-muted);
}

.stat-mini-icon {
  font-size: 14px;
}

.author-time {
  font-size: 12px;
  color: var(--text-light);
}

.author-arrow {
  color: var(--text-light);
  font-size: 20px;
  flex-shrink: 0;
  transition: transform 0.2s;
}

.author-card:hover .author-arrow {
  transform: translateX(4px);
  color: var(--primary);
}

@media (max-width: 640px) {
  .hero {
    flex-direction: column;
    gap: 24px;
  }
  .hero-stats {
    width: 100%;
    justify-content: space-around;
  }
  .author-rank {
    width: 28px;
    height: 28px;
    font-size: 13px;
  }
  .author-avatar {
    width: 44px;
    height: 44px;
    font-size: 18px;
  }
  .author-stats {
    gap: 8px;
  }
  .stat-mini {
    font-size: 12px;
  }
}
</style>
