<template>
  <div id="jobs">
    <Header class="main-header"></Header>
    <div class="content">
      <h1 class="title">RPM Packages</h1>
      <p class="summary">这是oepkgs仓库的搜索入口，请输入您要查询的软件名</p>
      <el-form :inline="true" class="demo-form-inline">
        <el-form-item label="reports per page" class="select-pages">
          <el-select
            v-model="listQuery.page_size"
            placeholder="请选择"
            size="medium"
            class="select-page"
            @change="getSrpms(listQuery)"
          >
            <el-option
              v-for="item in pageSizeOptions"
              :key="item"
              :label="item"
              :value="item"
            ></el-option>
          </el-select>
        </el-form-item>
        <el-form-item class="search">
          <el-input
            class="search-input"
            v-model="filter"
            placeholder="search software name like: python"
            size="medium"
            @keydown.enter.native="handSearch"
            clearable
            @clear="clearFilter"
          >
            <el-button
              slot="append"
              icon="el-icon-search"
              @click="handSearch"
            ></el-button>
          </el-input>
        </el-form-item>
      </el-form>
      <el-table :data="tableData" stripe class="jobs-data">
        <el-table-column label="序号" type="index" width="50"></el-table-column>
        <el-table-column
          :label="getLable(item)"
          :key="index"
          v-for="(item, index) in tableHead"
          :prop="item"
          :width="getWidth(item)"
        >
          <template slot-scope="scope">
            <el-tooltip
              class="item"
              effect="light"
              :content="String(scope.row[item])"
              placement="top-start"
              :disabled="toolDisabled"
            >
              <div slot="content" v-if="item == 'error_ids'">
                <template v-for="(ids, idsIndex) in scope.row[item]">
                  <p :key="idsIndex">{{ ids + "," }}</p>
                </template>
              </div>
              <div slot="content" v-else>{{ scope.row[item] }}</div>
              <span
                class="goUrl wrap"
                @click="goBlank(
                    updateRT(scope.row.result_url) + '/'
                  )
                "
                v-if="item === 'result_url'"
                @mouseover="showtip(item)"
                >测试结果</span
              >
              <span
                class="goUrl wrap"
                @click="
                  goBlank(
                    updateRepo(scope.row)
                  )
                "
                v-else-if="item === 'repo_name'"
                @mouseover="showtip(item)"
                >仓库地址</span
              >
              <span
                class="goUrl wrap"
                @click="goBlank(
                    updateRT(scope.row.rpmbuild_result_url) + '/'
                  )
                "
                v-else-if="item === 'rpmbuild_result_url'"
                @mouseover="showtip(item)"
                >测试结果</span
              >
              <span
                class="goUrl wrap"
                @click="goBlank(scope.row.srpm_addr)"
                v-else-if="item == 'srpm_addr'"
                @mouseover="showtip(item)"
                >源码地址</span
              >
              <span
                class="wrap"
                v-else-if="item == 'error_ids'"
                @mouseover="showtip(item)"
              >
                <template v-for="item in scope.row[item]">
                  <span :key="item">{{ item }}</span>
                </template>
              </span>
              <span class="wrap" v-else @mouseover="showtip(item)">{{
                scope.row[item]
              }}</span>
            </el-tooltip>
          </template>
        </el-table-column>
      </el-table>
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page.sync="listQuery.page_num"
        :page-size="listQuery.page_size"
        layout="total, prev, pager, next"
        :total="total"
      >
      </el-pagination>
    </div>
  </div>
</template>
<script>
import { getSrpms } from "../../api/jobs.js";
import Header from "@/components/Header";
export default {
  name: "Oepkgs",
  components: {
    Header,
  },
  data() {
    return {
      pageSizeOptions: [10, 20, 30],
      tableData: [],
      totalData: [],
      total: 1000,
      filter: null,
      tableHeader: {},
      toolDisabled: false,
      jobsQuery: {},
      listQuery: {
        page_size: 10,
        page_num: 0,
      },
      tableHead: [
        "srpm_addr",
        "softwareName",
        "version",
        "arch",
        "os",
        "repo_name",
        "result_url",
        "rpmbuild_result_url",
      ],
    };
  },
  computed: {
    filtpackages() {
      return this.tableData.filter((p) => {
        return p.softwareName.indexOf(this.filter) !== -1;
      });
    },
  },
  methods: {
    showtip(item) {
      if (
        item == "srpm_addr" ||
        item == "softwareName"
      ) {
        this.toolDisabled = false;
      } else {
        this.toolDisabled = true;
      }
    },
    parseFilter() {
      this.clearListQuery();
      this.listQuery["softwareName"] = this.filter;
    },
    handleCurrentChange(val) {
      this.listQuery.page_num = val;
      this.getSrpms(this.listQuery);
    },
    clearListQuery() {
      var tmp = { page_size: null, page_num: null };
      tmp.page_size = this.listQuery.page_size;
      tmp.page_num = 1;
      this.listQuery = tmp;
    },
    clearFilter() {
      this.clearListQuery();
      this.getSrpms(this.listQuery);
    },
    handSearch() {
      if (this.filter) {
        this.parseFilter();
        this.getSrpms(this.listQuery);
      } else {
        this.clearListQuery();
        this.getSrpms(this.listQuery);
      }
    },
    updateURL(data) {
      var routeQuery = {};
      for (var x in data) {
        if (x != "page_num" && x != "page_size") {
          routeQuery[x] = data[x];
        }
      }
      this.$router.push({
        path: this.$route.path,
        query: routeQuery,
      });
    },
    getSrpms(data) {
      this.updateURL(data);
      getSrpms(data).then((res) => {
        this.jobsQuery = res;
        this.tableData = this.jobsQuery.info;
        this.total = this.jobsQuery.total;
      });
    },
    goBlank(src) {
      window.open(src, "_blank");
    },
    handleSizeChange(val) {
      this.listQuery.page_size = val;
      this.getSrpms(this.listQuery);
    },
    updateOs(str) {
      let resultStr = "";
      let strArr = str.split(" ");
      let firstStr = strArr[0];
      let sedStr = strArr[1];
      if (firstStr) {
        resultStr = "openEuler-" + sedStr;
      } else {
        resultStr = firstStr;
      }
      return resultStr;
    },
    updateRepo(data) {
      let resultStr = "";
      let strArr = data.repo_name.split("/");
      let firstStr = strArr[0];
      let lastStr = strArr.slice(-1);
      if (firstStr == "stable" || firstStr == "unstable" ) {
        if (lastStr == "6" || lastStr == "7" || lastStr == "8" ) {
          strArr[strArr.length - 1] = 'c' + lastStr;
          resultStr = 'https://repo.oepkgs.net/openEuler/rpm/' +
                      this.updateOs(data.os) +
                      '/' + strArr.slice(1).join('/') + '/' + data.arch + '/';
        } else {
          resultStr = 'https://repo.oepkgs.net/openEuler/rpm/' +
                      this.updateOs(data.os) +
                      '/' + strArr.slice(1).join('/') + '/' + data.arch + '/';
        }
      } else if (firstStr == "experimental") {
        if (data.srpm_addr.split('/').length > 3) {
          resultStr = "https://gitee.com/src-openeuler/" + data.srpm_addr.split('/').slice(-1)[0].split('-').slice(0,-2).join('-');
        } else {
          resultStr = "https://gitee.com/src-openeuler/" + data.srpm_addr.split('/').slice(-1);
        }
      } else {
        resultStr = 'https://repo.oepkgs.net/openEuler/rpm/' +
                    this.updateOs(data.os) + '/' + data.repo_name + '/' + data.arch + '/';
      }
      return resultStr;
    },
    updateRT(str) {
      let resultStr = str.replace("api.compass-ci.openeuler.org:20007", "api.compass-ci.openeuler.org");
      return resultStr;
    },
    getWidth(item) {
      if (
        item == "srpm_addr" ||
        item == "repo_name" ||
        item == "version" ||
        item == "arch"
      ) {
        return 100;
      } else if (item == "os") {
        return 200;
      } else if (item == "softwareName") {
        return 300;
      } else if (item == "result_url" || item == "rpmbuild_result_url") {
        return 160;
      }
    },
    getLable(item) {
      if (item == "result_url") {
        return "install_result_url";
      }
      return item;
    },
  },
  mounted() {
    var data = this.$route.query;
    data.page_size = this.listQuery.page_size;
    data.page_num = this.listQuery.page_num;

    this.getSrpms(data);
  },
};
</script>


<style lang="less" scoped>
.jobs-data {
  width: 100%;
}
.search {
  margin-left: 92px;
  @media screen and (max-width: 1000px) {
    margin-left: 0;
  }
}
.search-input {
  width: 550px;
}
.select-page {
  width: 113px;
}
/deep/.el-form-item__label {
  font-weight: bold;
}
.ids-num {
  color: #002fa7;
}
/deep/.el-table th > .cell {
  font-weight: bold;
}
</style>

