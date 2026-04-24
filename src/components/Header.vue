<template>
    <el-card class="header-card" :style="cardStyle">
        <div class="header-inner">
            <h1 class="top-title" :style="titleStyle">Github Release Stats</h1>
            <button class="vp-theme-toggle" :class="{ 'is-dark': theme === 'dark' }" :aria-label="themeText"
                :title="themeText" @click="$emit('toggle-theme')">
                <!-- 太阳图标 (浅色模式显示) -->
                <svg class="vp-icon sun-icon" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24">
                    <circle cx="12" cy="12" r="5" />
                    <line x1="12" y1="1" x2="12" y2="3" />
                    <line x1="12" y1="21" x2="12" y2="23" />
                    <line x1="4.22" y1="4.22" x2="5.64" y2="5.64" />
                    <line x1="18.36" y1="18.36" x2="19.78" y2="19.78" />
                    <line x1="1" y1="12" x2="3" y2="12" />
                    <line x1="21" y1="12" x2="23" y2="12" />
                    <line x1="4.22" y1="19.78" x2="5.64" y2="18.36" />
                    <line x1="18.36" y1="5.64" x2="19.78" y2="4.22" />
                </svg>
                <!-- 月亮图标 (深色模式显示) -->
                <svg class="vp-icon moon-icon" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24">
                    <path d="M21 12.79A9 9 0 1 1 11.21 3 7 7 0 0 0 21 12.79z" />
                </svg>
            </button>
        </div>
    </el-card>
</template>

<script>
export default {
    props: {
        theme: {
            type: String,
            default: "light",
        },
    },
    computed: {
        cardStyle() {
            return {
                backgroundColor: this.theme === "dark" ? "#1a2438" : "#ffffff",
                borderColor: this.theme === "dark" ? "#2f3a52" : "#e5e7eb",
                color: this.theme === "dark" ? "#e5e7eb" : "#1f2937",
            };
        },
        titleStyle() {
            return {
                color: this.theme === "dark" ? "#e5e7eb" : "#1f2937",
            };
        },
        themeText() {
            return this.theme === "dark" ? "切换浅色模式" : "切换深色模式";
        },
    },
};
</script>

<style>
.header-card {
    border-radius: 0 !important;
    box-shadow: 0 10px 22px var(--shadow-color);
    transition: background-color 0.25s ease, border-color 0.25s ease;
}

.header-card .el-card__body {
    background-color: transparent;
}

.header-inner {
    display: flex;
    align-items: center;
    justify-content: space-between;
    gap: 12px;
}

.top-title {
    margin: 8px 0;
    font-size: 24px;
    line-height: 1.2;
    transition: color 0.25s ease;
}

/* VitePress 风格切换按钮 */
.vp-theme-toggle {
    display: flex;
    align-items: center;
    justify-content: center;
    width: 36px;
    height: 36px;
    border-radius: 50%;
    border: none;
    background: transparent;
    cursor: pointer;
    color: var(--text-primary);
    padding: 0;
    outline: none;
    transition: color 0.25s, background-color 0.25s;
    flex-shrink: 0;
}

.vp-theme-toggle:hover {
    background-color: rgba(125, 125, 125, 0.12);
}

.vp-icon {
    width: 20px;
    height: 20px;
    fill: none;
    stroke: currentColor;
    stroke-width: 2;
    stroke-linecap: round;
    stroke-linejoin: round;
    transition: opacity 0.25s ease, transform 0.25s ease;
}

/* 浅色模式：显示太阳，隐藏月亮 */
.vp-theme-toggle .sun-icon {
    opacity: 1;
    transform: rotate(0deg) scale(1);
    display: block;
}

.vp-theme-toggle .moon-icon {
    opacity: 0;
    transform: rotate(-90deg) scale(0.5);
    display: none;
}

/* 深色模式：隐藏太阳，显示月亮 */
.vp-theme-toggle.is-dark .sun-icon {
    opacity: 0;
    transform: rotate(90deg) scale(0.5);
    display: none;
}

.vp-theme-toggle.is-dark .moon-icon {
    opacity: 1;
    transform: rotate(0deg) scale(1);
    display: block;
    fill: currentColor;
    stroke: none;
}

@media (max-width: 768px) {
    .top-title {
        font-size: 18px;
    }
}
</style>
