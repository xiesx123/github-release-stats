# GitHub Release Stats

一个用于统计 GitHub Release 下载量的可视化工具，支持图表展示、文件过滤、日夜间主题切换以及 URL 参数传递。

## 功能特性

- 输入 GitHub 仓库地址，自动拉取所有 Release 的下载数据
- 以柱状图 / 饼图可视化各版本下载量分布
- 支持按文件名模式过滤不需要统计的资产文件
- 支持日间 / 夜间主题切换
- 支持通过 URL 参数预填表单，方便分享和嵌入

## URL 参数

| 参数      | 说明                                 | 示例                                  |
| --------- | ------------------------------------ | ------------------------------------- |
| `repo`    | 仓库地址                             | `?repo=https://github.com/owner/repo` |
| `pattern` | 排除文件名模式（支持多个，逗号分隔） | `?pattern=*.txt,*.yml`                |
| `auto`    | 是否自动开始分析（`1` / `0`）        | `?auto=1`                             |
| `theme`   | 主题（`light` / `dark`）             | `?theme=dark`                         |

**示例：**

```
https://your-site/?repo=https://github.com/owner/repo&auto=1&theme=dark
```

## 本地开发

```bash
# 安装依赖
npm install

# 启动开发服务器
npm run serve

# 构建生产包
npm run build:legacy
```

## 技术栈

- [Vue 2](https://v2.vuejs.org/)
- [Element UI](https://element.eleme.io/)
- [v-charts](https://v-charts.js.org/)
- [parse-github-url](https://github.com/jonschlinkert/parse-github-url)

## License

MIT
