<template>
  <div class="app" :class="{ 'sidebar-collapsed': sidebarCollapsed }">
    <aside class="sidebar">
      <div class="sidebar-header">
        <div class="sidebar-logo-full">
          <h1 class="sidebar-brand">{{ t('nav.companyName') }}</h1>
          <span class="sidebar-subtitle">{{ t('nav.subtitle') }}</span>
        </div>
        <div class="sidebar-logo-icon" aria-hidden="true">CC</div>
      </div>

      <nav class="sidebar-nav">
        <router-link class="sidebar-item" to="/" :class="{ active: $route.path === '/' }" data-tooltip="Overview">
          <svg class="nav-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true">
            <path d="M3 3h8v8H3V3zm10 0h8v8h-8V3zM3 13h8v8H3v-8zm10 0h8v8h-8v-8z"/>
          </svg>
          <span class="nav-label">{{ t('nav.overview') }}</span>
        </router-link>

        <router-link class="sidebar-item" to="/inventory" :class="{ active: $route.path === '/inventory' }" data-tooltip="Inventory">
          <svg class="nav-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true">
            <path d="M20 7l-8-4-8 4m16 0l-8 4m8-4v10l-8 4m0-10L4 7m8 4v10M4 7v10l8 4"/>
          </svg>
          <span class="nav-label">{{ t('nav.inventory') }}</span>
        </router-link>

        <router-link class="sidebar-item" to="/orders" :class="{ active: $route.path === '/orders' }" data-tooltip="Orders">
          <svg class="nav-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true">
            <path d="M9 5H7a2 2 0 00-2 2v12a2 2 0 002 2h10a2 2 0 002-2V7a2 2 0 00-2-2h-2M9 5a2 2 0 002 2h2a2 2 0 002-2M9 5a2 2 0 012-2h2a2 2 0 012 2m-3 7h3m-3 4h3m-6-4h.01M9 16h.01"/>
          </svg>
          <span class="nav-label">{{ t('nav.orders') }}</span>
        </router-link>

        <router-link class="sidebar-item" to="/spending" :class="{ active: $route.path === '/spending' }" data-tooltip="Finance">
          <svg class="nav-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true">
            <path d="M9 19v-6a2 2 0 00-2-2H5a2 2 0 00-2 2v6a2 2 0 002 2h2a2 2 0 002-2zm0 0V9a2 2 0 012-2h2a2 2 0 012 2v10m-6 0a2 2 0 002 2h2a2 2 0 002-2m0 0V5a2 2 0 012-2h2a2 2 0 012 2v14a2 2 0 01-2 2h-2a2 2 0 01-2-2z"/>
          </svg>
          <span class="nav-label">{{ t('nav.finance') }}</span>
        </router-link>

        <router-link class="sidebar-item" to="/demand" :class="{ active: $route.path === '/demand' }" data-tooltip="Demand Forecast">
          <svg class="nav-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true">
            <path d="M2.25 18L9 11.25l4.306 4.307a11.95 11.95 0 015.814-5.519l2.74-1.22m0 0l-5.94-2.28m5.94 2.28l-2.28 5.941"/>
          </svg>
          <span class="nav-label">{{ t('nav.demandForecast') }}</span>
        </router-link>

        <router-link class="sidebar-item" to="/reports" :class="{ active: $route.path === '/reports' }" data-tooltip="Reports">
          <svg class="nav-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true">
            <path d="M9 17v-2m3 2v-4m3 4v-6m2 10H7a2 2 0 01-2-2V5a2 2 0 012-2h5.586a1 1 0 01.707.293l5.414 5.414a1 1 0 01.293.707V19a2 2 0 01-2 2z"/>
          </svg>
          <span class="nav-label">Reports</span>
        </router-link>

        <button class="sidebar-toggle" @click="toggleSidebar" :title="sidebarCollapsed ? 'Expand sidebar' : 'Collapse sidebar'">
          <svg class="nav-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true">
            <path v-if="!sidebarCollapsed" d="M15 19l-7-7 7-7"/>
            <path v-else d="M9 5l7 7-7 7"/>
          </svg>
          <span class="nav-label">Collapse</span>
        </button>
      </nav>

      <div class="sidebar-footer">
        <LanguageSwitcher />
        <ProfileMenu
          @show-profile-details="showProfileDetails = true"
          @show-tasks="showTasks = true"
        />
      </div>
    </aside>

    <div class="content-wrapper">
      <FilterBar />
      <main class="main-content">
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
import { ref, onMounted, onUnmounted, computed } from 'vue'
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
    const { currentUser } = useAuth()
    const { t } = useI18n()
    const showProfileDetails = ref(false)
    const showTasks = ref(false)
    const apiTasks = ref([])

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

    // Sidebar collapse state — persisted in localStorage
    const sidebarCollapsed = ref(false)

    const COLLAPSE_BREAKPOINT = 768

    const toggleSidebar = () => {
      sidebarCollapsed.value = !sidebarCollapsed.value
      localStorage.setItem('sidebar-collapsed', String(sidebarCollapsed.value))
    }

    // Auto-collapse on small screens; restore saved preference on larger screens
    const handleResize = () => {
      if (window.innerWidth < COLLAPSE_BREAKPOINT) {
        sidebarCollapsed.value = true
      }
    }

    onMounted(() => {
      loadTasks()
      // Restore saved preference, then apply screen-size override
      const saved = localStorage.getItem('sidebar-collapsed')
      sidebarCollapsed.value = saved === 'true' || window.innerWidth < COLLAPSE_BREAKPOINT
      window.addEventListener('resize', handleResize)
    })

    onUnmounted(() => {
      window.removeEventListener('resize', handleResize)
    })

    return {
      t,
      showProfileDetails,
      showTasks,
      tasks,
      addTask,
      deleteTask,
      toggleTask,
      sidebarCollapsed,
      toggleSidebar
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

:root {
  --color-sidebar-bg:          #1e293b;
  --color-sidebar-border:      #334155;
  --color-sidebar-text:        #94a3b8;
  --color-sidebar-text-active: #f1f5f9;
  --color-sidebar-hover:       #334155;
  --color-sidebar-active:      #1e3a5f;
  --color-accent:              #3b82f6;

  --color-content-bg:          #f8fafc;
  --color-content-border:      #e2e8f0;

  --color-surface:             #ffffff;
  --color-surface-raised:      #f1f5f9;

  --color-text-primary:        #0f172a;
  --color-text-secondary:      #64748b;
  --color-text-muted:          #94a3b8;

  --color-success:  #10b981;
  --color-warning:  #f59e0b;
  --color-danger:   #ef4444;
  --color-info:     #3b82f6;
}

body {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, sans-serif;
  background: var(--color-content-bg);
  color: var(--color-text-primary);
  font-size: 14px;
  line-height: 1.5;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

/* ── App shell ── */
.app {
  display: flex;
  min-height: 100vh;
}

/* ── Sidebar ── */
.sidebar {
  width: 220px;
  flex-shrink: 0;
  background: var(--color-sidebar-bg);
  border-right: 1px solid var(--color-sidebar-border);
  display: flex;
  flex-direction: column;
  position: fixed;
  top: 0;
  left: 0;
  bottom: 0;
  z-index: 100;
  overflow: hidden;
  transition: width 0.2s ease;
}

.sidebar-header {
  padding: 20px 16px 16px;
  border-bottom: 1px solid var(--color-sidebar-border);
}

.sidebar-brand {
  font-size: 14px;
  font-weight: 700;
  color: var(--color-sidebar-text-active);
  letter-spacing: -0.2px;
  margin: 0;
}

.sidebar-subtitle {
  font-size: 11px;
  color: var(--color-sidebar-text);
  display: block;
  margin-top: 4px;
}

.sidebar-nav {
  flex: 1;
  padding: 8px 12px;
  display: flex;
  flex-direction: column;
  gap: 2px;
}

.sidebar-item {
  display: flex;
  align-items: center;
  height: 36px;
  padding: 0 12px;
  border-radius: 6px;
  font-size: 13px;
  font-weight: 500;
  color: var(--color-sidebar-text);
  text-decoration: none;
  transition: background 0.12s, color 0.12s;
  gap: 10px;
}

.sidebar-item:hover {
  background: var(--color-sidebar-hover);
  color: var(--color-sidebar-text-active);
}

.sidebar-item.active {
  background: var(--color-sidebar-active);
  color: var(--color-sidebar-text-active);
  font-weight: 600;
  border-left: 2px solid var(--color-accent);
  padding-left: 10px;
  gap: 8px;
}

.sidebar-footer {
  padding: 12px;
  border-top: 1px solid var(--color-sidebar-border);
  display: flex;
  flex-direction: column;
  gap: 8px;
}

/* ── Content area ── */
.content-wrapper {
  margin-left: 220px;
  flex: 1;
  min-width: 0;
  display: flex;
  flex-direction: column;
  min-height: 100vh;
  transition: margin-left 0.2s ease;
}

.main-content {
  flex: 1;
  padding: 24px;
}

/* ── Page headers ── */
.page-header {
  margin-bottom: 24px;
}

.page-header h2 {
  font-size: 22px;
  font-weight: 700;
  color: var(--color-text-primary);
  margin-bottom: 4px;
  letter-spacing: -0.3px;
}

.page-header p {
  color: var(--color-text-secondary);
  font-size: 14px;
}

/* ── Stats grid ── */
.stats-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(240px, 1fr));
  gap: 16px;
  margin-bottom: 24px;
}

.stat-card {
  background: var(--color-surface);
  padding: 20px;
  border-radius: 8px;
  border: 1px solid var(--color-content-border);
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.06);
  transition: box-shadow 0.2s ease;
}

.stat-card:hover {
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.08);
}

.stat-label {
  color: var(--color-text-secondary);
  font-size: 12px;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.5px;
  margin-bottom: 8px;
}

.stat-value {
  font-size: 28px;
  font-weight: 700;
  color: var(--color-text-primary);
  letter-spacing: -0.5px;
  line-height: 1.2;
}

.stat-card.warning .stat-value { color: #ea580c; }
.stat-card.success .stat-value { color: #059669; }
.stat-card.danger  .stat-value { color: #dc2626; }
.stat-card.info    .stat-value { color: #2563eb; }

/* ── Cards ── */
.card {
  background: var(--color-surface);
  border-radius: 8px;
  padding: 20px;
  border: 1px solid var(--color-content-border);
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.06);
  margin-bottom: 20px;
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 16px;
  padding-bottom: 12px;
  border-bottom: 1px solid var(--color-content-border);
}

.card-title {
  font-size: 15px;
  font-weight: 700;
  color: var(--color-text-primary);
  letter-spacing: -0.2px;
}

/* ── Tables ── */
.table-container {
  overflow-x: auto;
}

table {
  width: 100%;
  border-collapse: collapse;
}

thead {
  background: var(--color-surface-raised);
  border-top: 1px solid var(--color-content-border);
  border-bottom: 1px solid var(--color-content-border);
}

th {
  text-align: left;
  padding: 8px 12px;
  font-weight: 600;
  color: var(--color-text-secondary);
  font-size: 12px;
  text-transform: uppercase;
  letter-spacing: 0.05em;
}

td {
  padding: 8px 12px;
  border-top: 1px solid var(--color-surface-raised);
  color: #334155;
  font-size: 13px;
}

tbody tr {
  transition: background-color 0.15s ease;
}

tbody tr:hover {
  background: var(--color-surface-raised);
}

/* ── Badges ── */
.badge {
  display: inline-block;
  padding: 3px 10px;
  border-radius: 6px;
  font-size: 11px;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.025em;
}

.badge.success    { background: #d1fae5; color: #065f46; }
.badge.warning    { background: #fed7aa; color: #92400e; }
.badge.danger     { background: #fecaca; color: #991b1b; }
.badge.info       { background: #dbeafe; color: #1e40af; }
.badge.increasing { background: #d1fae5; color: #065f46; }
.badge.decreasing { background: #fecaca; color: #991b1b; }
.badge.stable     { background: #e0e7ff; color: #3730a3; }
.badge.high       { background: #fecaca; color: #991b1b; }
.badge.medium     { background: #fed7aa; color: #92400e; }
.badge.low        { background: #dbeafe; color: #1e40af; }

/* ── Utility ── */
.loading {
  text-align: center;
  padding: 48px;
  color: var(--color-text-secondary);
  font-size: 14px;
}

.error {
  background: #fef2f2;
  border: 1px solid #fecaca;
  color: #991b1b;
  padding: 16px;
  border-radius: 8px;
  margin: 16px 0;
  font-size: 14px;
}

/* Nav icon + label layout */
.nav-icon {
  width: 18px;
  height: 18px;
  flex-shrink: 0;
}

/* Toggle button matches nav item style */
.sidebar-toggle {
  display: flex;
  align-items: center;
  height: 36px;
  padding: 0 12px;
  gap: 10px;
  border-radius: 6px;
  font-size: 13px;
  font-weight: 500;
  color: var(--color-sidebar-text);
  background: transparent;
  border: none;
  cursor: pointer;
  width: 100%;
  transition: background 0.12s, color 0.12s;
  margin-top: 4px;
}

.sidebar-toggle:hover {
  background: var(--color-sidebar-hover);
  color: var(--color-sidebar-text-active);
}

/* Compact logo (hidden by default, shown when collapsed) */
.sidebar-logo-icon {
  display: none;
  font-size: 13px;
  font-weight: 800;
  color: var(--color-sidebar-text-active);
  letter-spacing: -0.5px;
}

/* ── Collapsed sidebar ── */
.app.sidebar-collapsed .sidebar {
  width: 64px;
  overflow: visible;
}

.app.sidebar-collapsed .sidebar-header {
  display: none;
}

.app.sidebar-collapsed .sidebar-nav {
  padding: 8px 8px;
  align-items: center;
}

.app.sidebar-collapsed .sidebar-item {
  padding: 0;
  justify-content: center;
  width: 40px;
  height: 40px;
  border-radius: 8px;
  position: relative;
}

.app.sidebar-collapsed .sidebar-item.active {
  border-left: 2px solid var(--color-accent);
  padding-left: 0;
  border-radius: 0 8px 8px 0;
}

.app.sidebar-collapsed .sidebar-item .nav-label {
  display: none;
}

.app.sidebar-collapsed .sidebar-toggle {
  padding: 0;
  justify-content: center;
  width: 40px;
  height: 40px;
}

.app.sidebar-collapsed .sidebar-toggle .nav-label {
  display: none;
}

.app.sidebar-collapsed .sidebar-footer {
  padding: 12px 8px;
  align-items: center;
}

.app.sidebar-collapsed .content-wrapper {
  margin-left: 64px;
}

/* CSS tooltips on hover in collapsed mode */
.app.sidebar-collapsed .sidebar-item[data-tooltip] {
  position: relative;
}

.app.sidebar-collapsed .sidebar-item[data-tooltip]:hover::after {
  content: attr(data-tooltip);
  position: absolute;
  left: calc(100% + 10px);
  top: 50%;
  transform: translateY(-50%);
  background: #0f172a;
  color: #f1f5f9;
  padding: 5px 10px;
  border-radius: 6px;
  font-size: 12px;
  font-weight: 500;
  white-space: nowrap;
  z-index: 9999;
  pointer-events: none;
  box-shadow: 0 4px 12px rgba(0,0,0,0.2);
}

/* Arrow on tooltip */
.app.sidebar-collapsed .sidebar-item[data-tooltip]:hover::before {
  content: '';
  position: absolute;
  left: calc(100% + 4px);
  top: 50%;
  transform: translateY(-50%);
  border: 5px solid transparent;
  border-right-color: #0f172a;
  z-index: 9999;
  pointer-events: none;
}

/* Collapsed mode: icon-only footer buttons */
.app.sidebar-collapsed .profile-name,
.app.sidebar-collapsed .profile-button .chevron,
.app.sidebar-collapsed .language-label,
.app.sidebar-collapsed .language-button .chevron {
  display: none;
}

.app.sidebar-collapsed .profile-button,
.app.sidebar-collapsed .language-button {
  padding: 0;
  background: transparent;
  border: 1px solid transparent;
  justify-content: center;
  width: 40px;
  height: 40px;
}

.app.sidebar-collapsed .profile-button:hover,
.app.sidebar-collapsed .language-button:hover {
  background: var(--color-sidebar-hover);
  border-color: transparent;
}

.app.sidebar-collapsed .sidebar-footer .globe-icon {
  color: var(--color-sidebar-text);
}

/* Open all sidebar footer dropdowns upward */
.app .sidebar-footer .dropdown-menu {
  top: auto;
  bottom: calc(100% + 8px);
}

/* Adjust horizontal alignment in collapsed mode */
.app.sidebar-collapsed .sidebar-footer .dropdown-menu {
  right: auto;
  left: 0;
}
</style>
