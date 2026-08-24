<template>
  <div class="language-switcher">
    <button
      class="language-button"
      @click="toggleDropdown"
      @blur="handleBlur"
    >
      <svg
        width="20"
        height="20"
        viewBox="0 0 20 20"
        fill="none"
        class="globe-icon"
      >
        <circle cx="10" cy="10" r="7.5" stroke="currentColor" stroke-width="1.5"/>
        <path d="M3 10H17" stroke="currentColor" stroke-width="1.5"/>
        <path d="M10 3C10 3 7.5 5.5 7.5 10C7.5 14.5 10 17 10 17" stroke="currentColor" stroke-width="1.5"/>
        <path d="M10 3C10 3 12.5 5.5 12.5 10C12.5 14.5 10 17 10 17" stroke="currentColor" stroke-width="1.5"/>
      </svg>
      <span class="language-label">{{ localeName }}</span>
      <svg
        class="chevron"
        :class="{ 'chevron-open': isDropdownOpen }"
        width="16"
        height="16"
        viewBox="0 0 16 16"
        fill="none"
      >
        <path d="M4 6L8 10L12 6" stroke="currentColor" stroke-width="2" stroke-linecap="round"/>
      </svg>
    </button>

    <div v-if="isDropdownOpen" class="dropdown-menu">
      <button
        v-for="locale in availableLocales"
        :key="locale"
        class="dropdown-item"
        :class="{ active: currentLocale === locale }"
        @mousedown.prevent="selectLanguage(locale)"
      >
        <span class="language-name">{{ getLanguageName(locale) }}</span>
        <svg
          v-if="currentLocale === locale"
          width="18"
          height="18"
          viewBox="0 0 18 18"
          fill="none"
          class="check-icon"
        >
          <path d="M4 9L7.5 12.5L14 6" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
        </svg>
      </button>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import { useI18n } from '../composables/useI18n'

const { currentLocale, setLocale, availableLocales, localeName } = useI18n()

const isDropdownOpen = ref(false)

const languageNames = {
  en: 'English',
  ja: '日本語'
}

const getLanguageName = (locale) => {
  return languageNames[locale] || locale
}

const toggleDropdown = () => {
  isDropdownOpen.value = !isDropdownOpen.value
}

const handleBlur = () => {
  // Delay to allow mousedown events on dropdown items to fire first
  setTimeout(() => {
    isDropdownOpen.value = false
  }, 200)
}

const selectLanguage = (locale) => {
  setLocale(locale)
  isDropdownOpen.value = false
}
</script>

<style scoped>
.language-switcher {
  position: relative;
}

.language-button {
  display: flex;
  align-items: center;
  gap: var(--space-3);
  width: 100%;
  padding: var(--space-3);
  background: transparent;
  border: none;
  border-radius: var(--radius-md);
  cursor: pointer;
  transition: var(--transition);
  font-family: inherit;
  font-size: 0.875rem;
  font-weight: 500;
  color: var(--text-muted);
}

.language-button:hover {
  background: var(--surface-hover);
  color: var(--text-primary);
}

.language-button:focus-visible {
  outline: none;
  box-shadow: var(--ring);
}

.globe-icon {
  flex-shrink: 0;
}

.language-label {
  font-weight: 500;
  white-space: nowrap;
}

.chevron {
  margin-left: auto;
  transition: transform 0.2s ease;
  flex-shrink: 0;
}

.chevron-open {
  transform: rotate(180deg);
}

/* Opens upward — the switcher now sits at the bottom of the sidebar */
.dropdown-menu {
  position: absolute;
  bottom: calc(100% + var(--space-2));
  left: 0;
  min-width: 180px;
  background: var(--surface);
  border: 1px solid var(--border);
  border-radius: var(--radius-lg);
  box-shadow: var(--shadow-lg);
  z-index: 1000;
  overflow: hidden;
}

.dropdown-item {
  width: 100%;
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: var(--space-3);
  padding: var(--space-3) var(--space-4);
  background: none;
  border: none;
  text-align: left;
  cursor: pointer;
  transition: background 0.15s ease;
  font-family: inherit;
  font-size: 0.875rem;
  font-weight: 500;
  color: var(--text-body);
}

.dropdown-item:hover {
  background: var(--surface-hover);
}

.dropdown-item.active {
  background: var(--accent-soft);
  color: var(--accent);
}

.language-name {
  flex: 1;
}

.check-icon {
  color: var(--accent);
  flex-shrink: 0;
}

/* Icons-only mode. The `collapsed` class lives on the shell, outside this
   component's scope, so these selectors must be fully global — a bare
   :global(.x) prefix on a descendant selector compiles incorrectly. */
:global(.sidebar.collapsed .language-button) {
  justify-content: center;
}

:global(.sidebar.collapsed .language-label),
:global(.sidebar.collapsed .chevron) {
  display: none;
}

:global(.sidebar.collapsed .dropdown-menu) {
  left: calc(100% + var(--space-2));
  bottom: 0;
}

@media (max-width: 1024px) {
  :global(.sidebar .language-button) {
    justify-content: center;
  }

  :global(.sidebar .language-label),
  :global(.sidebar .chevron) {
    display: none;
  }

  :global(.sidebar .dropdown-menu) {
    left: calc(100% + var(--space-2));
    bottom: 0;
  }
}

@media (max-width: 768px) {
  /* Mobile drawer is full width — restore the label */
  :global(.sidebar .language-button) {
    justify-content: flex-start;
  }

  :global(.sidebar .language-label),
  :global(.sidebar .chevron) {
    display: block;
  }

  :global(.sidebar .dropdown-menu) {
    left: 0;
    bottom: calc(100% + var(--space-2));
  }
}
</style>
