<template>
  <div class="app" :class="{ 'sidebar-collapsed': isCollapsed }">
    <aside class="sidebar" :class="{ collapsed: isCollapsed, open: isMobileOpen }">
      <div class="sidebar-brand">
        <span class="brand-mark">C</span>
        <div class="brand-text">
          <h1>{{ t('nav.companyName') }}</h1>
          <span class="brand-subtitle">{{ t('nav.subtitle') }}</span>
        </div>
      </div>

      <nav class="sidebar-nav">
        <router-link
          v-for="item in navItems"
          :key="item.path"
          :to="item.path"
          :class="{ active: $route.path === item.path }"
          :title="isCollapsed ? t(item.labelKey) : null"
          @click="isMobileOpen = false"
        >
          <span class="nav-icon" v-html="item.icon"></span>
          <span class="nav-label">{{ t(item.labelKey) }}</span>
        </router-link>
      </nav>

      <div class="sidebar-footer">
        <LanguageSwitcher />
        <ProfileMenu
          @show-profile-details="showProfileDetails = true"
          @show-tasks="showTasks = true"
        />
        <button
          class="collapse-toggle"
          :title="isCollapsed ? t('nav.expandSidebar') : t('nav.collapseSidebar')"
          @click="isCollapsed = !isCollapsed"
        >
          <svg width="18" height="18" viewBox="0 0 20 20" fill="none">
            <path
              :d="isCollapsed ? 'M7 4L13 10L7 16' : 'M13 4L7 10L13 16'"
              stroke="currentColor"
              stroke-width="1.75"
              stroke-linecap="round"
              stroke-linejoin="round"
            />
          </svg>
          <span class="nav-label">
            {{ isCollapsed ? t('nav.expandSidebar') : t('nav.collapseSidebar') }}
          </span>
        </button>
      </div>
    </aside>

    <div
      v-if="isMobileOpen"
      class="sidebar-overlay"
      @click="isMobileOpen = false"
    ></div>

    <div class="app-main">
      <button class="mobile-menu-btn" @click="isMobileOpen = true">
        <svg width="20" height="20" viewBox="0 0 20 20" fill="none">
          <path d="M3 5H17M3 10H17M3 15H17" stroke="currentColor" stroke-width="1.75" stroke-linecap="round"/>
        </svg>
      </button>
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
import { ref, onMounted, computed, watch } from 'vue'
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

    // Sidebar state. Collapsed preference persists; mobile drawer does not.
    const isCollapsed = ref(localStorage.getItem('sidebar-collapsed') === 'true')
    const isMobileOpen = ref(false)

    watch(isCollapsed, (value) => {
      localStorage.setItem('sidebar-collapsed', String(value))
    })

    // Nav definition. Icons are inline 20x20 stroke SVGs matching the style
    // already used by LanguageSwitcher and FilterBar.
    const navItems = [
      {
        path: '/',
        labelKey: 'nav.overview',
        icon: '<svg width="20" height="20" viewBox="0 0 20 20" fill="none"><path d="M3 10.5L10 4L17 10.5V16a1 1 0 0 1-1 1h-3.5v-4.5h-5V17H4a1 1 0 0 1-1-1v-5.5Z" stroke="currentColor" stroke-width="1.5" stroke-linejoin="round"/></svg>'
      },
      {
        path: '/inventory',
        labelKey: 'nav.inventory',
        icon: '<svg width="20" height="20" viewBox="0 0 20 20" fill="none"><path d="M3 6.5 10 3l7 3.5v7L10 17l-7-3.5v-7Z" stroke="currentColor" stroke-width="1.5" stroke-linejoin="round"/><path d="M3 6.5 10 10m0 0 7-3.5M10 10v7" stroke="currentColor" stroke-width="1.5" stroke-linejoin="round"/></svg>'
      },
      {
        path: '/orders',
        labelKey: 'nav.orders',
        icon: '<svg width="20" height="20" viewBox="0 0 20 20" fill="none"><path d="M5 3h10a1 1 0 0 1 1 1v13l-3-2-3 2-3-2-3 2V4a1 1 0 0 1 1-1Z" stroke="currentColor" stroke-width="1.5" stroke-linejoin="round"/><path d="M7.5 7.5h5M7.5 10.5h5" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/></svg>'
      },
      {
        path: '/spending',
        labelKey: 'nav.finance',
        icon: '<svg width="20" height="20" viewBox="0 0 20 20" fill="none"><circle cx="10" cy="10" r="7" stroke="currentColor" stroke-width="1.5"/><path d="M12 7.5c-.5-.6-1.2-1-2-1-1.2 0-2 .7-2 1.6 0 2 4 1.2 4 3.2 0 .9-.8 1.7-2 1.7-.8 0-1.5-.4-2-1M10 5.5v9" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/></svg>'
      },
      {
        path: '/demand',
        labelKey: 'nav.demandForecast',
        icon: '<svg width="20" height="20" viewBox="0 0 20 20" fill="none"><path d="M3 14l4-4 3 3 6-6" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/><path d="M12.5 7H16v3.5" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/></svg>'
      },
      {
        path: '/reports',
        labelKey: 'nav.reports',
        icon: '<svg width="20" height="20" viewBox="0 0 20 20" fill="none"><rect x="3.5" y="3" width="13" height="14" rx="1.5" stroke="currentColor" stroke-width="1.5"/><path d="M7 12.5v-2M10 12.5v-4M13 12.5v-6" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/></svg>'
      }
    ]

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
      showProfileDetails,
      showTasks,
      tasks,
      addTask,
      deleteTask,
      toggleTask,
      navItems,
      isCollapsed,
      isMobileOpen
    }
  }
}
</script>

<style>
:root {
  /* Spacing — 4px base */
  --space-1: 4px;
  --space-2: 8px;
  --space-3: 12px;
  --space-4: 16px;
  --space-5: 20px;
  --space-6: 24px;
  --space-8: 32px;
  --space-10: 40px;
  --space-12: 48px;

  /* Radius */
  --radius-sm: 6px;
  --radius-md: 8px;
  --radius-lg: 12px;
  --radius-full: 9999px;

  /* Elevation */
  --shadow-sm: 0 1px 2px rgba(15, 23, 42, 0.04);
  --shadow-md: 0 2px 8px rgba(15, 23, 42, 0.06);
  --shadow-lg: 0 8px 24px rgba(15, 23, 42, 0.10);

  /* Surfaces */
  --surface: #ffffff;
  --surface-sunken: #f8fafc;
  --surface-hover: #f1f5f9;

  /* Borders */
  --border: #e2e8f0;
  --border-strong: #cbd5e1;

  /* Text */
  --text-primary: #0f172a;
  --text-body: #334155;
  --text-muted: #64748b;
  --text-subtle: #94a3b8;
  --text-inverse: #ffffff;

  /* Accent */
  --accent: #2563eb;
  --accent-bright: #3b82f6;
  --accent-hover: #1d4ed8;
  --accent-soft: #eff6ff;

  /* Status */
  --success: #059669;
  --success-soft: #ecfdf5;
  --warning: #d97706;
  --warning-soft: #fffbeb;
  --danger: #dc2626;
  --danger-soft: #fef2f2;
  --danger-border: #fecaca;
  --info: #2563eb;
  --info-soft: #eff6ff;

  /* Layout */
  --sidebar-width: 248px;
  --sidebar-width-collapsed: 68px;
  --content-max: 1440px;

  /* Motion + focus */
  --transition: all 0.15s ease;
  --ring: 0 0 0 3px rgba(37, 99, 235, 0.12);
}

* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, sans-serif;
  background: var(--surface-sunken);
  color: var(--text-body);
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

/* The sidebar is position: fixed, so it is out of flow. The content column is
   offset with margin-left rather than a grid track — reserving a grid column
   for an out-of-flow element pushes the content past the viewport edge. */
.app {
  min-height: 100vh;
}

/* ---- Sidebar ---- */

.sidebar {
  position: fixed;
  top: 0;
  left: 0;
  bottom: 0;
  width: var(--sidebar-width);
  display: flex;
  flex-direction: column;
  background: var(--surface);
  border-right: 1px solid var(--border);
  z-index: 100;
  transition: width 0.2s ease, transform 0.2s ease;
}

.sidebar.collapsed {
  width: var(--sidebar-width-collapsed);
}

.sidebar-brand {
  display: flex;
  align-items: center;
  gap: var(--space-3);
  padding: var(--space-5) var(--space-4);
  border-bottom: 1px solid var(--border);
  min-height: 72px;
  overflow: hidden;
}

.brand-mark {
  display: grid;
  place-items: center;
  width: 34px;
  height: 34px;
  flex-shrink: 0;
  border-radius: var(--radius-md);
  background: var(--accent);
  color: var(--text-inverse);
  font-size: 1rem;
  font-weight: 700;
}

.brand-text {
  display: flex;
  flex-direction: column;
  min-width: 0;
  transition: opacity 0.15s ease;
}

.brand-text h1 {
  font-size: 0.938rem;
  font-weight: 700;
  color: var(--text-primary);
  letter-spacing: -0.01em;
  white-space: nowrap;
}

.brand-subtitle {
  font-size: 0.75rem;
  color: var(--text-muted);
  white-space: nowrap;
}

.sidebar-nav {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: var(--space-1);
  padding: var(--space-4) var(--space-3);
  overflow-y: auto;
  overflow-x: hidden;
}

.sidebar-nav a {
  display: flex;
  align-items: center;
  gap: var(--space-3);
  padding: var(--space-3);
  border-radius: var(--radius-md);
  color: var(--text-muted);
  text-decoration: none;
  font-size: 0.875rem;
  font-weight: 500;
  white-space: nowrap;
  position: relative;
  transition: var(--transition);
}

.sidebar-nav a:hover {
  background: var(--surface-hover);
  color: var(--text-primary);
}

.sidebar-nav a.active {
  background: var(--accent-soft);
  color: var(--accent);
  font-weight: 600;
}

/* Left accent bar — the vertical analogue of a tab underline */
.sidebar-nav a.active::before {
  content: '';
  position: absolute;
  left: 0;
  top: 50%;
  transform: translateY(-50%);
  width: 3px;
  height: 60%;
  border-radius: 0 var(--radius-sm) var(--radius-sm) 0;
  background: var(--accent);
}

.nav-icon {
  display: grid;
  place-items: center;
  width: 20px;
  height: 20px;
  flex-shrink: 0;
}

.nav-label {
  transition: opacity 0.15s ease;
}

.sidebar-footer {
  margin-top: auto;
  display: flex;
  flex-direction: column;
  gap: var(--space-2);
  padding: var(--space-3);
  border-top: 1px solid var(--border);
  overflow: hidden;
}

.collapse-toggle {
  display: flex;
  align-items: center;
  gap: var(--space-3);
  padding: var(--space-3);
  border: none;
  border-radius: var(--radius-md);
  background: transparent;
  color: var(--text-muted);
  font-family: inherit;
  font-size: 0.875rem;
  font-weight: 500;
  white-space: nowrap;
  cursor: pointer;
  transition: var(--transition);
}

.collapse-toggle:hover {
  background: var(--surface-hover);
  color: var(--text-primary);
}

.collapse-toggle svg {
  flex-shrink: 0;
}

/* Collapsed: hide labels but keep them in the DOM for screen readers */
.sidebar.collapsed .nav-label,
.sidebar.collapsed .brand-text {
  opacity: 0;
  width: 0;
  overflow: hidden;
  pointer-events: none;
}

.sidebar.collapsed .sidebar-nav a,
.sidebar.collapsed .collapse-toggle {
  justify-content: center;
}

.sidebar.collapsed .sidebar-brand {
  justify-content: center;
  padding: var(--space-5) 0;
}

.sidebar-overlay {
  position: fixed;
  inset: 0;
  background: rgba(15, 23, 42, 0.4);
  z-index: 99;
}

/* ---- Content column ---- */

.app-main {
  display: flex;
  flex-direction: column;
  min-height: 100vh;
  margin-left: var(--sidebar-width);
  min-width: 0; /* lets wide tables scroll instead of widening the page */
  transition: margin-left 0.2s ease;
}

.app.sidebar-collapsed .app-main {
  margin-left: var(--sidebar-width-collapsed);
}

.mobile-menu-btn {
  display: none;
  align-items: center;
  justify-content: center;
  width: 40px;
  height: 40px;
  margin: var(--space-3) 0 0 var(--space-4);
  border: 1px solid var(--border);
  border-radius: var(--radius-md);
  background: var(--surface);
  color: var(--text-body);
  cursor: pointer;
  transition: var(--transition);
}

.mobile-menu-btn:hover {
  background: var(--surface-hover);
  border-color: var(--border-strong);
}

.main-content {
  flex: 1;
  max-width: var(--content-max);
  width: 100%;
  margin: 0 auto;
  padding: var(--space-6) var(--space-8);
}

/* ---- Responsive ---- */

@media (max-width: 1024px) {
  .app-main,
  .app.sidebar-collapsed .app-main {
    margin-left: var(--sidebar-width-collapsed);
  }

  .sidebar {
    width: var(--sidebar-width-collapsed);
  }

  .sidebar .nav-label,
  .sidebar .brand-text {
    opacity: 0;
    width: 0;
    overflow: hidden;
    pointer-events: none;
  }

  .sidebar .sidebar-nav a,
  .sidebar .collapse-toggle {
    justify-content: center;
  }

  .sidebar .sidebar-brand {
    justify-content: center;
    padding: var(--space-5) 0;
  }

  .collapse-toggle {
    display: none;
  }
}

@media (max-width: 768px) {
  /* Off-canvas: the drawer overlays content, so no offset at all */
  .app-main,
  .app.sidebar-collapsed .app-main {
    margin-left: 0;
  }

  .sidebar,
  .sidebar.collapsed {
    width: var(--sidebar-width);
    transform: translateX(-100%);
    box-shadow: var(--shadow-lg);
  }

  .sidebar.open {
    transform: translateX(0);
  }

  /* Restore labels in the mobile drawer */
  .sidebar .nav-label,
  .sidebar .brand-text {
    opacity: 1;
    width: auto;
    pointer-events: auto;
  }

  .sidebar .sidebar-nav a {
    justify-content: flex-start;
  }

  .sidebar .sidebar-brand {
    justify-content: flex-start;
    padding: var(--space-5) var(--space-4);
  }

  .mobile-menu-btn {
    display: flex;
  }

  .main-content {
    padding: var(--space-4);
  }
}

.page-header {
  margin-bottom: var(--space-6);
}

.page-header h2 {
  font-size: 1.875rem;
  font-weight: 700;
  color: var(--text-primary);
  margin-bottom: var(--space-1);
  letter-spacing: -0.02em;
}

.page-header p {
  color: var(--text-muted);
  font-size: 0.938rem;
}

.stats-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(240px, 1fr));
  gap: var(--space-4);
  margin-bottom: var(--space-6);
}

.stat-card {
  position: relative;
  background: var(--surface);
  padding: var(--space-5);
  border-radius: var(--radius-lg);
  border: 1px solid var(--border);
  box-shadow: var(--shadow-sm);
  overflow: hidden;
  transition: var(--transition);
}

.stat-card:hover {
  border-color: var(--border-strong);
  box-shadow: var(--shadow-md);
}

/* Status accent as a top rule rather than a thick left border */
.stat-card.success::before,
.stat-card.warning::before,
.stat-card.danger::before,
.stat-card.info::before {
  content: '';
  position: absolute;
  inset: 0 0 auto 0;
  height: 3px;
}

.stat-card.success::before { background: var(--success); }
.stat-card.warning::before { background: var(--warning); }
.stat-card.danger::before { background: var(--danger); }
.stat-card.info::before { background: var(--info); }

.stat-label {
  color: var(--text-muted);
  font-size: 0.75rem;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.05em;
  margin-bottom: var(--space-2);
}

.stat-value {
  font-size: 1.875rem;
  font-weight: 700;
  color: var(--text-primary);
  letter-spacing: -0.02em;
}

.stat-card.warning .stat-value {
  color: var(--warning);
}

.stat-card.success .stat-value {
  color: var(--success);
}

.stat-card.danger .stat-value {
  color: var(--danger);
}

.stat-card.info .stat-value {
  color: var(--info);
}

.card {
  background: var(--surface);
  border-radius: var(--radius-lg);
  padding: var(--space-5);
  border: 1px solid var(--border);
  box-shadow: var(--shadow-sm);
  margin-bottom: var(--space-6);
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: var(--space-4);
  margin-bottom: var(--space-4);
  padding-bottom: var(--space-3);
  border-bottom: 1px solid var(--border);
}

.card-title {
  font-size: 1.125rem;
  font-weight: 700;
  color: var(--text-primary);
  letter-spacing: -0.01em;
}

.btn-primary,
.btn-secondary {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: var(--space-2);
  padding: var(--space-3) var(--space-5);
  border-radius: var(--radius-md);
  border: 1px solid transparent;
  font-family: inherit;
  font-size: 0.875rem;
  font-weight: 600;
  cursor: pointer;
  transition: var(--transition);
}

.btn-primary {
  background: var(--accent);
  color: var(--text-inverse);
}

.btn-primary:hover:not(:disabled) {
  background: var(--accent-hover);
}

.btn-secondary {
  background: var(--surface);
  border-color: var(--border);
  color: var(--text-body);
}

.btn-secondary:hover:not(:disabled) {
  background: var(--surface-hover);
  border-color: var(--border-strong);
}

.btn-primary:disabled,
.btn-secondary:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.btn-primary:focus-visible,
.btn-secondary:focus-visible {
  outline: none;
  box-shadow: var(--ring);
}

.table-container {
  overflow-x: auto;
}

table {
  width: 100%;
  border-collapse: collapse;
}

thead {
  background: var(--surface-sunken);
  border-top: 1px solid var(--border);
  border-bottom: 1px solid var(--border);
}

th {
  text-align: left;
  padding: var(--space-3);
  font-weight: 600;
  color: var(--text-muted);
  font-size: 0.75rem;
  text-transform: uppercase;
  letter-spacing: 0.05em;
  white-space: nowrap;
}

td {
  padding: var(--space-3);
  border-top: 1px solid var(--surface-hover);
  color: var(--text-body);
  font-size: 0.875rem;
}

tbody tr {
  transition: background-color 0.1s ease;
}

tbody tr:hover {
  background: var(--surface-sunken);
}

.badge {
  display: inline-flex;
  align-items: center;
  gap: var(--space-1);
  padding: var(--space-1) var(--space-2);
  border-radius: var(--radius-sm);
  font-size: 0.75rem;
  font-weight: 600;
  line-height: 1.4;
  letter-spacing: 0.02em;
  white-space: nowrap;
}

.badge.success {
  background: var(--success-soft);
  color: var(--success);
}

.badge.warning {
  background: var(--warning-soft);
  color: var(--warning);
}

.badge.danger {
  background: var(--danger-soft);
  color: var(--danger);
}

.badge.info {
  background: var(--info-soft);
  color: var(--info);
}

.badge.increasing {
  background: var(--success-soft);
  color: var(--success);
}

.badge.decreasing {
  background: var(--danger-soft);
  color: var(--danger);
}

.badge.stable {
  background: var(--accent-soft);
  color: var(--accent);
}

.badge.high {
  background: var(--danger-soft);
  color: var(--danger);
}

.badge.medium {
  background: var(--warning-soft);
  color: var(--warning);
}

.badge.low {
  background: var(--info-soft);
  color: var(--info);
}

.loading {
  text-align: center;
  padding: var(--space-12);
  color: var(--text-muted);
  font-size: 0.938rem;
}

.error {
  background: var(--danger-soft);
  border: 1px solid var(--danger-border);
  color: var(--danger);
  padding: var(--space-4);
  border-radius: var(--radius-md);
  margin: var(--space-4) 0;
  font-size: 0.938rem;
}
</style>
