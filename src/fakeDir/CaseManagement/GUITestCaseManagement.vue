<template>
  <div>
    <div class="crumbs">
      <el-breadcrumb separator="/">
        <el-breadcrumb-item>
          <i class="el-icon-lx-cascades"></i> GUI测试用例管理
        </el-breadcrumb-item>
      </el-breadcrumb>
    </div>
    <div v-loading="loading">
      <div class="container" style="padding: 10px">
        <div class="handle-box" style="display: inline">
          <el-button
            type="primary"
            icon="el-icon-plus"
            class="handle-del"
            @click="insertNew"
            >新增</el-button
          >
        </div>
        <!--                <div class="handle-box" style="display: inline; margin-left: 10px">-->
        <!--                    <el-button type="primary" icon="el-icon-plus" class="handle-del" @click="startLoading">脚本录制</el-button>-->
        <!--                </div>-->
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
          label="脚本文件"
          align="center"
        ></el-table-column>
        <el-table-column
          prop="checkpoints"
          label="检查点数量"
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
    </div>

    <!-- 编辑弹出框 -->
    <el-dialog title="修改订单" :visible.sync="editVisible" width="50%">
      <el-form ref="editForm" :model="editForm" label-width="150px">
        <el-form-item label="用例名称" prop="name">
          <el-input v-model="editForm.name"></el-input>
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
        <el-form-item label="脚本文件名" prop="file">
          <el-input v-model="editForm.file" disabled></el-input>
        </el-form-item>
        <el-form-item label="脚本" prop="content">
          <el-input
            v-model="editForm.content"
            type="textarea"
            rows="10"
          ></el-input>
        </el-form-item>
        <el-form-item label="检查点" prop="checkpoints">
          <el-upload
            class="upload-demo"
            action="https://jsonplaceholder.typicode.com/posts/"
            :on-preview="handlePreview"
            :on-remove="handleRemove"
            :file-list="editForm.fileList"
            list-type="picture"
            multiple
            :auto-upload="false"
          >
            <el-button size="small" type="primary">点击上传</el-button>
          </el-upload>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editVisible = false">取消</el-button>
        <el-button type="primary" @click="saveEdit">修改</el-button>
      </span>
    </el-dialog>
    <!-- 新建弹出框 -->
    <el-dialog title="新增订单" :visible.sync="insertVisible" width="50%">
      <el-form ref="insertForm" :model="insertForm" label-width="150px">
        <el-form-item label="用例名称" prop="name">
          <el-input v-model="insertForm.name"></el-input>
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
        <el-form-item label="脚本文件命名" prop="file">
          <el-input v-model="insertForm.file"></el-input>
        </el-form-item>
        <el-form-item label="脚本" prop="content">
          <el-input
            v-model="insertForm.content"
            type="textarea"
            rows="10"
          ></el-input>
        </el-form-item>
        <el-form-item label="检查点" prop="checkpoints">
          <el-upload
            class="upload-demo"
            action="https://jsonplaceholder.typicode.com/posts/"
            :on-preview="handlePreview"
            :on-remove="handleRemove"
            :file-list="insertForm.fileList"
            list-type="picture"
            multiple
            :on-change="addPic"
            :auto-upload="false"
          >
            <el-button size="small" type="primary">点击上传</el-button>
          </el-upload>
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
import api from "@api/GUITestCaseManagement";
export default {
  components: {},
  props: {},
  data() {
    return {
      query: {
        pageIndex: 1,
        pageSize: 10
      },
      serviceTable: {},
      tableData: [],
      pageTotal: 0,
      editVisible: false,
      editForm: {},
      idex: 0,
      insertVisible: false,
      insertForm: {},
      loading: false
    };
  },
  watch: {},
  computed: {},
  methods: {
    getData() {
      let data = JSON.parse(localStorage.getItem("GUIManageTable"));
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
        "GUIManageTable",
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
            "GUIManageTable",
            JSON.stringify({ list: this.tableData, pageTotal: this.pageTotal })
          );
        })
        .catch(() => {});
    },
    //处理新增弹窗
    insertNew() {
      this.insertForm = {};
      // this.insertForm['checkpoints'] = 3;
      this.insertVisible = true;
    },
    //保存新增
    saveInsert() {
      this.insertForm["checkpoints"] = 3;
      this.insertForm["fileUrl"] =
        "https://richardzpc-oss.oss-cn-beijing.aliyuncs.com/GUITest2.xlsx";
      this.insertForm["fileList"] = [
        {
          name: "1.png",
          url: "https://richardzpc-oss.oss-cn-beijing.aliyuncs.com/4.png"
        },
        {
          name: "2.png",
          url: "https://richardzpc-oss.oss-cn-beijing.aliyuncs.com/5.png"
        },
        {
          name: "3.png",
          url: "https://richardzpc-oss.oss-cn-beijing.aliyuncs.com/6.png"
        }
      ];
      let tempPar = JSON.parse(JSON.stringify(this.insertForm));
      this.$message.success("新增成功!");
      this.insertVisible = false;
      this.tableData.push(JSON.parse(JSON.stringify(tempPar)));
      localStorage.setItem(
        "GUIManageTable",
        JSON.stringify({ list: this.tableData, pageTotal: this.pageTotal })
      );
    },
    handleRemove(file, fileList) {
      console.log(file, fileList);
    },
    handlePreview(file) {
      console.log(file);
    },
    startLoading() {
      this.loading = true;
      setTimeout(() => {
        this.loading = false;
      }, 5000);
    },
    addPic() {
      if (this.insertForm.checkpoints) {
        this.insertForm.checkpoints += 1;
      } else {
        this.insertForm.checkpoints = 1;
      }
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
  margin: 0px;
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
