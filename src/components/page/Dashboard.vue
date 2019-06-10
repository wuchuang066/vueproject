<template>
  <div>
    <el-row :gutter="20">
      <el-col :span="8">
        <el-card shadow="hover" class="mgb20" style="height:252px;">
          <div class="user-info">
            <img src="../../assets/img/img.jpg" class="user-avator" alt @click="changePic()">
            <div class="user-info-cont">
              <div class="user-info-name">{{name}}</div>
              <div>{{role}}</div>
            </div>
          </div>
          <div class="user-info-list">
            上次登录时间：
            <span>{{loginDate.replace("T"," ").substring(0,19)}}</span>
          </div>
          <div class="user-info-list">
            上次登录地点：
            <span>未知</span>
          </div>
        </el-card>
        <el-card shadow="hover" style="height:252px;">
          <div slot="header" class="clearfix">
            <span>语言详情</span>
          </div>Vue
          <el-progress :percentage="system.xVue" color="#42b983"></el-progress>JavaScript
          <el-progress :percentage="system.xJs" color="#f1e05a"></el-progress>CSS
          <el-progress :percentage="system.xCss"></el-progress>HTML
          <el-progress :percentage="system.xHtml" color="#f56c6c"></el-progress>
        </el-card>
      </el-col>
      <el-col :span="16">
        <el-row :gutter="20" class="mgb20">
          <el-col :span="8">
            <el-card shadow="hover" :body-style="{padding: '0px'}">
              <div class="grid-content grid-con-1">
                <i class="el-icon-lx-people grid-con-icon"></i>
                <div class="grid-cont-right">
                  <div class="grid-num">{{system.user_count}}</div>
                  <div>用户访问量</div>
                </div>
              </div>
            </el-card>
          </el-col>
          <el-col :span="8">
            <el-card shadow="hover" :body-style="{padding: '0px'}">
              <div class="grid-content grid-con-2">
                <i class="el-icon-lx-notice grid-con-icon"></i>
                <div class="grid-cont-right">
                  <div class="grid-num">{{system.showMessage}}</div>
                  <div>系统消息</div>
                </div>
              </div>
            </el-card>
          </el-col>
          <el-col :span="8">
            <el-card shadow="hover" :body-style="{padding: '0px'}">
              <div class="grid-content grid-con-3">
                <i class="el-icon-lx-goods grid-con-icon"></i>
                <div class="grid-cont-right">
                  <div class="grid-num">{{system.numbers}}</div>
                  <div>数量</div>
                </div>
              </div>
            </el-card>
          </el-col>
        </el-row>
        <el-card shadow="hover" style="height:403px;">
          <div slot="header" class="clearfix">
            <span>待办事项</span>
            <el-button style="float: right; padding: 3px 0" type="text">添加</el-button>
          </div>
          <el-table
            :data="todoList"
            :show-header="false"
            height="304"
            style="width: 100%;font-size:14px;"
          >
            <el-table-column width="40">
              <template slot-scope="scope">
                <el-checkbox v-model="scope.row.status" @change="updateStatus(scope.row)"></el-checkbox>
              </template>
            </el-table-column>
            <el-table-column>
              <template slot-scope="scope">
                <div
                  class="todo-item"
                  :class="{'todo-item-del': scope.row.status}"
                >{{scope.row.title}}</div>
              </template>
            </el-table-column>
            <el-table-column width="60">
              <template slot-scope="scope">
                <i class="el-icon-edit"></i>
                <i class="el-icon-delete"></i>
              </template>
            </el-table-column>
          </el-table>
        </el-card>
      </el-col>
    </el-row>
    <el-row :gutter="20">
      <el-col :span="12">
        <el-card shadow="hover">
          <schart
            ref="bar"
            class="schart"
            canvasId="bar"
            :data="data"
            type="bar"
            :options="options"
          ></schart>
        </el-card>
      </el-col>
      <el-col :span="12">
        <el-card shadow="hover">
          <schart
            ref="line"
            class="schart"
            canvasId="line"
            :data="data"
            type="line"
            :options="options2"
          ></schart>
        </el-card>
      </el-col>
    </el-row>
  </div>
</template>

<script>
import Schart from "vue-schart";
import bus from "../common/bus";
export default {
  name: "dashboard",
  data() {
    return {
      test: 123.0,
      name: localStorage.getItem("ms_username"),
      loginDate: localStorage.getItem("ms_loginDate"),
      system: {
        xVue: 0,
        xJs: 0,
        xCss: 0,
        xHtml: 0,
        user_count: "",
        showMessage: "",
        numbers: ""
      },
      todoList: [],
      data: [
        {
          name: "2018/09/04",
          value: 1083
        },
        {
          name: "2018/09/05",
          value: 941
        },
        {
          name: "2018/09/06",
          value: 1139
        },
        {
          name: "2018/09/07",
          value: 816
        },
        {
          name: "2018/09/08",
          value: 327
        },
        {
          name: "2018/09/09",
          value: 228
        },
        {
          name: "2018/09/10",
          value: 1065
        }
      ],
      options: {
        title: "最近七天每天的用户访问量",
        showValue: false,
        fillColor: "rgb(45, 140, 240)",
        bottomPadding: 30,
        topPadding: 30
      },
      options2: {
        title: "最近七天用户访问趋势",
        fillColor: "#FC6FA1",
        axisColor: "#008ACD",
        contentColor: "#EEEEEE",
        bgColor: "#F5F8FD",
        bottomPadding: 30,
        topPadding: 30
      }
    };
  },
  components: {
    Schart
  },
  computed: {
    role() {
      return this.name === "admin" ? "超级管理员" : "普通用户";
    }
  },
  created() {
    this.queryTodoList();
    this.queryData();
    this.handleListener();
  },
  beforeUpdate() {
    this.changeDate();
  },
  activated() {
    this.handleListener();
  },
  deactivated() {
    window.removeEventListener("resize", this.renderChart);
    bus.$off("collapse", this.handleBus);
  },
  methods: {
    updateStatus(request) {
      const updateUrl = "http://localhost:8086/message/updateTodo";
      const data = this.$qs.stringify({
        id: request.id,
        status: request.status ? 0 : 1
      });
      this.$ajax.post(updateUrl, data).catch(response => {
        console.log(response);
        this.$router.push("/404");
      });
    },
    changeDate() {
      const now = new Date().getTime();
      this.data.forEach((item, index) => {
        const date = new Date(now - (6 - index) * 86400000);
        item.name = `${date.getFullYear()}/${date.getMonth() +
          1}/${date.getDate()}`;
      });
    },
    handleListener() {
      bus.$on("collapse", this.handleBus);
      // 调用renderChart方法对图表进行重新渲染
      window.addEventListener("resize", this.renderChart);
    },
    handleBus(msg) {
      setTimeout(() => {
        this.renderChart();
      }, 300);
    },
    renderChart() {
      this.$refs.bar.renderChart();
      this.$refs.line.renderChart();
    },
    changePic() {
      alert("上传图片待开发");
    },
    queryData() {
      const url = "http://localhost:8086/message/queryMessage";
      const data = this.$qs.stringify({
        xUserid: localStorage.getItem("ms_userid")
      });
      this.$ajax
        .post(url, data)
        .then(response => {
          console.log(response);
          this.system.user_count = response.data.xVisitCount;
          this.system.showMessage = response.data.xSysMessage;
          this.system.numbers = response.data.xCount;
          this.system.xVue = response.data.xVue;
          this.system.xJs = response.data.xJs;
          this.system.xCss = response.data.xCss;
          this.system.xHtml = response.data.xHtml;
          const vList = response.data.xVisits;
          if (vList != null && vList.length > 0) {
            for (let i = 0; i < vList.length; i++) {
              vList[i].name = vList[i].name.substring(0, 10);
            }
          }
          this.data = response.data.xVisits;
        })
        .catch(response => {
          alert("系统异常，请联系管理员");
          this.$router.push("/404");
        });
    },
    queryTodoList() {
      const url = "http://localhost:8086/message/queryTodoList";
      const data = this.$qs.stringify({
        xUserid: localStorage.getItem("ms_userid")
      });
      this.$ajax
        .post(url, data)
        .then(response => {
          const list = response.data;
          if (list != null && list.length > 0) {
            for (var i = 0; i < list.length; i++) {
              if (list[i].status == 0) {
                list[i].status = true;
              } else {
                list[i].status = false;
              }
            }
          }
          this.todoList = response.data;
        })
        .catch(response => {
          alert("系统异常，请联系管理员");
          this.$router.push("/404");
        });
    }
  }
};
</script>


<style scoped>
.el-row {
  margin-bottom: 20px;
}

.grid-content {
  display: flex;
  align-items: center;
  height: 100px;
}

.grid-cont-right {
  flex: 1;
  text-align: center;
  font-size: 14px;
  color: #999;
}

.grid-num {
  font-size: 30px;
  font-weight: bold;
}

.grid-con-icon {
  font-size: 50px;
  width: 100px;
  height: 100px;
  text-align: center;
  line-height: 100px;
  color: #fff;
}

.grid-con-1 .grid-con-icon {
  background: rgb(45, 140, 240);
}

.grid-con-1 .grid-num {
  color: rgb(45, 140, 240);
}

.grid-con-2 .grid-con-icon {
  background: rgb(100, 213, 114);
}

.grid-con-2 .grid-num {
  color: rgb(45, 140, 240);
}

.grid-con-3 .grid-con-icon {
  background: rgb(242, 94, 67);
}

.grid-con-3 .grid-num {
  color: rgb(242, 94, 67);
}

.user-info {
  display: flex;
  align-items: center;
  padding-bottom: 20px;
  border-bottom: 2px solid #ccc;
  margin-bottom: 20px;
}

.user-avator {
  width: 120px;
  height: 120px;
  border-radius: 50%;
}

.user-info-cont {
  padding-left: 50px;
  flex: 1;
  font-size: 14px;
  color: #999;
}

.user-info-cont div:first-child {
  font-size: 30px;
  color: #222;
}

.user-info-list {
  font-size: 14px;
  color: #999;
  line-height: 25px;
}

.user-info-list span {
  margin-left: 70px;
}

.mgb20 {
  margin-bottom: 20px;
}

.todo-item {
  font-size: 14px;
}

.todo-item-del {
  text-decoration: line-through;
  color: #999;
}

.schart {
  width: 100%;
  height: 300px;
}
</style>
