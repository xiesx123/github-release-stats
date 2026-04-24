<template>
  <div id="app" :data-theme="theme">
    <el-container>
      <el-header class="container-top" height="90px">
        <Header :theme="theme" @toggle-theme="toggleTheme"></Header>
      </el-header>

      <el-main class="container-main">
        <Content :theme="theme"></Content>
      </el-main>
    </el-container>
  </div>
</template>

<script>
import Content from "./components/Content.vue";
import Header from "./components/Header.vue";

export default {
  name: "app",
  data() {
    return {
      theme: "light",
    };
  },
  components: {
    Content,
    Header,
  },
  methods: {
    getThemeFromURL() {
      const theme = new URL(window.location.href).searchParams.get("theme");

      if (theme === "dark" || theme === "light") {
        return theme;
      }

      return "";
    },
    setTheme(theme) {
      this.theme = theme === "dark" ? "dark" : "light";
      localStorage.setItem("ghrs-theme", this.theme);
      document.documentElement.setAttribute("data-theme", this.theme);
      document.body.setAttribute("data-theme", this.theme);
    },
    toggleTheme() {
      this.setTheme(this.theme === "dark" ? "light" : "dark");
    },
    initTheme() {
      const urlTheme = this.getThemeFromURL();
      const storedTheme = localStorage.getItem("ghrs-theme");
      const prefersDark =
        window.matchMedia && window.matchMedia("(prefers-color-scheme: dark)").matches;

      this.setTheme(urlTheme || storedTheme || (prefersDark ? "dark" : "light"));
    },
  },
  created() {
    this.initTheme();
  },
};
</script>

<style>
:root {
  --bg-primary: #f4f7fb;
  --bg-gradient-start: #f4f7fb;
  --bg-gradient-end: #eef3ff;
  --bg-card: #ffffff;
  --text-primary: #1f2937;
  --text-secondary: #6b7280;
  --border-color: #e5e7eb;
  --link-color: #1d4ed8;
  --shadow-color: rgba(15, 23, 42, 0.08);
  --bg-hover: rgba(0, 0, 0, 0.04);
}

:root[data-theme="dark"],
body[data-theme="dark"],
#app[data-theme="dark"] {
  --bg-primary: #0b1220;
  --bg-gradient-start: #0b1220;
  --bg-gradient-end: #111a2f;
  --bg-card: #1a2438;
  --text-primary: #e5e7eb;
  --text-secondary: #9ca3af;
  --border-color: #2f3a52;
  --link-color: #93c5fd;
  --shadow-color: rgba(0, 0, 0, 0.35);
  --bg-hover: rgba(255, 255, 255, 0.06);
}

body {
  margin: 0;
  background: linear-gradient(135deg,
      var(--bg-gradient-start) 0%,
      var(--bg-gradient-end) 100%);
  color: var(--text-primary);
  transition: background 0.25s ease, color 0.25s ease;
}

#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: var(--text-primary);
  min-height: 100vh;
}

#app .container-top {
  padding: 0;
}

#app .container-main {
  padding: 12px 16px 20px;
}

a {
  color: var(--link-color);
}
</style>
