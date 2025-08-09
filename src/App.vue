<script setup>
import { ref, onMounted, nextTick, computed } from 'vue'
import axios from 'axios'
import { 
  MagnifyingGlassIcon, 
  PlusIcon, 
  TrashIcon,
  CheckCircleIcon
} from '@heroicons/vue/24/outline'

const value = ref('')
const list = ref([])
const loading = ref(true)
const showSearch = ref(false)
const searchQuery = ref('')
const toast = ref({ show: false, message: '', type: 'success' })

// 获取列表数据
async function getList() {
  try {
    loading.value = true
    const response = await axios({
      url: 'https://e2knygut8e.bja.sealos.run/get_list',
      method: 'GET',
      timeout: 10000,
      headers: {
        'Content-Type': 'application/json'
      }
    })
    if (response.data.code === 0) {
      list.value = response.data.data
    } else {
      showToast('获取列表失败', 'error')
    }
  } catch (error) {
    console.error('获取列表失败:', error)
    showToast('获取列表失败', 'error')
  } finally {
    loading.value = false
  }
}

// 添加数据
async function add() {
  if (!value.value.trim()) return
  
  try {
    const response = await axios({
      url: 'https://e2knygut8e.bja.sealos.run/add_todo',
      method: 'POST',
      data: {
        value: value.value,
        isCompleted: false,
      },
    })
    if (response.data.code === 0) {
      showToast('添加成功', 'success')
      getList()
      value.value = ''
    } else {
      showToast('添加失败', 'error')
    }
  } catch (error) {
    console.error('添加失败:', error)
    showToast('添加失败', 'error')
  }
}

// 删除数据
async function del(id) {
  try {
    const response = await axios({
      url: 'https://e2knygut8e.bja.sealos.run/delete_todo',
      method: 'POST',
      data: { id: id },
    })
    if (response.data.code === 0) {
      showToast('删除成功', 'success')
      getList()
    } else {
      showToast('删除失败', 'error')
    }
  } catch (error) {
    console.error('删除失败:', error)
    showToast('删除失败', 'error')
  }
}

// 切换完成状态
async function toggleComplete(id, currentStatus) {
  try {
    const response = await axios({
      url: 'https://e2knygut8e.bja.sealos.run/update_todo',
      method: 'POST',
      data: { id: id },
    })
    if (response.data.code === 0) {
      showToast('状态更新成功', 'success')
      getList()
    } else {
      showToast('状态更新失败', 'error')
    }
  } catch (error) {
    console.error('状态切换失败:', error)
    showToast('状态更新失败', 'error')
  }
}

// 显示 Toast 提示
function showToast(message, type = 'success') {
  toast.value = { show: true, message, type }
  setTimeout(() => {
    toast.value.show = false
  }, 3000)
}

// 过滤列表
const filteredList = computed(() => {
  if (!searchQuery.value) return list.value
  return list.value.filter(item => 
    item.value.toLowerCase().includes(searchQuery.value.toLowerCase())
  )
})

// 页面加载时获取数据
onMounted(() => {
  getList()
})
</script>

<template>
  <div class="app-container">
    <!-- 毛玻璃导航栏 -->
    <nav class="navbar">
      <div class="nav-content">
        <h1 class="nav-title">Todo</h1>
        <div class="nav-actions">
          <button 
            @click="showSearch = !showSearch" 
            class="search-button"
            :class="{ active: showSearch }"
            title="搜索待办事项"
          >
            <MagnifyingGlassIcon class="w-5 h-5" />
            <span class="button-text">搜索</span>
          </button>
        </div>
      </div>
    </nav>

    <!-- 搜索栏 -->
    <div 
      v-if="showSearch" 
      class="search-container"
      :class="{ 'search-visible': showSearch }"
    >
      <div class="search-input-wrapper">
        <MagnifyingGlassIcon class="search-icon" />
        <input
          v-model="searchQuery"
          type="text"
          placeholder="搜索待办事项..."
          class="search-input"
        />
      </div>
    </div>

    <!-- 主要内容 -->
    <main class="main-content">
      <!-- 添加表单 -->
      <div class="add-form-container">
        <div class="add-form">
          <input
            v-model="value"
            type="text"
            placeholder="添加新的待办事项..."
            class="add-input"
            @keyup.enter="add"
          />
          <button @click="add" class="add-button" title="添加新待办事项">
            <PlusIcon class="w-5 h-5" />
            <span class="button-text">添加</span>
          </button>
        </div>
      </div>

      <!-- 操作指南 -->
      <div class="guide-container">
        <h3 class="guide-title">操作指南</h3>
        <div class="guide-items">
          <div class="guide-item">
            <div class="guide-icon add-icon">
              <PlusIcon class="w-4 h-4" />
            </div>
            <span class="guide-text">蓝色方框 - 添加新待办事项</span>
          </div>
          <div class="guide-item">
            <div class="guide-icon search-icon">
              <MagnifyingGlassIcon class="w-4 h-4" />
            </div>
            <span class="guide-text">蓝色圆点 - 搜索待办事项</span>
          </div>
          <div class="guide-item">
            <div class="guide-icon check-icon">
              <CheckCircleIcon class="w-4 h-4" />
            </div>
            <span class="guide-text">圆形图标 - 标记完成/未完成</span>
          </div>
          <div class="guide-item">
            <div class="guide-icon delete-icon">
              <TrashIcon class="w-4 h-4" />
            </div>
            <span class="guide-text">红色圆点 - 删除待办事项</span>
          </div>
        </div>
      </div>

      <!-- 列表容器 -->
      <div class="list-container">
        <!-- 骨架屏 -->
        <div v-if="loading" class="skeleton-container">
          <div 
            v-for="i in 3" 
            :key="i" 
            class="skeleton-item"
          >
            <div class="skeleton-checkbox"></div>
            <div class="skeleton-text"></div>
            <div class="skeleton-button"></div>
          </div>
        </div>

        <!-- 实际列表 -->
        <div v-else-if="filteredList.length > 0" class="todo-list">
          <div
            v-for="item in filteredList"
            :key="item._id"
            class="todo-item"
            :class="{ completed: item.isCompleted }"
          >
            <div class="todo-content">
              <button 
                @click="toggleComplete(item._id, item.isCompleted)"
                class="todo-checkbox"
                :class="{ checked: item.isCompleted }"
                :title="item.isCompleted ? '标记为未完成' : '标记为已完成'"
              >
                <CheckCircleIcon v-if="item.isCompleted" class="w-5 h-5" />
                <div v-else class="circle-icon"></div>
              </button>
              <span class="todo-text">{{ item.value }}</span>
            </div>
            <button 
              @click="del(item._id)" 
              class="delete-button"
              title="删除此待办事项"
            >
              <TrashIcon class="w-4 h-4" />
            </button>
          </div>
        </div>

        <!-- 空状态 -->
        <div v-else class="empty-state">
          <div class="empty-icon">📝</div>
          <h3 class="empty-title">暂无待办事项</h3>
          <p class="empty-description">
            {{ searchQuery ? '没有找到匹配的待办事项' : '开始添加你的第一个待办事项吧' }}
          </p>
        </div>
      </div>
    </main>

    <!-- 作者信息 -->
    <footer class="author-footer">
      <div class="author-content">
        <div class="author-info">
          <span class="author-label">开发者：</span>
          <a 
            href="https://rayscout.github.io/My_Blog/" 
            target="_blank" 
            class="author-name"
          >
            Ray Scout
          </a>
        </div>
      </div>
    </footer>

    <!-- Toast 提示 -->
    <div 
      v-if="toast.show" 
      class="toast"
      :class="toast.type"
    >
      {{ toast.message }}
    </div>
  </div>
</template>

<style scoped>
/* 苹果系统字体 */
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&display=swap');

/* 基础样式 */
* {
  box-sizing: border-box;
}

.app-container {
  min-height: 100vh;
  background: linear-gradient(135deg, #f5f5f7 0%, #ffffff 100%);
  font-family: -apple-system, BlinkMacSystemFont, 'SF Pro Display', 'SF Pro Text', 'Inter', 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
  color: #1d1d1f;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  font-feature-settings: "kern" 1;
  text-rendering: optimizeLegibility;
}

/* 毛玻璃导航栏 */
.navbar {
  position: sticky;
  top: 0;
  z-index: 100;
  background: rgba(255, 255, 255, 0.8);
  backdrop-filter: blur(20px);
  -webkit-backdrop-filter: blur(20px);
  border-bottom: 1px solid rgba(0, 0, 0, 0.1);
  transition: all 0.3s ease;
}

.nav-content {
  max-width: 800px;
  margin: 0 auto;
  padding: 1rem 2rem;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.nav-title {
  font-size: 1.5rem;
  font-weight: 600;
  color: #1d1d1f;
  margin: 0;
  font-family: -apple-system, BlinkMacSystemFont, 'SF Pro Display', 'SF Pro Text', 'Inter', 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
}

.nav-actions {
  display: flex;
  gap: 0.5rem;
}

.search-button {
  background: rgba(0, 122, 255, 0.1);
  border: none;
  border-radius: 12px;
  padding: 0.75rem 1rem;
  color: #007aff;
  cursor: pointer;
  transition: all 0.2s ease;
  display: flex;
  align-items: center;
  justify-content: center;
  font-family: -apple-system, BlinkMacSystemFont, 'SF Pro Display', 'SF Pro Text', 'Inter', 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
  gap: 0.5rem;
}

.search-button:hover {
  background: rgba(0, 122, 255, 0.2);
  transform: scale(1.05);
}

.search-button.active {
  background: #007aff;
  color: white;
}

.button-text {
  font-size: 0.875rem;
  font-weight: 500;
  white-space: nowrap;
}

/* 搜索栏 */
.search-container {
  background: rgba(255, 255, 255, 0.9);
  backdrop-filter: blur(20px);
  -webkit-backdrop-filter: blur(20px);
  border-bottom: 1px solid rgba(0, 0, 0, 0.1);
  transform: translateY(-100%);
  transition: transform 0.3s ease;
}

.search-container.search-visible {
  transform: translateY(0);
}

.search-input-wrapper {
  max-width: 800px;
  margin: 0 auto;
  padding: 1rem 2rem;
  display: flex;
  align-items: center;
  gap: 0.75rem;
}

.search-icon {
  width: 1.25rem;
  height: 1.25rem;
  color: #8e8e93;
}

.search-input {
  flex: 1;
  border: none;
  background: transparent;
  font-size: 1rem;
  color: #1d1d1f;
  outline: none;
  padding: 0.5rem 0;
  font-family: -apple-system, BlinkMacSystemFont, 'SF Pro Display', 'SF Pro Text', 'Inter', 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
}

.search-input::placeholder {
  color: #8e8e93;
}

/* 主要内容 */
.main-content {
  max-width: 800px;
  margin: 0 auto;
  padding: 2rem;
}

/* 添加表单 */
.add-form-container {
  margin-bottom: 2rem;
}

.add-form {
  background: rgba(255, 255, 255, 0.8);
  backdrop-filter: blur(20px);
  -webkit-backdrop-filter: blur(20px);
  border-radius: 16px;
  border: 1px solid rgba(0, 0, 0, 0.1);
  padding: 1.5rem;
  display: flex;
  gap: 1rem;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.05);
  transition: all 0.3s ease;
}

.add-form:hover {
  box-shadow: 0 8px 30px rgba(0, 0, 0, 0.1);
  transform: translateY(-2px);
}

.add-input {
  flex: 1;
  border: none;
  background: transparent;
  font-size: 1rem;
  color: #1d1d1f;
  outline: none;
  padding: 0.75rem;
  border-radius: 12px;
  background: rgba(0, 0, 0, 0.02);
  transition: all 0.2s ease;
  font-family: -apple-system, BlinkMacSystemFont, 'SF Pro Display', 'SF Pro Text', 'Inter', 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
}

.add-input:focus {
  background: rgba(0, 122, 255, 0.05);
}

.add-input::placeholder {
  color: #8e8e93;
}

.add-button {
  background: #007aff;
  border: none;
  border-radius: 12px;
  padding: 0.75rem;
  color: white;
  cursor: pointer;
  transition: all 0.2s ease;
  display: flex;
  align-items: center;
  justify-content: center;
  font-family: -apple-system, BlinkMacSystemFont, 'SF Pro Display', 'SF Pro Text', 'Inter', 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
  gap: 0.5rem;
}

.add-button:hover {
  background: #0056cc;
  transform: scale(1.05);
}

/* 操作指南 */
.guide-container {
  margin-bottom: 2rem;
  padding: 1.5rem;
  background: rgba(255, 255, 255, 0.8);
  backdrop-filter: blur(20px);
  -webkit-backdrop-filter: blur(20px);
  border-radius: 16px;
  border: 1px solid rgba(0, 0, 0, 0.1);
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.05);
}

.guide-title {
  font-size: 1.25rem;
  font-weight: 600;
  color: #1d1d1f;
  margin-bottom: 1rem;
  font-family: -apple-system, BlinkMacSystemFont, 'SF Pro Display', 'SF Pro Text', 'Inter', 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
}

.guide-items {
  display: flex;
  flex-wrap: wrap;
  gap: 1.5rem;
  justify-content: center;
}

.guide-item {
  display: flex;
  align-items: center;
  gap: 0.75rem;
  color: #8e8e93;
  font-size: 0.9rem;
  font-family: -apple-system, BlinkMacSystemFont, 'SF Pro Display', 'SF Pro Text', 'Inter', 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
}

.guide-icon {
  width: 2rem;
  height: 2rem;
  border-radius: 50%;
  background: rgba(0, 122, 255, 0.1);
  display: flex;
  align-items: center;
  justify-content: center;
  color: #007aff;
}

.guide-text {
  flex: 1;
}

/* 列表容器 */
.list-container {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

/* 骨架屏 */
.skeleton-container {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.skeleton-item {
  background: rgba(255, 255, 255, 0.8);
  backdrop-filter: blur(20px);
  -webkit-backdrop-filter: blur(20px);
  border-radius: 16px;
  border: 1px solid rgba(0, 0, 0, 0.1);
  padding: 1.5rem;
  display: flex;
  align-items: center;
  gap: 1rem;
  animation: pulse 2s infinite;
}

.skeleton-checkbox {
  width: 1.5rem;
  height: 1.5rem;
  border-radius: 50%;
  background: #e5e5ea;
}

.skeleton-text {
  flex: 1;
  height: 1rem;
  background: #e5e5ea;
  border-radius: 4px;
}

.skeleton-button {
  width: 2rem;
  height: 2rem;
  border-radius: 8px;
  background: #e5e5ea;
}

@keyframes pulse {
  0%, 100% { opacity: 1; }
  50% { opacity: 0.5; }
}

/* Todo 列表 */
.todo-list {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.todo-item {
  background: rgba(255, 255, 255, 0.8);
  backdrop-filter: blur(20px);
  -webkit-backdrop-filter: blur(20px);
  border-radius: 16px;
  border: 1px solid rgba(0, 0, 0, 0.1);
  padding: 1.5rem;
  display: flex;
  align-items: center;
  justify-content: space-between;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.05);
  transition: all 0.3s ease;
}

.todo-item:hover {
  box-shadow: 0 8px 30px rgba(0, 0, 0, 0.1);
  transform: translateY(-2px);
}

.todo-item.completed {
  opacity: 0.6;
}

.todo-content {
  display: flex;
  align-items: center;
  gap: 1rem;
  flex: 1;
}

.todo-checkbox {
  background: transparent;
  border: none;
  color: #8e8e93;
  cursor: pointer;
  transition: all 0.2s ease;
  display: flex;
  align-items: center;
  justify-content: center;
}

.todo-checkbox:hover {
  color: #007aff;
  transform: scale(1.1);
}

.todo-checkbox.checked {
  color: #007aff;
}

.circle-icon {
  width: 1.25rem;
  height: 1.25rem;
  border: 2px solid #8e8e93;
  border-radius: 50%;
  transition: all 0.2s ease;
}

.todo-checkbox:hover .circle-icon {
  border-color: #007aff;
}

.todo-text {
  font-size: 1rem;
  color: #1d1d1f;
  transition: all 0.2s ease;
  font-family: -apple-system, BlinkMacSystemFont, 'SF Pro Display', 'SF Pro Text', 'Inter', 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
}

.todo-item.completed .todo-text {
  text-decoration: line-through;
  color: #8e8e93;
}

.delete-button {
  background: rgba(255, 59, 48, 0.1);
  border: none;
  border-radius: 8px;
  padding: 0.5rem;
  color: #ff3b30;
  cursor: pointer;
  transition: all 0.2s ease;
  display: flex;
  align-items: center;
  justify-content: center;
}

.delete-button:hover {
  background: rgba(255, 59, 48, 0.2);
  transform: scale(1.1);
}

/* 空状态 */
.empty-state {
  text-align: center;
  padding: 4rem 2rem;
  background: rgba(255, 255, 255, 0.8);
  backdrop-filter: blur(20px);
  -webkit-backdrop-filter: blur(20px);
  border-radius: 16px;
  border: 1px solid rgba(0, 0, 0, 0.1);
}

.empty-icon {
  font-size: 3rem;
  margin-bottom: 1rem;
}

.empty-title {
  font-size: 1.5rem;
  font-weight: 600;
  color: #1d1d1f;
  margin: 0 0 0.5rem 0;
  font-family: -apple-system, BlinkMacSystemFont, 'SF Pro Display', 'SF Pro Text', 'Inter', 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
}

.empty-description {
  color: #8e8e93;
  margin: 0;
  font-family: -apple-system, BlinkMacSystemFont, 'SF Pro Display', 'SF Pro Text', 'Inter', 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
}

/* 作者信息 */
.author-footer {
  position: fixed;
  bottom: 0;
  left: 0;
  width: 100%;
  background: rgba(255, 255, 255, 0.8);
  backdrop-filter: blur(20px);
  -webkit-backdrop-filter: blur(20px);
  border-top: 1px solid rgba(0, 0, 0, 0.1);
  padding: 0.75rem 2rem;
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 2rem;
  z-index: 99;
}

.author-content {
  display: flex;
  gap: 2rem;
  align-items: center;
}

.author-info {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  color: #8e8e93;
  font-size: 0.875rem;
  font-family: -apple-system, BlinkMacSystemFont, 'SF Pro Display', 'SF Pro Text', 'Inter', 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
}

.author-name {
  color: #007aff;
  font-weight: 500;
  text-decoration: none;
  transition: color 0.2s ease;
}

.author-name:hover {
  color: #0056cc;
}

.author-label {
  color: #8e8e93;
}

/* Toast 提示 */
.toast {
  position: fixed;
  top: 2rem;
  right: 2rem;
  padding: 1rem 1.5rem;
  border-radius: 12px;
  color: white;
  font-weight: 500;
  z-index: 1000;
  animation: slideIn 0.3s ease;
  backdrop-filter: blur(20px);
  -webkit-backdrop-filter: blur(20px);
  font-family: -apple-system, BlinkMacSystemFont, 'SF Pro Display', 'SF Pro Text', 'Inter', 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
}

.toast.success {
  background: rgba(52, 199, 89, 0.9);
}

.toast.error {
  background: rgba(255, 59, 48, 0.9);
}

@keyframes slideIn {
  from {
    transform: translateX(100%);
    opacity: 0;
  }
  to {
    transform: translateX(0);
    opacity: 1;
  }
}

/* 响应式设计 */
@media (max-width: 768px) {
  .nav-content {
    padding: 1rem;
  }
  
  .main-content {
    padding: 1rem;
  }
  
  .add-form {
    padding: 1rem;
  }
  
  .todo-item {
    padding: 1rem;
  }
  
  .toast {
    right: 1rem;
    left: 1rem;
  }

  .author-footer {
    flex-direction: column;
    gap: 0.5rem;
    padding: 0.75rem 1rem;
  }

  .author-content {
    flex-direction: column;
    gap: 0.5rem;
  }
}
</style>
