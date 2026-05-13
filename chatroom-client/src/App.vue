<template>
  <router-view />
</template>

<script setup>
</script>

<style>
:root {
  --apple-black: #1d1d1f;
  --apple-blue: #0071e3;
  --apple-blue-hover: #0077ed;
  --apple-gray: #86868b;
  --apple-gray-light: #f5f5f7;
  --apple-gray-ultra-light: #fbfbfd;
  --apple-text-primary: #1d1d1f;
  --apple-text-secondary: #86868b;
  --apple-white: #ffffff;
  
  --apple-success: #34c759;
  --apple-warning: #ffd60a;
  --apple-error: #ff3b30;
  
  --apple-shadow-sm: 0 1px 3px rgba(0, 0, 0, 0.08);
  --apple-shadow-md: 0 4px 12px rgba(0, 0, 0, 0.1);
  --apple-shadow-lg: 0 8px 24px rgba(0, 0, 0, 0.12);
  --apple-shadow-xl: 0 16px 48px rgba(0, 0, 0, 0.15);
  
  --apple-radius-sm: 6px;
  --apple-radius-md: 12px;
  --apple-radius-lg: 18px;
  --apple-radius-xl: 24px;
  --apple-radius-full: 9999px;
  
  --apple-transition-fast: 0.2s ease-out;
  --apple-transition-normal: 0.3s ease-out;
  --apple-transition-slow: 0.4s ease-out;
  
  --glass-blur: 20px;
  --glass-opacity: 0.72;
}

* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

html, body, #app {
  height: 100%;
  font-family: -apple-system, BlinkMacSystemFont, 'SF Pro Display', 'SF Pro Text', 'Helvetica Neue', Arial, sans-serif;
  background-color: var(--apple-gray-ultra-light);
  color: var(--apple-text-primary);
  scroll-behavior: smooth;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

::-webkit-scrollbar {
  width: 8px;
  height: 8px;
}

::-webkit-scrollbar-track {
  background: transparent;
}

::-webkit-scrollbar-thumb {
  background: rgba(0, 0, 0, 0.2);
  border-radius: var(--apple-radius-full);
}

::-webkit-scrollbar-thumb:hover {
  background: rgba(0, 0, 0, 0.3);
}

.el-button--primary {
  background: var(--apple-blue);
  border: none;
  border-radius: var(--apple-radius-md);
  color: var(--apple-white);
  font-weight: 400;
  letter-spacing: -0.01em;
  box-shadow: none;
  transition: all var(--apple-transition-fast);
}

.el-button--primary:hover {
  background: var(--apple-blue-hover);
  transform: none;
  box-shadow: none;
}

.el-button--primary:active {
  transform: scale(0.98);
}

.el-input__wrapper {
  border-radius: var(--apple-radius-sm);
  border-color: rgba(0, 0, 0, 0.1);
  background: rgba(0, 0, 0, 0.04);
  box-shadow: none;
  transition: all var(--apple-transition-fast);
}

.el-input__wrapper:hover {
  background: rgba(0, 0, 0, 0.06);
  box-shadow: none;
}

.el-input__wrapper.is-focus {
  background: var(--apple-white);
  box-shadow: 0 0 0 4px rgba(0, 113, 227, 0.15);
}

.el-card {
  border-radius: var(--apple-radius-lg);
  box-shadow: var(--apple-shadow-md);
  border: none;
  background: var(--apple-white);
}

.el-dialog {
  border-radius: var(--apple-radius-xl);
  box-shadow: var(--apple-shadow-xl);
  background: var(--apple-white);
  border: none;
}

.el-dialog__header {
  background: transparent;
  border-bottom: none;
  padding: 24px 24px 0;
}

.el-dialog__body {
  padding: 24px;
}

.el-tabs__nav-wrap::after {
  display: none;
}

.el-tabs__item {
  color: var(--apple-text-secondary);
  transition: all var(--apple-transition-fast);
  font-weight: 400;
}

.el-tabs__item:hover {
  color: var(--apple-text-primary);
}

.el-tabs__item.is-active {
  color: var(--apple-text-primary);
  font-weight: 600;
}

.el-tabs__active-bar {
  background: var(--apple-black);
  height: 2px;
  border-radius: var(--apple-radius-full);
}

.el-avatar {
  font-weight: 500;
  background: linear-gradient(135deg, var(--apple-blue), #5856d6);
}

.el-badge__content {
  background: var(--apple-error);
  border: 2px solid #fff;
}

.el-dropdown-menu__item {
  transition: all var(--apple-transition-fast);
}

.el-dropdown-menu__item:hover {
  background: var(--apple-gray-light);
  color: var(--apple-text-primary);
}

.glass-effect {
  background: rgba(255, 255, 255, var(--glass-opacity));
  backdrop-filter: blur(var(--glass-blur));
  -webkit-backdrop-filter: blur(var(--glass-blur));
  border: 1px solid rgba(255, 255, 255, 0.5);
}

.apple-gradient {
  background: linear-gradient(180deg, var(--apple-gray-ultra-light) 0%, var(--apple-gray-light) 100%);
}
</style>
