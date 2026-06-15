<template>
  <div class="author-profile">
    <div class="container">
      <div class="back-bar">
        <router-link to="/" class="back-link">← 返回故事广场</router-link>
        <router-link to="/authors" class="back-link" style="margin-left: 16px;">← 作者列表</router-link>
      </div>

      <div v-if="loading" class="loading card">正在加载作者信息...</div>

      <div v-else-if="!profile" class="empty card">
        <div class="empty-icon">❓</div>
        <p>作者不存在或暂无贡献</p>
        <router-link to="/authors" class="btn-primary" style="margin-top: 16px; display: inline-block;">
          查看作者列表
        </router-link>
      </div>

      <template v-else>
        <header class="author-header card">
          <div class="author-avatar" :style="{ background: avatarColor }">
            {{ profile.name.slice(0, 1) }}
          </div>
          <div class="author-info">
            <h1 class="author-name">{{ profile.name }}</h1>
            <p class="author-time">
              活跃于 {{ formatTime(profile.firstActiveAt) }} - {{ formatTime(profile.lastActiveAt) }}
            </p>
          </div>
        </header>

        <section class="stats-section">
          <div class="stat-card card">
            <div class="stat-icon">📚</div>
            <div class="stat-content">
              <span class="stat-num">{{ profile.storyCount }}</span>
              <span class="stat-label">参与故事</span>
            </div>
          </div>
          <div class="stat-card card">
            <div class="stat-icon">✏️</div>
            <div class="stat-content">
              <span class="stat-num">{{ profile.entryCount }}</span>
              <span class="stat-label">写过段落</span>
            </div>
          </div>
          <div class="stat-card card">
            <div class="stat-icon">📝</div>
            <div class="stat-content">
              <span class="stat-num">{{ profile.totalChars }}</span>
              <span class="stat-label">累计字数</span>
            </div>
          </div>
        </section>

        <section class="stories-section card">
          <h2 class="section-title">📖 参与的故事</h2>
          <div v-if="profile.stories.length === 0" class="empty-mini">
            暂无参与的故事
          </div>
          <div v-else class="story-list">
            <div
              v-for="story in profile.stories"
              :key="story.id"
              class="story-item"
            >
              <div class="story-item-header">
                <router-link :to="`/story/${story.id}`" class="story-link">
                  {{ story.title }}
                </router-link>
                <span :class="['tag', story.locked ? 'tag-success' : 'tag-warning']">
                  {{ story.locked ? '已完结' : '接龙中' }}
                </span>
              </div>
              <div class="story-item-meta">
                <span class="meta-item">
                  <span class="meta-icon">✏️</span>
                  贡献 {{ story.entryCount }} 段
                </span>
                <span class="meta-item">
                  <span class="meta-icon">📝</span>
                  {{ story.totalChars }} 字
                </span>
                <span class="meta-item">
                  <span class="meta-icon">🕐</span>
                  最近 {{ formatTime(story.lastEntryAt) }}
                </span>
              </div>
              <div class="story-entries">
                <h4 class="entries-title">该作者的段落：</h4>
                <div
                  v-for="entry in story.entries"
                  :key="entry.id"
                  class="entry-item"
                >
                  <div class="entry-header">
                    <span class="entry-order">第 {{ entry.order }} 棒</span>
                    <span class="entry-meta">
                      {{ formatTime(entry.createdAt) }} · {{ entry.contentLength }} 字
                    </span>
                  </div>
                  <p class="entry-content">{{ entry.content }}</p>
                </div>
              </div>
            </div>
          </div>
        </section>
      </template>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, watch } from 'vue'
import { useRoute } from 'vue-router'
import api from '../api.js'
import { formatTime } from '../utils.js'

const route = useRoute()
const profile = ref(null)
const loading = ref(false)

const avatarColors = [
  '#6366f1', '#ec4899', '#10b981', '#f59e0b', '#3b82f6',
  '#8b5cf6', '#ef4444', '#14b8a6', '#f97316', '#06b6d4'
]

const avatarColor = computed(() => {
  if (!profile.value) return avatarColors[0]
  let hash = 0
  for (let i = 0; i < profile.value.name.length; i++) {
    hash = (hash * 31 + profile.value.name.charCodeAt(i)) >>> 0
  }
  return avatarColors[hash % avatarColors.length]
})

async function loadProfile() {
  loading.value = true
  try {
    profile.value = await api.getAuthor(route.params.name)
  } catch (e) {
    console.error(e)
    profile.value = null
  } finally {
    loading.value = false
  }
}

watch(() => route.params.name, loadProfile)
onMounted(loadProfile)
</script>

<style scoped>
.back-bar {
  margin-bottom: 16px;
  display: flex;
  gap: 8px;
}

.back-link {
  font-size: 14px;
  color: var(--text-muted);
}

.back-link:hover {
  color: var(--primary);
}

.author-header {
  display: flex;
  align-items: center;
  gap: 24px;
  margin-bottom: 20px;
  background: linear-gradient(135deg, #eef2ff 0%, #fdf2f8 100%);
  border-color: #e0e7ff;
}

.author-avatar {
  width: 80px;
  height: 80px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  color: white;
  font-size: 32px;
  font-weight: 700;
  flex-shrink: 0;
  box-shadow: var(--shadow);
}

.author-info {
  flex: 1;
}

.author-name {
  font-size: 28px;
  font-weight: 700;
  margin-bottom: 8px;
  color: var(--text);
}

.author-time {
  color: var(--text-muted);
  font-size: 14px;
}

.stats-section {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 16px;
  margin-bottom: 20px;
}

.stat-card {
  display: flex;
  align-items: center;
  gap: 16px;
  padding: 20px;
}

.stat-icon {
  font-size: 36px;
  width: 56px;
  height: 56px;
  display: flex;
  align-items: center;
  justify-content: center;
  background: var(--surface-alt);
  border-radius: var(--radius);
  flex-shrink: 0;
}

.stat-content {
  display: flex;
  flex-direction: column;
}

.stat-num {
  font-size: 28px;
  font-weight: 700;
  background: linear-gradient(135deg, var(--primary), var(--accent));
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  line-height: 1.2;
}

.stat-label {
  font-size: 13px;
  color: var(--text-muted);
  margin-top: 4px;
}

.section-title {
  font-size: 17px;
  margin-bottom: 20px;
  padding-bottom: 12px;
  border-bottom: 1px solid var(--border);
}

.story-list {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.story-item {
  padding: 20px;
  background: var(--surface-alt);
  border-radius: var(--radius);
  border: 1px solid var(--border);
}

.story-item-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 12px;
  margin-bottom: 12px;
}

.story-link {
  font-size: 18px;
  font-weight: 600;
  color: var(--primary);
}

.story-link:hover {
  text-decoration: underline;
}

.story-item-meta {
  display: flex;
  flex-wrap: wrap;
  gap: 16px;
  margin-bottom: 16px;
  font-size: 13px;
  color: var(--text-muted);
}

.meta-item {
  display: flex;
  align-items: center;
  gap: 4px;
}

.meta-icon {
  font-size: 14px;
}

.story-entries {
  padding-top: 16px;
  border-top: 1px dashed var(--border);
}

.entries-title {
  font-size: 14px;
  color: var(--text-muted);
  margin-bottom: 12px;
  font-weight: 600;
}

.entry-item {
  padding: 12px 16px;
  background: var(--surface);
  border-radius: var(--radius-sm);
  margin-bottom: 10px;
  border-left: 3px solid var(--primary);
}

.entry-item:last-child {
  margin-bottom: 0;
}

.entry-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 8px;
}

.entry-order {
  font-size: 12px;
  font-weight: 600;
  color: var(--primary);
  background: rgba(99, 102, 241, 0.1);
  padding: 2px 8px;
  border-radius: 999px;
}

.entry-meta {
  font-size: 12px;
  color: var(--text-light);
}

.entry-content {
  font-size: 14px;
  color: var(--text);
  line-height: 1.8;
  white-space: pre-wrap;
  word-break: break-word;
}

.empty-mini {
  text-align: center;
  padding: 40px;
  color: var(--text-muted);
}

@media (max-width: 640px) {
  .stats-section {
    grid-template-columns: 1fr;
  }
  .author-header {
    flex-direction: column;
    text-align: center;
  }
  .story-item-meta {
    flex-direction: column;
    gap: 6px;
  }
}
</style>
