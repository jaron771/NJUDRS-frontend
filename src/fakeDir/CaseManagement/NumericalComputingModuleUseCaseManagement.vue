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
        <el-button
          type="primary"
          icon="el-icon-plus"
          class="handle-del"
          @click="insertNew"
          >新增</el-button
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
      <el-table-column
        type="index"
        width="50"
        align="center"
        label="序号"
      ></el-table-column>
      <el-table-column
        prop="name"
        label="用例名称"
        align="center"
      ></el-table-column>
      <el-table-column
        prop="object"
        label="测试接口"
        align="center"
      ></el-table-column>
      <el-table-column
        prop="service"
        label="资源对象"
        align="center"
      ></el-table-column>
      <el-table-column
        prop="description"
        label="用例简介"
        align="center"
      ></el-table-column>
      <el-table-column
        prop="file"
        label="用例文件"
        align="center"
      ></el-table-column>
      <!-- <el-table-column prop="content" label="详细用例" align="center"></el-table-column> -->
      <el-table-column
        prop="checkpoints"
        label="用例数量"
        align="center"
        width="100"
      ></el-table-column>
      <el-table-column label="操作" width="180" align="center">
        <template slot-scope="scope">
          <el-button
            type="success"
            icon="el-icon-edit"
            @click="handleEdit(scope.$index, scope.row)"
            >编辑</el-button
          >
          <el-button
            type="danger"
            icon="el-icon-delete"
            @click="handleDelete(scope.$index, scope.row)"
            >删除</el-button
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

    <!-- 编辑弹出框 -->
    <el-dialog title="修改订单" :visible.sync="editVisible" width="30%">
      <el-form ref="editForm" :model="editForm" label-width="150px">
        <el-form-item label="用例名称" prop="name">
          <el-input v-model="editForm.name"></el-input>
        </el-form-item>
        <el-form-item label="测试接口" prop="object">
          <el-input v-model="editForm.object"></el-input>
        </el-form-item>
        <el-form-item label="资源对象" prop="service">
          <el-select v-model="editForm.service">
            <el-option
              v-for="item in serviceTable.list"
              :label="item.name"
              :value="item.name"
              :key="item.name"
            ></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="用例简介" prop="description">
          <el-input v-model="editForm.description"></el-input>
        </el-form-item>
        <el-form-item label="用例文件命名" prop="file">
          <el-input v-model="editForm.file" disabled></el-input>
        </el-form-item>
        <el-form-item label="详细用例" prop="content">
          <el-input
            v-model="editForm.content"
            type="textarea"
            rows="10"
          ></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editVisible = false">取消</el-button>
        <el-button type="primary" @click="saveEdit">修改</el-button>
      </span>
    </el-dialog>
    <!-- 新建弹出框 -->
    <el-dialog title="新增订单" :visible.sync="insertVisible" width="30%">
      <el-form ref="insertForm" :model="insertForm" label-width="150px">
        <el-form-item label="用例名称" prop="name">
          <el-input v-model="insertForm.name"></el-input>
        </el-form-item>
        <el-form-item label="测试接口" prop="object">
          <el-input v-model="insertForm.object"></el-input>
        </el-form-item>
        <el-form-item label="资源对象" prop="service">
          <el-select v-model="insertForm.service">
            <el-option
              v-for="item in serviceTable.list"
              :label="item.name"
              :value="item.name"
              :key="item.name"
            ></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="用例简介" prop="description">
          <el-input v-model="insertForm.description"></el-input>
        </el-form-item>
        <el-form-item label="用例文件命名" prop="file">
          <el-input v-model="insertForm.file"></el-input>
        </el-form-item>
        <el-form-item label="详细用例" prop="content">
          <el-input
            v-model="insertForm.content"
            type="textarea"
            rows="10"
          ></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="insertVisible = false">取消</el-button>
        <el-button type="primary" @click="saveInsert">新增</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
import api from "@api/NumericalComputingModuleUseCaseManagement";
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
      serviceTable: {}
    };
  },
  watch: {},
  computed: {},
  methods: {
    getData() {
      let data = JSON.parse(localStorage.getItem("numericalManageTable"));
      if (!data) this.$router.push("/");
      else {
        this.tableData = data.list;
        this.pageTotal = data.pageTotal || 50;
      }
    },
    handlePageChange(val) {
      this.$set(this.query, "pageIndex", val);
      this.getOrderData();
    },
    handleEdit(index, row) {
      this.editForm = JSON.parse(JSON.stringify(row)); //深拷贝
      this.idex = index;
      this.editVisible = true;
    },
    saveEdit() {
      let tempPar = JSON.parse(JSON.stringify(this.editForm));
      this.$message.success("修改成功!");
      this.editVisible = false;
      this.$set(this.tableData, this.idex, tempPar);
      localStorage.setItem(
        "numericalManageTable",
        JSON.stringify({ list: this.tableData, pageTotal: this.pageTotal })
      );
    },
    // 删除操作
    handleDelete(index, row) {
      // 二次确认删除
      this.$confirm("确定要删除吗？", "提示", {
        type: "warning"
      })
        .then(() => {
          this.$message.success("删除成功");
          this.tableData.splice(index, 1);
          localStorage.setItem(
            "numericalManageTable",
            JSON.stringify({ list: this.tableData, pageTotal: this.pageTotal })
          );
        })
        .catch(() => {});
    },
    //处理新增订单弹窗
    insertNew() {
      this.insertForm = {};
      this.insertVisible = true;
    },
    //保存新增订单
    saveInsert() {
      this.insertForm["checkpoints"] =
        this.insertForm["content"].split("\n").length / 2;
      //桩
      this.insertForm["fileUrl"] =
        "https://richardzpc-oss.oss-cn-beijing.aliyuncs.com/CalTest5.xlsx";
      let tempPar = JSON.parse(JSON.stringify(this.insertForm));
      this.$message.success("新增成功!");
      this.insertVisible = false;
      this.tableData.push(JSON.parse(JSON.stringify(tempPar)));
      localStorage.setItem(
        "numericalManageTable",
        JSON.stringify({ list: this.tableData, pageTotal: this.pageTotal })
      );
    }
  },
  created() {
    this.getData();
    this.serviceTable = JSON.parse(localStorage.getItem("serviceTable"));
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
