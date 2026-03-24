<template>
  <div class="app-layout">
    <aside class="sidebar" :class="{ collapsed: sidebarCollapsed }">
      <div class="sidebar-header">
        <div class="logo-full" v-if="!sidebarCollapsed">
          <span class="logo-name">{{ t('nav.companyName') }}</span>
          <span class="logo-subtitle">{{ t('nav.subtitle') }}</span>
        </div>
        <div class="logo-collapsed" v-else>
          <span class="logo-initial">{{ t('nav.companyName').charAt(0) }}</span>
        </div>
      </div>

      <nav class="sidebar-nav">
        <router-link to="/" class="sidebar-link" :class="{ active: $route.path === '/' }">
          <svg viewBox="0 0 20 20" fill="none" xmlns="http://www.w3.org/2000/svg">
            <path d="M3 3h7v7H3V3zm11 0h4v4h-4V3zm0 7h4v7h-4v-7zM3 13h7v4H3v-4z" fill="currentColor"/>
          </svg>
          <span class="link-label">{{ t('nav.overview') }}</span>
        </router-link>

        <router-link to="/inventory" class="sidebar-link" :class="{ active: $route.path === '/inventory' }">
          <svg viewBox="0 0 20 20" fill="none" xmlns="http://www.w3.org/2000/svg">
            <path d="M2 4l8-2 8 2v12l-8 2-8-2V4z" stroke="currentColor" stroke-width="1.5" fill="none"/>
            <path d="M10 2v16M2 4l8 2 8-2" stroke="currentColor" stroke-width="1.5" fill="none"/>
          </svg>
          <span class="link-label">{{ t('nav.inventory') }}</span>
        </router-link>

        <router-link to="/orders" class="sidebar-link" :class="{ active: $route.path === '/orders' }">
          <svg viewBox="0 0 20 20" fill="none" xmlns="http://www.w3.org/2000/svg">
            <path d="M6 2h8v2H6V2zM4 4h12v14H4V4z" stroke="currentColor" stroke-width="1.5" fill="none"/>
            <path d="M7 9h6M7 12h4" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/>
          </svg>
          <span class="link-label">{{ t('nav.orders') }}</span>
        </router-link>

        <router-link to="/spending" class="sidebar-link" :class="{ active: $route.path === '/spending' }">
          <svg viewBox="0 0 20 20" fill="none" xmlns="http://www.w3.org/2000/svg">
            <path d="M10 2v16M6 8l4-4 4 4M4 18h12" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round" fill="none"/>
          </svg>
          <span class="link-label">{{ t('nav.finance') }}</span>
        </router-link>

        <router-link to="/demand" class="sidebar-link" :class="{ active: $route.path === '/demand' }">
          <svg viewBox="0 0 20 20" fill="none" xmlns="http://www.w3.org/2000/svg">
            <path d="M3 17l5-5 3 3 6-8" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round" fill="none"/>
            <path d="M14 7h3v3" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round" fill="none"/>
          </svg>
          <span class="link-label">{{ t('nav.demandForecast') }}</span>
        </router-link>

        <router-link to="/reports" class="sidebar-link" :class="{ active: $route.path === '/reports' }">
          <svg viewBox="0 0 20 20" fill="none" xmlns="http://www.w3.org/2000/svg">
            <path d="M4 16V10M8 16V6M12 16V8M16 16V4" stroke="currentColor" stroke-width="2" stroke-linecap="round"/>
          </svg>
          <span class="link-label">Reports</span>
        </router-link>

        <router-link to="/restocking" class="sidebar-link" :class="{ active: $route.path === '/restocking' }">
          <svg viewBox="0 0 20 20" fill="none" xmlns="http://www.w3.org/2000/svg">
            <path d="M3 10a7 7 0 0113-3.5M17 10a7 7 0 01-13 3.5" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" fill="none"/>
            <path d="M16 3v4h-4M4 17v-4h4" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round" fill="none"/>
          </svg>
          <span class="link-label">{{ t('restocking.navLabel') }}</span>
        </router-link>
      </nav>

      <div class="sidebar-footer">
        <div class="footer-language">
          <LanguageSwitcher />
        </div>
        <div class="footer-user">
          <div class="user-avatar">{{ getInitials(currentUser.name) }}</div>
          <div class="user-info" v-if="!sidebarCollapsed">
            <span class="user-name">{{ currentUser.name }}</span>
            <span class="user-role">{{ currentUser.jobTitle }}</span>
          </div>
        </div>
      </div>

      <button class="sidebar-toggle" @click="toggleSidebar" :title="sidebarCollapsed ? 'Expand sidebar' : 'Collapse sidebar'">
        <svg v-if="!sidebarCollapsed" viewBox="0 0 20 20" fill="none" xmlns="http://www.w3.org/2000/svg" width="18" height="18">
          <path d="M13 15l-5-5 5-5" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
        </svg>
        <svg v-else viewBox="0 0 20 20" fill="none" xmlns="http://www.w3.org/2000/svg" width="18" height="18">
          <path d="M7 5l5 5-5 5" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
        </svg>
      </button>
    </aside>

    <div class="main-panel" :class="{ 'sidebar-collapsed': sidebarCollapsed }">
      <header class="topbar">
        <div class="topbar-left">
          <nav class="breadcrumb">
            <span class="breadcrumb-home">Home</span>
            <span class="breadcrumb-sep">/</span>
            <span class="breadcrumb-current">{{ currentPageTitle }}</span>
          </nav>
        </div>
        <div class="topbar-right">
          <FilterBar />
          <ProfileMenu
            @show-profile-details="showProfileDetails = true"
            @show-tasks="showTasks = true"
          />
        </div>
      </header>

      <main class="page-content">
        <router-view />
      </main>
    </div>

    <ProfileDetailsModal
      :is-open="showProfileDetails"
      @close="showProfileDetails = false"
    />

    <TasksModal
      :is-open="showTasks"
      :tasks="tasks"
      @close="showTasks = false"
      @add-task="addTask"
      @delete-task="deleteTask"
      @toggle-task="toggleTask"
    />
  </div>
</template>

<script>
import { ref, onMounted, computed } from 'vue'
import { useRoute } from 'vue-router'
import { api } from './api'
import { useAuth } from './composables/useAuth'
import { useI18n } from './composables/useI18n'
import FilterBar from './components/FilterBar.vue'
import ProfileMenu from './components/ProfileMenu.vue'
import ProfileDetailsModal from './components/ProfileDetailsModal.vue'
import TasksModal from './components/TasksModal.vue'
import LanguageSwitcher from './components/LanguageSwitcher.vue'

export default {
  name: 'App',
  components: {
    FilterBar,
    ProfileMenu,
    ProfileDetailsModal,
    TasksModal,
    LanguageSwitcher
  },
  setup() {
    const { currentUser, getInitials } = useAuth()
    const { t } = useI18n()
    const route = useRoute()
    const showProfileDetails = ref(false)
    const showTasks = ref(false)
    const apiTasks = ref([])
    const sidebarCollapsed = ref(false)

    const toggleSidebar = () => {
      sidebarCollapsed.value = !sidebarCollapsed.value
    }

    const pageTitleMap = {
      '/': 'Overview',
      '/inventory': 'Inventory',
      '/orders': 'Orders',
      '/spending': 'Finance',
      '/demand': 'Demand Forecast',
      '/reports': 'Reports',
      '/restocking': 'Restocking'
    }

    const currentPageTitle = computed(() => {
      return pageTitleMap[route.path] || 'Overview'
    })

    // Merge mock tasks from currentUser with API tasks
    const tasks = computed(() => {
      return [...currentUser.value.tasks, ...apiTasks.value]
    })

    const loadTasks = async () => {
      try {
        apiTasks.value = await api.getTasks()
      } catch (err) {
        console.error('Failed to load tasks:', err)
      }
    }

    const addTask = async (taskData) => {
      try {
        const newTask = await api.createTask(taskData)
        // Add new task to the beginning of the array
        apiTasks.value.unshift(newTask)
      } catch (err) {
        console.error('Failed to add task:', err)
      }
    }

    const deleteTask = async (taskId) => {
      try {
        // Check if it's a mock task (from currentUser)
        const isMockTask = currentUser.value.tasks.some(t => t.id === taskId)

        if (isMockTask) {
          // Remove from mock tasks
          const index = currentUser.value.tasks.findIndex(t => t.id === taskId)
          if (index !== -1) {
            currentUser.value.tasks.splice(index, 1)
          }
        } else {
          // Remove from API tasks
          await api.deleteTask(taskId)
          apiTasks.value = apiTasks.value.filter(t => t.id !== taskId)
        }
      } catch (err) {
        console.error('Failed to delete task:', err)
      }
    }

    const toggleTask = async (taskId) => {
      try {
        // Check if it's a mock task (from currentUser)
        const mockTask = currentUser.value.tasks.find(t => t.id === taskId)

        if (mockTask) {
          // Toggle mock task status
          mockTask.status = mockTask.status === 'pending' ? 'completed' : 'pending'
        } else {
          // Toggle API task
          const updatedTask = await api.toggleTask(taskId)
          const index = apiTasks.value.findIndex(t => t.id === taskId)
          if (index !== -1) {
            apiTasks.value[index] = updatedTask
          }
        }
      } catch (err) {
        console.error('Failed to toggle task:', err)
      }
    }

    onMounted(loadTasks)

    return {
      t,
      currentUser,
      getInitials,
      sidebarCollapsed,
      toggleSidebar,
      currentPageTitle,
      showProfileDetails,
      showTasks,
      tasks,
      addTask,
      deleteTask,
      toggleTask
    }
  }
}
</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, sans-serif;
  background: #f8fafc;
  color: #1e293b;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

/* ─── Layout ─────────────────────────────────────── */

.app-layout {
  display: flex;
  min-height: 100vh;
}

/* ─── Sidebar ─────────────────────────────────────── */

.sidebar {
  --sidebar-width: 260px;
  width: var(--sidebar-width);
  min-height: 100vh;
  background: #0f172a;
  display: flex;
  flex-direction: column;
  position: fixed;
  top: 0;
  left: 0;
  z-index: 200;
  transition: width 0.2s ease;
  overflow: hidden;
}

.sidebar.collapsed {
  --sidebar-width: 72px;
  width: 72px;
}

.sidebar-header {
  padding: 1.25rem 1rem;
  border-bottom: 1px solid rgba(255, 255, 255, 0.08);
  min-height: 72px;
  display: flex;
  align-items: center;
}

.logo-full {
  display: flex;
  flex-direction: column;
  gap: 0.25rem;
  overflow: hidden;
  white-space: nowrap;
}

.logo-name {
  font-size: 1rem;
  font-weight: 700;
  color: #ffffff;
  letter-spacing: -0.025em;
}

.logo-subtitle {
  font-size: 0.75rem;
  color: #64748b;
  font-weight: 400;
}

.logo-collapsed {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 100%;
}

.logo-initial {
  font-size: 1.25rem;
  font-weight: 700;
  color: #ffffff;
  background: #3b82f6;
  width: 36px;
  height: 36px;
  border-radius: 8px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.sidebar-nav {
  flex: 1;
  padding: 0.75rem 0;
  overflow-y: auto;
  overflow-x: hidden;
}

.sidebar-link {
  display: flex;
  align-items: center;
  gap: 0.75rem;
  padding: 0.625rem 1rem;
  margin: 0.125rem 0.75rem;
  border-radius: 8px;
  color: #94a3b8;
  text-decoration: none;
  font-size: 0.875rem;
  font-weight: 500;
  transition: all 0.15s ease;
  position: relative;
  overflow: hidden;
  white-space: nowrap;
}

.sidebar-link:hover {
  color: #e2e8f0;
  background: rgba(255, 255, 255, 0.05);
}

.sidebar-link.active {
  color: #ffffff;
  background: #1e293b;
  border-left: 3px solid #3b82f6;
  margin-left: calc(0.75rem - 3px);
}

.sidebar-link svg {
  width: 20px;
  height: 20px;
  flex-shrink: 0;
}

.sidebar-link .link-label {
  opacity: 1;
  transition: opacity 0.15s ease;
}

.sidebar.collapsed .sidebar-link .link-label {
  opacity: 0;
  width: 0;
  overflow: hidden;
}

.sidebar.collapsed .sidebar-link {
  justify-content: center;
  padding: 0.625rem;
  margin: 0.125rem 0.75rem;
}

.sidebar.collapsed .sidebar-link.active {
  margin-left: calc(0.75rem - 3px);
}

/* ─── Sidebar Footer ──────────────────────────────── */

.sidebar-footer {
  border-top: 1px solid rgba(255, 255, 255, 0.1);
  padding: 0.75rem;
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.footer-language {
  display: flex;
  justify-content: center;
}

.sidebar.collapsed .footer-language {
  overflow: hidden;
}

.footer-user {
  display: flex;
  align-items: center;
  gap: 0.75rem;
  padding: 0.5rem 0.25rem;
  overflow: hidden;
}

.user-avatar {
  width: 34px;
  height: 34px;
  border-radius: 50%;
  background: #3b82f6;
  color: #ffffff;
  font-size: 0.75rem;
  font-weight: 700;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
}

.user-info {
  display: flex;
  flex-direction: column;
  gap: 0.125rem;
  overflow: hidden;
  white-space: nowrap;
}

.user-name {
  font-size: 0.813rem;
  font-weight: 600;
  color: #e2e8f0;
}

.user-role {
  font-size: 0.75rem;
  color: #64748b;
}

/* ─── Sidebar Toggle ──────────────────────────────── */

.sidebar-toggle {
  border-top: 1px solid rgba(255, 255, 255, 0.1);
  padding: 0.75rem;
  display: flex;
  align-items: center;
  justify-content: center;
  background: none;
  border-left: none;
  border-right: none;
  border-bottom: none;
  cursor: pointer;
  color: #64748b;
  transition: color 0.15s ease;
  width: 100%;
}

.sidebar-toggle:hover {
  color: #e2e8f0;
}

/* ─── Main Panel ──────────────────────────────────── */

.main-panel {
  margin-left: 260px;
  flex: 1;
  display: flex;
  flex-direction: column;
  min-height: 100vh;
  transition: margin-left 0.2s ease;
}

.main-panel.sidebar-collapsed {
  margin-left: 72px;
}

/* ─── Topbar ──────────────────────────────────────── */

.topbar {
  background: #ffffff;
  border-bottom: 1px solid #e2e8f0;
  height: 60px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 1.5rem;
  position: sticky;
  top: 0;
  z-index: 100;
}

.topbar-left {
  display: flex;
  align-items: center;
}

.breadcrumb {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  font-size: 0.875rem;
}

.breadcrumb-home {
  color: #94a3b8;
  font-weight: 400;
}

.breadcrumb-sep {
  color: #cbd5e1;
}

.breadcrumb-current {
  color: #0f172a;
  font-weight: 600;
}

.topbar-right {
  display: flex;
  align-items: center;
  gap: 0.75rem;
}

/* ─── Page Content ────────────────────────────────── */

.page-content {
  flex: 1;
  padding: 1.5rem;
  background: #f1f5f9;
  overflow-y: auto;
}

/* ─── Page Header ─────────────────────────────────── */

.page-header {
  margin-bottom: 1.5rem;
}

.page-header h2 {
  font-size: 1.875rem;
  font-weight: 700;
  color: #0f172a;
  margin-bottom: 0.375rem;
  letter-spacing: -0.025em;
}

.page-header p {
  color: #64748b;
  font-size: 0.938rem;
}

/* ─── Stats Grid ──────────────────────────────────── */

.stats-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
  gap: 1rem;
  margin-bottom: 1.5rem;
}

.stat-card {
  background: white;
  padding: 1.5rem 1.25rem;
  border-radius: 10px;
  border: 1px solid #e2e8f0;
  transition: all 0.2s ease;
}

.stat-card:hover {
  border-color: #cbd5e1;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.06);
}

.stat-label {
  color: #64748b;
  font-size: 0.875rem;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.5px;
  margin-bottom: 0.625rem;
}

.stat-value {
  font-size: 2.25rem;
  font-weight: 700;
  color: #0f172a;
  letter-spacing: -0.025em;
}

.stat-card.warning .stat-value {
  color: #ea580c;
}

.stat-card.success .stat-value {
  color: #059669;
}

.stat-card.danger .stat-value {
  color: #dc2626;
}

.stat-card.info .stat-value {
  color: #2563eb;
}

/* ─── Card ────────────────────────────────────────── */

.card {
  background: white;
  border-radius: 12px;
  padding: 1.25rem;
  border: 1px solid #e2e8f0;
  margin-bottom: 1.25rem;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.04), 0 1px 2px rgba(0, 0, 0, 0.06);
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1rem;
  padding-bottom: 0.875rem;
  border-bottom: 1px solid #e2e8f0;
}

.card-title {
  font-size: 1.125rem;
  font-weight: 700;
  color: #0f172a;
  letter-spacing: -0.025em;
}

/* ─── Table ───────────────────────────────────────── */

.table-container {
  overflow-x: auto;
}

table {
  width: 100%;
  border-collapse: collapse;
}

thead {
  background: #f8fafc;
  border-top: 1px solid #e2e8f0;
  border-bottom: 1px solid #e2e8f0;
}

th {
  text-align: left;
  padding: 0.5rem 0.75rem;
  font-weight: 600;
  color: #475569;
  font-size: 0.75rem;
  text-transform: uppercase;
  letter-spacing: 0.05em;
}

td {
  padding: 0.5rem 0.75rem;
  border-top: 1px solid #f1f5f9;
  color: #334155;
  font-size: 0.875rem;
}

tbody tr {
  transition: background-color 0.15s ease;
}

tbody tr:hover {
  background: #f8fafc;
}

/* ─── Badge ───────────────────────────────────────── */

.badge {
  display: inline-block;
  padding: 0.313rem 0.75rem;
  border-radius: 6px;
  font-size: 0.75rem;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.025em;
}

.badge.success {
  background: #d1fae5;
  color: #065f46;
}

.badge.warning {
  background: #fed7aa;
  color: #92400e;
}

.badge.danger {
  background: #fecaca;
  color: #991b1b;
}

.badge.info {
  background: #dbeafe;
  color: #1e40af;
}

.badge.increasing {
  background: #d1fae5;
  color: #065f46;
}

.badge.decreasing {
  background: #fecaca;
  color: #991b1b;
}

.badge.stable {
  background: #e0e7ff;
  color: #3730a3;
}

.badge.high {
  background: #fecaca;
  color: #991b1b;
}

.badge.medium {
  background: #fed7aa;
  color: #92400e;
}

.badge.low {
  background: #dbeafe;
  color: #1e40af;
}

/* ─── States ──────────────────────────────────────── */

.loading {
  text-align: center;
  padding: 3rem;
  color: #64748b;
  font-size: 0.938rem;
}

.error {
  background: #fef2f2;
  border: 1px solid #fecaca;
  color: #991b1b;
  padding: 1rem;
  border-radius: 8px;
  margin: 1rem 0;
  font-size: 0.938rem;
}
</style>
