<template>
  <div id="repos">
    <Header class="main-header"></Header>
    <div class="content">
      <h1 class="title">Repos</h1>
      <p class="summary">你的开源软件在这里吗？</p>
      <el-form :inline="true" class="demo-form-inline">
        <el-form-item label="reports per page" class="select-pages">
          <el-select
            v-model="listQuery.page_size"
            placeholder="请选择"
            size="medium"
            class="select-page"
            @change="getAllRepos"
          >
            <el-option v-for="item in pageSizeOptions" :key="item" :label="item" :value="item"></el-option>
          </el-select>
        </el-form-item>
        <el-form-item class="search">
          <el-input
            v-model="listQuery.git_repo"
            placeholder="search git_repo"
            size="medium"
            @keydown.enter.native="searchRepos"
            clearable
            @clear="searchRepos"
          >
            <el-button slot="append" icon="el-icon-search" @click="searchRepos"></el-button>
          </el-input>
        </el-form-item>
      </el-form>
      <el-table :data="reposList" stripe class="repos-data">
        <el-table-column label="序号" type="index" width="50"></el-table-column>
        <el-table-column prop="git_repo" label="git_repo">
          <template slot-scope="scope">
            <span @click="goTree(scope.row.git_repo)" class="goUrl">{{ scope.row.git_repo }}</span>
          </template>
        </el-table-column>
        <el-table-column label="git_url">
          <template slot-scope="scope">
            <span @click="goRepos(scope.row.git_url)" class="goUrl">{{ scope.row.git_url }}</span>
          </template>
        </el-table-column>
      </el-table>
      <el-pagination
        @current-change="handleCurrentChange"
        :current-page="currentPage"
        layout="total, pager, jumper"
        :total="repoData.total"
        class="pagination"
        :pager-count="5"
        :page-size="listQuery.page_size"
        :small="isSmall"
      ></el-pagination>
    </div>
  </div>
</template>
<script>
import { getAllRepos } from "../../api/repo";
import Header from "@/components/Header";
export default {
  name: "Repos",
  components: {
    Header
  },
  data() {
    return {
      listQuery: {
        git_repo: null,
        page_size: 10,
        page_num: 1
      },
      repoData: {},
      reposList: [],
      pageSizeOptions: [10, 20, 30],
      isSmall: false,
      currentPage: 1
    };
  },
  methods: {
    getAllRepos() {
      getAllRepos(this.listQuery).then(res => {
        this.repoData = res;
        this.reposList = this.repoData.repos;
        if (this.isMobile()) {
          this.isSmall = true;
        } else {
          this.isSmall = false;
        }
      });
    },
    searchRepos() {
      var routeQuery = null;
      if (this.listQuery.git_repo) {
        routeQuery = { git_repo: this.listQuery.git_repo };
      }
      this.$router.push({
        path: this.$route.path,
        query: routeQuery
      });
      this.getAllRepos();
    },
    goTree(item) {
      this.$router.push({
        path: "/tree",
        query: {
          upstream_repo: item
        }
      });
    },
    goRepos(src) {
      let srcReg = /^git/;
      if (srcReg.test(src)) {
        src = src.replace(/^git/, "https");
        window.open(src, "_blank");
      } else {
        window.open(src, "_blank");
      }
    },
    handleCurrentChange(val) {
      this.listQuery.page_num = val;
      this.getAllRepos();
    },
    isMobile() {
      let flag = navigator.userAgent.match(
        /(phone|pad|pod|iPhone|iPod|ios|iPad|Android|Mobile|BlackBerry|IEMobile|MQQBrowser|JUC|Fennec|wOSBrowser|BrowserNG|WebOS|Symbian|Windows Phone)/i
      );
      return flag;
    }
  },
  mounted() {
    this.listQuery.git_repo = this.$route.query.git_repo;
    this.getAllRepos();
  }
};
</script>
<style lang="less" scoped>
.page-tips {
  font-weight: bold;
  margin-right: 2%;
}
.jobs-tips {
  padding: 2% 0;
  line-height: 30px;
}
.search {
  margin-left: 92px;
  @media screen and (max-width: 1000px) {
    margin-left: 0;
  }
}
.select-page {
  width: 113px;
}
/deep/.el-form-item__label {
  font-weight: bold;
}
/deep/.el-table th > .cell {
  font-weight: bold;
}
</style>