<template>
  <div id="jobs">
    <Header class="main-header"></Header>
    <div class="content">
      <h1 class="title">Jobs</h1>
      <p class="summary">开源软件的测试结果如何，这里告诉你答案</p>
      <el-form :inline="true" class="demo-form-inline">
        <el-form-item label="reports per page" class="select-pages">
          <el-select
            v-model="listQuery.page_size"
            placeholder="请选择"
            size="medium"
            class="select-page"
            @change="getJobs(listQuery)"
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
            placeholder="search jobs like: suite=iperf os=openeuler"
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
          :label="item"
          :key="index"
          v-for="(item, index) in tableHead"
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
                @click="goBlank(testBoxUrl + checkStr(scope.row.testbox))"
                v-if="item === 'testbox'"
                @mouseover="showtip(item)"
                >{{ scope.row[item] }}</span
              >
              <span
                class="goUrl wrap"
                @click="goBlank(resultUrl + scope.row.result_root + '/')"
                v-else-if="item === 'job_state'"
                @mouseover="showtip(item)"
                >{{ scope.row[item] }}</span
              >
              <span
                class="goUrl wrap"
                @click="goTree(scope.row[item])"
                v-else-if="item == 'upstream_repo'"
                @mouseover="showtip(item)"
                >{{ scope.row[item] }}</span
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
        @current-change="handleCurrentChange"
        :current-page.sync="listQuery.page_num"
        layout="total, pager, jumper"
        :total="jobsQuery.total"
        class="pagination"
        :pager-count="5"
        :page-size="listQuery.page_size"
        :small="isSmall"
      ></el-pagination>
    </div>
  </div>
</template>
<script>
import { getJobs } from "../../api/jobs.js";
import { BASEURLTESTBOX, BASEURLRESULT } from "../../utils/baseUrl.js";
import Header from "@/components/Header";
export default {
  name: "Jobs",
  components: {
    Header,
  },
  data() {
    return {
      tableHead: [],
      tableData: [],
      state: "",
      tableHeader: {},
      filter: null,
      listQuery: {
        page_size: 10,
        page_num: 1,
      },
      pageSizeOptions: [10, 20, 30],
      jobsQuery: {},
      isSmall: false,
      toolDisabled: false,
      resultUrl: "",
      testBoxUrl: "",
    };
  },
  methods: {
    showtip(item) {
      if (
        item == "testbox" ||
        item == "error_ids" ||
        item == "upstream_repo" ||
        item == "upstream_commit" ||
        item == "start_time" ||
        item == "id" ||
        item == "error_ids" ||
        item == "upstream_branch" ||
        item == "suite" ||
        item == "os_version"
      ) {
        this.toolDisabled = false;
      } else {
        this.toolDisabled = true;
      }
    },
    goTree(item) {
      this.$router.push({
        path: "/tree",
        query: {
          upstream_repo: item,
        },
      });
    },
    goBlank(src) {
      window.open(src, "_blank");
    },
    parseFilter() {
      var filters = this.filter.split(" ");
      this.clearListQuery();
      for (var f of filters) {
        var key_vaule = f.split("=");
        this.listQuery[key_vaule[0]] = key_vaule[1];
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
    clearListQuery(){
      var tmp = { page_size: null, page_num: null };
      tmp.page_size = this.listQuery.page_size;
      tmp.page_num = 1;
      this.listQuery = tmp;
    },
    clearFilter(){
      this.clearListQuery();
      this.getJobs(this.listQuery);
    },
    handSearch() {
      if (this.filter) {
        this.parseFilter();
        this.getJobs(this.listQuery);
      } else {
        this.clearListQuery();
        this.getJobs(this.listQuery);
      }
    },
    getJobs(data) {
      this.updateURL(data);
      getJobs(data).then((res) => {
        this.jobsQuery = res;
        this.tableHead = this.jobsQuery.fields;
        this.tableData = this.jobsQuery.jobs;
        if (this.isMobile()) {
          this.isSmall = true;
        } else {
          this.isSmall = false;
        }
      });
    },
    handleCurrentChange(val) {
      this.listQuery.page_num = val;
      this.getJobs(this.listQuery);
    },
    isMobile() {
      let flag = navigator.userAgent.match(
        /(phone|pad|pod|iPhone|iPod|ios|iPad|Android|Mobile|BlackBerry|IEMobile|MQQBrowser|JUC|Fennec|wOSBrowser|BrowserNG|WebOS|Symbian|Windows Phone)/i
      );
      return flag;
    },
    checkStr(str) {
      let resultStr = "";
      let strReg = /(^vm|^dc)/;
      if (strReg.test(str)) {
        resultStr = str.split(".")[0];
      } else {
        let strArr = str.split("--");
        let firstStr = strArr[0];
        let sedStr = strArr[1];
        if (sedStr) {
          resultStr = firstStr + "--" + sedStr.split("-")[0];
        } else {
          resultStr = firstStr;
        }
      }
      return resultStr;
    },
  },
  mounted() {
    this.testBoxUrl = BASEURLTESTBOX;
    this.resultUrl = BASEURLRESULT;
    var data = this.$route.query;
    data.page_size = this.listQuery.page_size;
    data.page_num = this.listQuery.page_num;

    this.getJobs(data);
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
