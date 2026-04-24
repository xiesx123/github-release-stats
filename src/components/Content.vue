<template>
  <div class="content-root">
    <el-container class="container">
      <el-header height="auto" class="query-header">
        <div class="repo-left">
          <el-input autofocus v-model="repoURL" @keyup.enter.native="startAnalysis"
            :placeholder="defaultURL"></el-input>
        </div>
        <div class="repo-right">
          <el-button style="width: 100%" type="info" plain @click="startAnalysis">
            <i :class="beginStatusClass"></i>Analysis
          </el-button>
        </div>
      </el-header>

      <el-main v-if="data.length !== 0">
        <div class="common-info-container" v-if="totalDownloads">
          <!-- top total downloads -->
          <el-alert :closable="false" type="success" id="top-total">
            <i class="el-icon-download">Total Downloads: <b>{{ totalDownloads }}</b></i>
          </el-alert>

          <el-button size="mini" type="info" plain @click="toggleChart">Toggle Chart Type</el-button>
          <el-button size="mini" type="info" plain @click="setPatternDialog = true">
            Exclude Asset File
            <span style="color: red">{{ filterPattern }}</span>
          </el-button>
          <ve-chart :legend-visible="isPieChart" :data="chartData" :settings="chartSettings"></ve-chart>

          <!-- release downloads -->
          <el-card>
            <el-table v-if="downloadsByReleases.length != 0" :data="downloadsByReleases" show-summary
              sum-text="Total Downloads" max-height="300" :key="theme">
              <el-table-column prop="name" label="Release">
                <template slot-scope="scope">
                  <i class="el-icon-goods"></i>
                  <a style="margin-left: 10px">{{ scope.row.tag }}</a>
                </template>
              </el-table-column>
              <el-table-column sortable prop="count" label="Download"></el-table-column>
              <el-table-column sortable prop="percent" label="Percent %" :sort-method="sortBy"></el-table-column>
            </el-table>
          </el-card>
        </div>

        <!-- release cards -->
        <el-card v-for="release in data" :key="release.id || release.tag_name" class="release-cards">
          <div slot="header" class="clearfix">
            <span class="el-icon-star-on release-card-tag-name">{{ release.tag_name }}</span>
            <a :href="release.html_url">{{ release.name }}</a>
            <span class="release-card-right el-icon-time">
              {{ new Date(release.published_at).toLocaleDateString() }} by
              <a :href="'https://github.com/' + release.author.login">{{ release.author.login }}</a>
            </span>
          </div>

          <div>
            <el-table v-if="release.assets && release.assets.length != 0" :data="release.assets"
              :show-summary="release.assets.length > 1" sum-text="Total Downloads" :key="theme">
              <el-table-column prop="name" label="Assets">
                <template slot-scope="scope">
                  <i class="el-icon-sold-out"></i>
                  <a :href="scope.row.browser_download_url" style="margin-left: 10px">
                    {{ scope.row.name }}
                  </a>
                </template>
              </el-table-column>
              <el-table-column prop="download_count" label="Download"></el-table-column>
            </el-table>
          </div>
        </el-card>
      </el-main>
    </el-container>

    <!-- set pattern dialog -->
    <el-dialog title="Set Exclude Asset Files" :visible.sync="setPatternDialog" append-to-body>
      <p>exclude file pattern, use commas to separate multiple patterns, such as "*.yml, *.txt"</p>
      <el-input v-model="filterPattern" @keyup.enter.native="setFilePattern" />
      <div slot="footer" class="dialog-footer">
        <el-button @click="setFilePattern">Set</el-button>
      </div>
    </el-dialog>

    <div class="copyright-declare">
      Powered by <a href="https://github.com/xiesx123">xiesx123</a>
    </div>
  </div>
</template>

<script>
import gh from "parse-github-url";

export default {
  props: {
    theme: {
      type: String,
      default: "light",
    },
  },
  data() {
    return {
      repoURL: "",
      data: [],
      beginStatusClass: "",
      defaultURL: "https://github.com/xiesx123/CreatorBox",
      chartIndex: 0,
      pieRoseType: "radius",
      chartToggles: ["histogram", "pie", "pie"],
      filterPattern: "",
      setPatternDialog: false,
    };
  },
  computed: {
    isPieChart() {
      return this.chartToggles[this.chartIndex] === "pie";
    },
    chartSettings() {
      return {
        type: this.chartToggles[this.chartIndex],
        roseType: this.pieRoseType,
      };
    },
    totalDownloads() {
      let count = 0;
      for (const releaseData of this.downloadsByReleases) {
        count += releaseData.count;
      }
      return count;
    },
    chartData() {
      return {
        columns: ["tag", "count"],
        rows: this.downloadsByReleases,
      };
    },
    downloadsByReleases() {
      let download = [];
      let total = 0;

      for (const release of this.data) {
        let releaseDownload = { tag: release.tag_name, count: 0 };

        for (const asset of release.assets) {
          if (!asset) continue;
          releaseDownload.count += asset.download_count;
          total += asset.download_count;
        }

        download.push(releaseDownload);
      }

      for (const releaseData of download) {
        releaseData.percent = ((releaseData.count / total).toFixed(4) * 100).toFixed(2);
      }

      return download;
    },
  },
  watch: {
    theme() {
      this.syncQueryParams();
    },
  },
  methods: {
    syncQueryParams(extra = {}) {
      const url = new URL(window.location.href);

      const repo = (extra.repo !== undefined ? extra.repo : this.repoURL || "").trim();
      const pattern = (extra.pattern !== undefined ? extra.pattern : this.filterPattern || "").trim();
      const theme = this.theme === "dark" ? "dark" : "light";
      const auto = extra.auto;

      repo ? url.searchParams.set("repo", repo) : url.searchParams.delete("repo");
      pattern ? url.searchParams.set("pattern", pattern) : url.searchParams.delete("pattern");
      url.searchParams.set("theme", theme);

      if (auto !== undefined) {
        auto ? url.searchParams.set("auto", "1") : url.searchParams.delete("auto");
      }

      window.history.replaceState({}, "", url.toString());
    },
    startAnalysis() {
      this.beginStatusClass = "el-icon-loading";
      !this.repoURL && (this.repoURL = this.defaultURL);
      this.syncQueryParams({ auto: true });

      let url = this.repoURL;
      if (url.substr(0, 4) !== "http") {
        url = "https://" + url;
      }

      const info = gh(url);
      if (!info || !info.owner || !info.name) {
        this.$message.error("URL error!");
        this.data = [];
        this.beginStatusClass = "";
        return false;
      }

      const releaseURL = `https://api.github.com/repos/${info.owner}/${info.name}/releases?ran=` + Math.random();

      this.$http
        .get(releaseURL)
        .then((response) => {
          this.data = this.filterFile(response.body);
          this.beginStatusClass = "";
        })
        .catch((response) => {
          const body = response.body || {};
          const msg = body.message || response.statusText;
          this.$message({
            message: msg,
            type: "error",
            duration: 6000,
            showClose: true,
          });
          this.data = [];
          this.beginStatusClass = "";
        });
    },
    toggleChart() {
      this.chartIndex += 1;
      if (this.chartIndex >= this.chartToggles.length) this.chartIndex = 0;

      if (this.isPieChart) {
        this.pieRoseType = this.pieRoseType ? "" : "radius";
      }
    },
    sortBy(a, b) {
      return a.percent - b.percent;
    },
    filterFile(data) {
      const pattern = this.getfilterPattern();
      if (!pattern) return data;

      for (const release of data) {
        for (let assetIndex = 0; assetIndex < release.assets.length; assetIndex++) {
          if (release.assets[assetIndex].name.match(pattern)) {
            delete release.assets[assetIndex];
          }
        }
      }

      return data;
    },
    getfilterPattern() {
      const pattern = this.filterPattern;
      if (!pattern) return false;

      // 处理逗号分隔的多个 pattern，并转义为正则表达式的 OR 逻辑
      // 例如： "*.yml, *.json" -> ".*\.yml|.*\.json"
      const patterns = pattern
        .split(",")
        .map((p) => p.trim())
        .filter((p) => p !== "");

      if (patterns.length === 0) return false;

      const regexParts = patterns.map((p) =>
        p
          .replace(/\*/g, "__STAR__")
          .replace(/[.*+?^${}()|[\]\\]/g, "\\$&")
          .replace(/__STAR__/g, ".*")
      );

      return new RegExp(regexParts.join("|"));
    },
    setFilePattern() {
      this.setPatternDialog = false;
      this.syncQueryParams();
      this.startAnalysis();
    },
  },
  mounted() {
    const params = new URL(window.location.href).searchParams;
    const repo = params.get("repo");
    const pattern = params.get("pattern");
    const auto = params.get("auto");

    pattern && (this.filterPattern = pattern);

    if (!repo) {
      this.syncQueryParams({ auto: false });
      return true;
    }

    this.repoURL = repo;

    if (auto === "0") {
      this.syncQueryParams({ auto: false });
      return true;
    }

    this.startAnalysis();
  },
};
</script>

<style>
body a {
  color: var(--link-color);
}

.content-root {
  color: var(--text-primary);
}

.container {
  max-width: 950px;
  margin: 10px auto 16px;
}

.container .el-header,
.container .el-main {
  padding: 12px;
}

.query-header {
  border-radius: 8px;
  border: 1px solid var(--border-color);
  background: var(--bg-card);
  box-shadow: 0 8px 20px var(--shadow-color);
  display: flex;
  flex-wrap: wrap;
  align-items: center;
  gap: 10px;
}

.query-header h3 {
  width: 100%;
  color: var(--text-primary);
  margin: 8px 0 14px;
}

.repo-left {
  flex: 1;
  min-width: 0;
}

.repo-right {
  width: 125px;
  flex-shrink: 0;
}

.common-info-container {
  margin-bottom: 20px;
}

.common-info-container>.el-card,
.release-cards {
  border-radius: 8px;
  border: 1px solid var(--border-color);
  background: var(--bg-card);
  box-shadow: 0 8px 20px var(--shadow-color);
}

#top-total {
  margin-bottom: 10px;
}

.release-cards {
  margin-bottom: 15px;
}

.release-card-tag-name {
  margin-right: 8px;
}

.release-card-right {
  float: right;
  font-size: 80%;
  color: var(--text-secondary);
  margin-top: 5px;
}

.copyright-declare {
  position: fixed;
  bottom: 5px;
  left: 10px;
  font-size: 12px;
  color: var(--text-secondary);
}

.copyright-declare a {
  color: var(--text-secondary);
}

.container .el-input__inner,
.container .el-dialog,
.container .el-table,
.container .el-table tr,
.container .el-table th,
.container .el-table td,
.container .el-card,
.container .el-alert {
  background-color: var(--bg-card);
  color: var(--text-primary);
  border-color: var(--border-color);
}

.container .el-table::before {
  background-color: var(--border-color);
}

.container .el-table .has-gutter tr,
.container .el-table__row {
  border-color: var(--border-color);
}

.container .el-table td,
.container .el-table th.is-leaf {
  border-bottom: 1px solid var(--border-color);
}

.container .el-table--border th,
.container .el-table--border td {
  border-right: 1px solid var(--border-color);
}

.container .el-table--enable-row-hover .el-table__body tr:hover>td,
.container .el-table__body tr:hover>td {
  background-color: var(--bg-hover) !important;
}

.container .el-card__header {
  border-bottom: 1px solid var(--border-color);
}

.container .el-button--info.is-plain {
  color: var(--text-primary);
  background: transparent;
  border-color: var(--border-color);
}

.container .el-button--info.is-plain:hover,
.container .el-button--info.is-plain:focus,
.container .el-button--info.is-plain:active {
  color: var(--text-primary);
  background: var(--bg-hover);
  border-color: var(--border-color);
  outline: none;
}

@media (max-width: 768px) {
  .query-header {
    flex-direction: column;
    align-items: stretch;
  }

  .repo-left {
    width: 100%;
    order: 1;
  }

  .repo-right {
    width: 100%;
    order: 2;
  }
}
</style>
