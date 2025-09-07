<script setup>
import { ref, reactive } from 'vue'

const owner = ref('')
const repo = ref('')
const stargazers = ref([])
const loading = ref(false)
const error = ref('')

// 键名中文映射
const keyMapping = {
  'login': '用户名',
  'id': 'ID',
  'node_id': '节点ID',
  'avatar_url': '头像URL',
  'gravatar_id': 'Gravatar ID',
  'url': 'API URL',
  'html_url': 'GitHub URL',
  'followers_url': '关注者URL',
  'following_url': '正在关注URL',
  'gists_url': 'Gists URL',
  'starred_url': '星标URL',
  'subscriptions_url': '订阅URL',
  'organizations_url': '组织URL',
  'repos_url': '仓库URL',
  'events_url': '事件URL',
  'received_events_url': '接收事件URL',
  'type': '类型',
  'user_view_type': '用户视图类型',
  'site_admin': '站点管理员'
}

// 获取星标用户
const fetchStargazers = async () => {
  if (!owner.value || !repo.value) {
    error.value = '请输入用户名和仓库名'
    return
  }
  
  loading.value = true
  error.value = ''
  stargazers.value = []
  
  try {
    const response = await fetch(`https://api.github.com/repos/${owner.value}/${repo.value}/stargazers`, {
      headers: {
        'Accept': 'application/vnd.github.v3+json'
      }
    })
    
    if (!response.ok) {
      throw new Error(`获取失败: ${response.status} ${response.statusText}`)
    }
    
    const data = await response.json()
    stargazers.value = data
  } catch (err) {
    error.value = err.message
  } finally {
    loading.value = false
  }
}
</script>

<template>
  <div class="container">
    <h1>GitHub 星标查询工具</h1>
    
    <div class="search-form">
      <div class="input-group">
        <label for="owner">用户名:</label>
        <input
          id="owner"
          v-model="owner"
          type="text"
          placeholder="输入 GitHub 用户名"
        />
      </div>
      
      <div class="input-group">
        <label for="repo">仓库名:</label>
        <input
          id="repo"
          v-model="repo"
          type="text"
          placeholder="输入仓库名"
        />
      </div>
      
      <button @click="fetchStargazers" :disabled="loading">
        {{ loading ? '查询中...' : '查询星标用户' }}
      </button>
    </div>
    
    <div v-if="error" class="error-message">{{ error }}</div>
    
    <div v-if="stargazers.length > 0" class="result-section">
      <h2>星标用户列表 (共 {{ stargazers.length }} 人)</h2>
      <div class="table-wrapper">
        <table>
          <thead>
            <tr>
              <th>用户名</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="user in stargazers" :key="user.id">
              <td class="user-info">
                <img :src="user.avatar_url" :alt="user.login" class="avatar" />
                <a :href="user.html_url" target="_blank" class="username">{{ user.login }}</a>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
    
    <div v-else-if="!loading && owner && repo" class="empty-state">
      <p>暂无星标用户数据</p>
    </div>
  </div>
</template>

<style scoped>
.container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
  text-align: center;
}

h1 {
  font-size: 2.5em;
  margin-bottom: 1em;
  color: #333;
}

h2 {
  font-size: 1.8em;
  margin: 1.5em 0 1em;
  color: #444;
}

.search-form {
  display: flex;
  flex-direction: column;
  gap: 15px;
  max-width: 600px;
  margin: 0 auto 30px;
}

.input-group {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  gap: 5px;
}

.input-group label {
  font-weight: 500;
  color: #555;
}

input {
  width: 100%;
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 1em;
}

button {
  background-color: #646cff;
  color: white;
  border: none;
  padding: 12px 24px;
  border-radius: 4px;
  font-size: 1em;
  cursor: pointer;
  transition: background-color 0.2s;
}

button:hover:not(:disabled) {
  background-color: #535bf2;
}

button:disabled {
  background-color: #ccc;
  cursor: not-allowed;
}

.error-message {
  color: #e53e3e;
  background-color: #fed7d7;
  padding: 12px;
  border-radius: 4px;
  margin: 20px 0;
}

.result-section {
  margin-top: 30px;
}

.table-wrapper {
  overflow-x: auto;
  border-radius: 4px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

table {
  width: 100%;
  border-collapse: collapse;
  background-color: white;
}

th {
  background-color: #f7fafc;
  padding: 12px;
  text-align: left;
  font-weight: 600;
  color: #4a5568;
  border-bottom: 2px solid #e2e8f0;
}

td {
  padding: 12px;
  border-bottom: 1px solid #e2e8f0;
  text-align: left;
}

.user-info {
  display: flex;
  align-items: center;
  gap: 12px;
  min-width: 200px;
}

.avatar {
  width: 40px;
  height: 40px;
  border-radius: 50%;
  object-fit: cover;
}

.username {
  font-weight: 500;
  color: #646cff;
  text-decoration: none;
}

.username:hover {
  text-decoration: underline;
}

.details {
  max-width: 600px;
}

.detail-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
  gap: 8px;
}

.detail-item {
  display: flex;
  flex-direction: column;
  gap: 2px;
  font-size: 0.9em;
}

.detail-key {
  font-weight: 500;
  color: #718096;
}

.detail-value {
  color: #2d3748;
  word-break: break-all;
}

.detail-value.url a {
  color: #646cff;
  text-decoration: none;
}

.detail-value.url a:hover {
  text-decoration: underline;
}

.empty-state {
  padding: 40px;
  color: #718096;
  font-style: italic;
}

/* 响应式设计 */
@media (max-width: 768px) {
  .container {
    padding: 10px;
  }
  
  h1 {
    font-size: 2em;
  }
  
  h2 {
    font-size: 1.5em;
  }
  
  .detail-grid {
    grid-template-columns: 1fr;
  }
  
  table {
    font-size: 0.9em;
  }
  
  .user-info {
    flex-direction: column;
    align-items: flex-start;
    gap: 8px;
  }
}
</style>
