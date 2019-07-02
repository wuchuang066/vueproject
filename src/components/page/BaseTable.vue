<template>
  <div class="table">
    <div class="crumbs">
      <el-breadcrumb separator="/">
        <el-breadcrumb-item>
          <i class="el-icon-lx-cascades"></i> 基础表格
        </el-breadcrumb-item>
      </el-breadcrumb>
    </div>
    <div class="container">
      <div class="handle-box">
        <el-button
          type="primary"
          icon="delete"
          class="handle-del mr10"
          @click="handleDeleteAll"
        >批量删除</el-button>
        <!-- <el-select v-model="select_cate" placeholder="筛选省份" class="handle-select mr20"> -->
        <!-- <el-option v-for="item in privinceData" :key="item.areaId" :label="item.areaName" :value="item.areaName"></el-option> -->
        <el-cascader
          :options="options"
          v-model="form.options"
          placeholder="筛选省份"
          class="handle-select mr20"
        ></el-cascader>
        <!-- <el-option key="1" label="广东省" value="广东省"></el-option>
        <el-option key="2" label="湖南省" value="湖南省"></el-option>-->
        <!-- </el-select> -->
        <el-input v-model="select_word" placeholder="筛选姓名" class="handle-input"></el-input>
        <el-button type="primary" icon="search" @click="getData" @keyup.enter.native="getData()">搜索</el-button>
        <el-button type="primary" icon="clear" @click="clear">重置</el-button>
        <el-button type="primary" icon="clear" @click="handleAdd()" class="handle-add">添加人员信息</el-button>
      </div>
      <el-table
        :data="tableData.slice((currentPage-1)*pagesize,currentPage*pagesize)"
        border
        class="table"
        ref="multipleTable"
        @selection-change="handleSelectionChange"
      >
        <el-table-column type="selection" width="55" align="center"></el-table-column>
        <el-table-column prop="date" label="日期" sortable width="230" :formatter="formatterDate"></el-table-column>
        <el-table-column prop="name" label="姓名" width="120"></el-table-column>
        <el-table-column prop="address" label="地址" :formatter="formatter"></el-table-column>
        <el-table-column label="操作" width="180" align="center">
          <template slot-scope="scope">
            <el-button
              type="text"
              icon="el-icon-edit"
              @click="handleEdit(scope.$index, scope.row)"
            >编辑</el-button>
            <el-button
              type="text"
              icon="el-icon-delete"
              class="red"
              @click="handleDelete(scope.$index, scope.row)"
            >删除</el-button>
          </template>
        </el-table-column>
      </el-table>
      <div class="pagination">
        <el-pagination
          background
          @current-change="handleCurrentChange"
          layout="prev, pager, next"
          :total="total"
        ></el-pagination>
      </div>
    </div>
    <!-- 添加弹出框 -->
    <el-dialog title="添加人员信息" :visible.sync="addVisible" width="30%">
      <el-form ref="form" :model="form" label-width="50px">
        <el-form-item label="日期">
          <el-date-picker
            type="datetime"
            placeholder="选择日期"
            v-model="form.date"
            style="width: 100%;"
          ></el-date-picker>
        </el-form-item>
        <el-form-item label="姓名">
          <el-input v-model="form.name"></el-input>
        </el-form-item>
        <el-form-item label="地址">
          <!-- <el-input v-model="form.address"></el-input> -->
          <el-cascader
            :options="options"
            v-model="form.address"
            placeholder="请选择省份"
            class="handle-select mr20"
          ></el-cascader>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addVisible = false">取 消</el-button>
        <el-button type="primary" @click="saveAdd">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 编辑弹出框 -->
    <el-dialog title="编辑" :visible.sync="editVisible" width="30%">
      <el-form ref="form" :model="form" label-width="50px">
        <el-form-item label="日期">
          <el-date-picker
            type="datetime"
            placeholder="选择日期"
            v-model="form.date"
            style="width: 100%;"
          ></el-date-picker>
        </el-form-item>
        <el-form-item label="姓名">
          <el-input v-model="form.name"></el-input>
        </el-form-item>
        <el-form-item label="地址">
          <!-- <el-input v-model="form.address"></el-input> -->
          <el-cascader
            :options="options"
            v-model="form.address"
            placeholder="请选择省份"
            class="handle-select mr20"
          ></el-cascader>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editVisible = false">取 消</el-button>
        <el-button type="primary" @click="saveEdit">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 删除提示框 -->
    <el-dialog title="提示" :visible.sync="delVisible" width="300px" center>
      <div class="del-dialog-cnt">删除不可恢复，是否确定删除？</div>
      <span slot="footer" class="dialog-footer">
        <el-button @click="delVisible = false">取 消</el-button>
        <el-button type="primary" @click="deleteRow">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 批量删除提示框 -->
    <el-dialog title="提示" :visible.sync="delAllVisible" width="300px" center>
      <div class="del-dialog-cnt">删除不可恢复，是否确定删除？</div>
      <span slot="footer" class="dialog-footer">
        <el-button @click="delAllVisible = false">取 消</el-button>
        <el-button type="primary" @click="delAll">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  name: "basetable",
  data() {
    return {
      url: "./vuetable.json",
      tableData: [],
      privinceData: [],
      cur_page: 1,
      multipleSelection: [],
      multipleSelectionTemp: [],
      total: 0,
      pagesize: 10,
      currentPage: 1,
      select_cate: "",
      select_word: "",
      del_list: [],
      is_search: false,
      addVisible: false,
      editVisible: false,
      delVisible: false,
      delAllVisible: false,
      options: [],
      sjidfj: [130000, 130300, 130304],
      form: {
        name: "",
        date: "",
        address: "",
        options: []
      },
      idx: -1
    };
  },
  created() {
    this.getData();
    this.getProvinceCode();
    this.keyupSubmit();
  },
  computed: {
    // data() {
    //   return this.tableData.filter(d => {
    //     let is_del = false;
    //     for (let i = 0; i < this.del_list.length; i++) {
    //       if (d.name === this.del_list[i].name) {
    //         is_del = true;
    //         break;
    //       }
    //     }
    //     if (!is_del) {
    //       if (
    //         d.address.indexOf(this.select_cate) > -1 &&
    //         (d.name.indexOf(this.select_word) > -1 ||
    //           d.address.indexOf(this.select_word) > -1)
    //       ) {
    //         return d;
    //       }
    //     }
    //   });
    // }
  },
  methods: {
    // 分页导航
    handleCurrentChange(val) {
      this.cur_page = val;
      this.getData();
    },
    keyupSubmit() {
      document.onkeydown = e => {
        let _key = window.event.keyCode;
        if (_key === 13) {
          this.getData();
        }
      };
    },
    getProvinceCode() {
      const data = {};
      const url = "http://localhost:8086/common/queryProvince";
      // const url = "https://www.easy-mock.com/mock/5cffccdd023e223175552420/example/getAddress";

      this.$axios
        .get(url, data)
        .then(response => {
          this.options = response.data;
        })
        .catch(res => {
          this.$message.error("系统异常，请联系管理员");
        });
    },
    getData() {
      let options = [];
      if (this.form.options != undefined) {
        options = this.form.options[2];
      }
      const data = this.$qs.stringify({
        xName: this.select_word,
        xAddress: options,
        pageNumber: this.cur_page
      });
      const url = "http://localhost:8086/table/queryTableMsg";
      this.$axios
        .post(url, data)
        .then(res => {
          this.tableData = res.data.list;
          this.total = res.data.total;
        })
        .catch(res => {
          this.$message.error("系统异常，请联系管理员");
        });
    },
    clear() {
      this.select_word = "";
      this.form.options = "";
    },
    formatter(row, column) {
      let xAddress = "";
      const province = row.address.indexOf("省");
      const city = row.address.indexOf("市");
      xAddress =
        row.address.substring(0, province + 1) +
        "  " +
        row.address.substring(province + 1, city + 1) +
        "  " +
        row.address.substr(city + 1, 100);
      return xAddress;
      //   const provinceCode = row.xProvinceCode;
      //   const cityCode = row.xCityCode;
      //   const regionCode = row.xRegionCode;
      //   const data = this.$qs.stringify({
      //     xProvinceCode: provinceCode,
      //     xCityCode: cityCode,
      //     xRegionCode: regionCode
      //   });
      //   const url = "http://localhost:8086/table/queryJoinAddress";
      //   let test = "1";
      //   this.$axios
      //     .post(url, data)
      //     .then(res => {
      //       console.log(res.data);
      //       test=res.data
      //     })
      //  return res.data;
    },
    formatterDate(row) {
      if (typeof row.date == "object") {
        return this.formatDateTime(row.date);
      } else {
        return row.date.substr(0, 10) + " " + row.date.substr(11, 8);
      }
    },
    filterTag(value, row) {
      return row.tag === value;
    },
    handleEdit(index, row) {
      this.idx = index;
      const item = this.tableData[index];
      this.form = {
        name: item.name,
        date: new Date(item.date),
        options: this.form.options,
        address: [
          JSON.parse(item.xProvinceCode),
          JSON.parse(item.xCityCode),
          JSON.parse(item.xRegionCode)
        ],
        id: item.id
      };
      this.editVisible = true;
    },
    handleAdd() {
      this.addVisible = true;
      this.form = {
        name: "",
        date: "",
        address: []
      };
    },
    handleDelete(index, row) {
      this.idx = row.id;
      this.delVisible = true;
    },
    // 确定删除
    deleteRow() {
      const datas = { array: [] };
      datas.array[0] = this.idx;
      console.log(datas);
      const url = "http://localhost:8086/table/handleDelete";
      this.$axios
        .post(url, datas)
        .then(res => {
          // this.tableData.splice(this.idx, 1);
          this.getData();
          this.$message.success("删除成功");
          this.delVisible = false;
        })
        .catch(res => {
          this.$message.error("系统异常，请联系管理员");
        });
    },
    handleDeleteAll() {
      const length = this.multipleSelection.length;
      if (length != 0) {
        this.multipleSelectionTemp = this.multipleSelection;
        this.delAllVisible = true;
      } else {
        this.$message.warning("请选择需要删除的数据");
      }
    },
    delAll() {
      let str = "";
      this.del_list = this.del_list.concat(this.multipleSelectionTemp);
      const datas = { array: [] };
      for (let i = 0; i < this.del_list.length; i++) {
        datas.array[i] = this.del_list[i].id;
      }
      console.log(datas);
      const url = "http://localhost:8086/table/handleDelete";
      this.$axios
        .post(url, datas)
        .then(res => {
          const length = this.del_list.length;
          for (let i = 0; i < length; i++) {
            str += this.multipleSelectionTemp[i].name + " ";
          }
          this.$message.error("删除了" + str);
          this.getData();
          this.del_list = [];
          this.multipleSelection = [];
          this.multipleSelectionTemp = [];
          this.delAllVisible = false;
        })
        .catch(res => {
          this.$message.error("系统异常，请联系管理员");
        });
    },
    handleSelectionChange(val) {
      this.multipleSelection = val;
    },
    // 保存编辑
    saveEdit() {
      if (this.form.date == null) {
        this.$message.warning("请选择日期");
        return;
      } else if (this.form.name == "") {
        this.$message.warning("请输入姓名");
        return;
      } else {
        const date = this.formatDateTime(this.form.date);
        const data = this.$qs.stringify({
          id: this.form.id,
          xDate: date,
          xName: this.form.name,
          xProvinceCode: this.form.address[0],
          xCityCode: this.form.address[1],
          xRegionCode: this.form.address[2]
        });
        const url = "http://localhost:8086/table/updateTableMsg";
        this.$axios
          .post(url, data)
          .then(res => {
            this.getData();
            this.editVisible = false;
            this.$message.success(`修改第 ${this.idx + 1} 行成功`);
          })
          .catch(res => {
            this.$message.error("系统异常，请联系管理员");
          });
      }
      //  this.$set(this.tableData, this.idx, this.form);
    },

    // 保存添加
    saveAdd() {
      if (this.form.date == null) {
        this.$message.warning("请选择日期");
        return;
      } else if (this.form.name == "") {
        this.$message.warning("请输入姓名");
        return;
      } else if (this.form.address == null || this.form.address.length == 0) {
        this.$message.warning("请选择所在地区");
        return;
      } else {
        const date = this.formatDateTime(this.form.date);
        const data = this.$qs.stringify({
          id: this.form.id,
          xDate: date,
          xName: this.form.name,
          xProvinceCode: this.form.address[0],
          xCityCode: this.form.address[1],
          xRegionCode: this.form.address[2]
        });
        const url = "http://localhost:8086/table/addTableMsg";
        this.$axios
          .post(url, data)
          .then(res => {
            this.addVisible = false;
            this.$message.success(`添加成功`);
            this.getData();
          })
          .catch(res => {
            this.$message.error("系统异常，请联系管理员");
          });
      }
      //  this.$set(this.tableData, this.idx, this.form);
    },
    formatDateTime(oldDate) {
      let date = oldDate;
      let y = date.getFullYear();
      let MM = date.getMonth() + 1;
      MM = MM < 10 ? "0" + MM : MM;
      let d = date.getDate();
      d = d < 10 ? "0" + d : d;
      let h = date.getHours();
      h = h < 10 ? "0" + h : h;
      let m = date.getMinutes();
      m = m < 10 ? "0" + m : m;
      let s = date.getSeconds();
      s = s < 10 ? "0" + s : s;
      date = y + "-" + MM + "-" + d + " " + h + ":" + m + ":" + s;
      return date;
    }
  }
};
</script>

<style scoped>
.handle-box {
  margin-bottom: 20px;
}

.handle-select {
  width: 120px;
}

.handle-input {
  width: 200px;
  margin-right: 10px;
  display: inline-block;
}
.handle-add {
  float: right;
  width: 120px;
}
.del-dialog-cnt {
  font-size: 16px;
  text-align: center;
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
.mr20 {
  margin-right: 10px;
  width: auto;
}
</style>
