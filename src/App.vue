<template>
  <div id="app">
    <img src="./assets/logo.png">
    <div>
      <el-container>
        <el-header>
        </el-header>
        <el-main>
          <el-row>
            <el-col :span="20" :offset="2">
              <el-row>
                <el-button type="primary" @click="showNewWindow()" icon="el-icon-plus" >新增股票</el-button>
                <el-button type="success" @click="getList()" icon="el-icon-refresh-left" circle>刷新列表</el-button>
              </el-row>
              <el-row>
                <div class="grid-content bg-purple-light">
                  <template>
                    <el-table :data="tableData" style="width: 100%">
                      <el-table-column prop="stock_id" fixed label="股票编号" width="150"></el-table-column>
                      <el-table-column prop="stock_name" fixed label="股票名" width="120"></el-table-column>
                      <el-table-column prop="extrem_high" fixed label="最高价" width="120"></el-table-column>
                      <el-table-column prop="extremly_low" fixed label="最低价" width="120"></el-table-column>
                      <el-table-column fixed="right" label="操作" width="300">
                        <template slot-scope="scope">
                          <el-button @click="handleCalcClick(scope.row)" type="primary" icon="el-icon-data-line"
                            circle></el-button>
                          <el-button @click="handleEditClick(scope.row)" type="primary" icon="el-icon-edit"
                            circle></el-button>
                          <el-button @click="handleRemoveClick(scope.row)" type="danger" icon="el-icon-delete"
                            circle></el-button>
                        </template>
                      </el-table-column>
                    </el-table>
                  </template>
                </div>
              </el-row>
            </el-col>
          </el-row>
          <el-dialog title="编辑股票信息" :visible.sync="dialogVisible" width="50%" :before-close="handleClose">
            <el-form :model="form">
              <el-form-item label="股票编号" :label-width="formLabelWidth">
                <el-input v-model="form.stock_id" autocomplete="off"></el-input>
              </el-form-item>
              <el-form-item label="股票名" :label-width="formLabelWidth">
                <el-input v-model="form.stock_name" autocomplete="off"></el-input>
              </el-form-item>
              <el-form-item label="最高价" :label-width="formLabelWidth">
                <el-input v-model="form.extrem_high" type="number" max="10000" min="0.01" step="0.01"
                  autocomplete="off"></el-input>
              </el-form-item>
              <el-form-item label="最低价" :label-width="formLabelWidth">
                <el-input v-model="form.extremly_low" type="number" max="10000" min="0.01" step="0.01"
                  autocomplete="off"></el-input>
              </el-form-item>
            </el-form>
            <div slot="footer" class="dialog-footer">
              <el-button type="primary" @click="updateStock()">确 定</el-button>
              <el-button @click="closeWindow()">取 消</el-button>
            </div>
          </el-dialog>
          
        </el-main>
        <el-footer></el-footer>
      </el-container>
    </div>
  </div>
</template>

<script>

const axios = require('axios').default;

export default {
  data() {
    return {
      info: null,
      baseURL: window.location.origin+'/trade_api',
      tableData: [],
      dialogVisible: false,
      form: {},
    }
  },
  mounted() {
    this.getList()
  },
  methods: {
    showCalcResult(data){
      let _this = this
      _this.$alert(data, '计算结果', {
            confirmButtonText: '关闭',
            dangerouslyUseHTMLString: true
          });
    },
    handleCalcClick(row){
      let _this = this
      if("logs" in row){
        console.log("包含logs")
        _this.showCalcResult(row.logs)
      }else{
        axios
        .post(_this.baseURL + '/stocks/calc/'+row.stock_id)
        .then(function (response) {
          console.log("response", response)
          if (200 == response.data.code) {
            var htmlCode=""
            for (const ele of response.data.data) {
              htmlCode=htmlCode+ele+"<br>"
            }
            row.logs=htmlCode
            console.log("row1", row)
            _this.showCalcResult(row.logs)
          }
        })
      }
    },
    resetForm() {
      let _this = this
      _this.form = {
        stock_id: "",
        stock_name: "",
        extrem_high: 0,
        extremly_low: 0
      }
    },
    getList() {
      let _this = this
      axios
        .get(_this.baseURL + '/stocks/list')
        .then(function (response) {
          console.log("response", response)
          if (200 == response.data.code) {
            _this.tableData = response.data.data
            console.log(_this.tableData)
          }
        })
    },
    updateStock() {
      let _this = this
      _this.dialogVisible = false
      var stock = _this.form
      axios
        .post(_this.baseURL + '/stocks/update', stock)
        .then(function (response) {
          console.log("response", response)
          if (200 == response.data.code) {
            console.log(_this.tableData)
            _this.getList()
          }
        })
    },
    showNewWindow() {
      let _this = this
      _this.resetForm()
      _this.dialogVisible = true
    },
    handleEditClick(row) {
      let _this = this
      _this.resetForm()
      _this.form = row
      _this.dialogVisible = true
    },
    closeWindow() {
      let _this = this
      _this.dialogVisible = false
      _this.resetForm()
    },
    handleRemoveClick(row) {
      let _this = this
      axios
        .post(_this.baseURL + '/stocks/remove/'+row.stock_id)
        .then(function (response) {
          console.log("response", response)
          if (200 == response.data.code) {
            console.log(_this.tableData)
            _this.getList()
          }
        })
    },

  },
}
</script>

<style>
#app {
  font-family: Helvetica, sans-serif;
  text-align: center;
}
</style>
