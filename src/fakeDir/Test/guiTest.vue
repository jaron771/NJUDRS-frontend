<template>
  <div>
    <div class="crumbs">
      <el-breadcrumb separator="/">
        <el-breadcrumb-item>
          <i class="el-icon-lx-cascades"></i> 数值计算模块用例管理
        </el-breadcrumb-item>
      </el-breadcrumb>
    </div>
    <div class="container" style="padding: 0px">
      <div class="handle-box">
        <el-button type="primary" class="handle-del" @click="allTest"
          >一键测试</el-button
        >
      </div>
    </div>
    <el-table
      :data="tableData"
      border
      class="table"
      ref="Table"
      header-cell-class-name="table-header"
    >
</el-table-column>
      <el-table-column label="测试状态" align="center" width="200">
        <template slot-scope="scope">
          <el-button type="danger" plain round v-if="scope.row['ran'] == 0"
            >未运行</el-button
          >
          <el-button
            type="success"
            plain
            round
            v-else-if="scope.row['ran'] == 1"
            >正在运行</el-button
          >
          <el-progress
            :percentage="(scores[scope.$index] * 100) / total[scope.$index]"
            status="success"
            v-else-if="
              scope.row['ran'] == 2 &&
                scores[scope.$index] / total[scope.$index] == 1
            "
          ></el-progress>
          <el-progress
            :percentage="(scores[scope.$index] * 100) / total[scope.$index]"
            status="exception"
            v-else-if="
              scope.row['ran'] == 2 &&
                scores[scope.$index] / total[scope.$index] != 1
            "
          ></el-progress>
        </template>
      </el-table-column>
      <el-table-column label="操作" align="center" width="250px">
        <template slot-scope="scope">
          <el-link
            v-if="scope.row['ran'] == 2"
            target="_blank"
            :href="scope.row['fileUrl']"
          >
            <el-button type="success" class="mr10">下载测试结果</el-button>
          </el-link>
          <el-button type="primary" @click="singleTest(scope.$index, scope.row)"
            >测试</el-button
          >
        </template>
      </el-table-column>
    </el-table>
    <div class="pagination">
      <el-pagination
        background
        layout="total, prev, pager, next"
        :current-page="query.pageIndex"
        :page-size="query.pageSize"
        :total="pageTotal"
        @current-change="handlePageChange"
      ></el-pagination>
    </div>
  </div>
</template>

<script>
import api from "@api/guiTest";
export default {
  components: {},
  props: {},
  data() {
    return {
      query: {
        pageIndex: 1,
        pageSize: 10
      },
      tableData: [],
      pageTotal: 0,
      editVisible: false,
      editForm: {},
      idex: 0,
      insertVisible: false,
      insertForm: {},
      scores: [3, 3],
      total: [3, 3],
      waitTime: [30000, 35000]
    };
  },
  watch: {},
  computed: {},
  methods: {
    getData() {
      let data = JSON.parse(localStorage.getItem("GUIManageTable"));
      if (!data) this.$router.push("/EngineeringCalculationModuleTest");
      else {
        for (let i = 0; i < data.list.length; i++) {
          data.list[i]["ran"] = 0;
        }
        this.tableData = data.list;
        this.pageTotal = data.pageTotal || 50;
      }
    },
    handlePageChange(val) {
      this.$set(this.query, "pageIndex", val);
      this.getOrderData();
    },
    // 删除操作
    singleTest(index, row) {
      api.runTest({ index: index });
      this.$set(this.tableData[index], "ran", 1);
      setTimeout(() => {
        this.tableData[index]["ran"] = 2;
      }, this.waitTime[index]);
    },
    allTest() {
      for (let i = 0; i < this.tableData.length; i++) {
        this.$set(this.tableData[i], "ran", 1);
        setTimeout(() => {
          this.tableData[i]["ran"] = 2;
        }, this.waitTime[i]);
      }
      console.log(JSON.stringify(this.tableData));
    },
    singleSubTest(index, row) {
      this.$set(this.tableData[index], "ran", 1);
      setTimeout(() => {
        this.tableData[index]["ran"] = 2;
      }, this.subWaitTime[index]);
    },
    allSubTest() {
      for (let i = 0; i < this.tableData.length; i++) {
        this.$set(this.tableData[i], "ran", 1);
        setTimeout(() => {
          this.tableData[i]["ran"] = 2;
        }, this.subWaitTime[i]);
      }
      console.log(JSON.stringify(this.tableData));
    }
  },
  created() {
    this.getData();
  },
  mounted() {}
};
</script>
<style scoped>
.handle-box {
  margin: 10px;
}

.handle-select {
  width: 120px;
}

.handle-input {
  width: 300px;
  display: inline-block;
}
.table {
  width: 100%;
  font-size: 14px;
}
.red {
  color: #ff0000;
}
.mr10 {
  margin-right: 10px;
}
.table-td-thumb {
  display: block;
  margin: auto;
  width: 40px;
  height: 40px;
}
</style>
