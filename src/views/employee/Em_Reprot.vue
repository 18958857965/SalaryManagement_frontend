<template>
  <div class="border">

    <div>
      <el-select v-model="reportType" placeholder="请选择需要的报告类型" class="inputInfo" style="margin-right: 100px"
                 @change="empPid">
        <el-option
            v-for="item in options"
            :key="item.value"
            :label="item.label"
            :value="item.value">
        </el-option>
      </el-select>
      <el-date-picker
          class="inputInfo"
          v-model="send.startTime"
          type="datetime"
          placeholder="选择开始日期时间"
          value-format="yyyy-MM-dd HH:mm:ss">
      </el-date-picker>
      <span style="margin-left: 10px;margin-right: 10px">至</span>
      <el-date-picker
          class="inputInfo"
          v-model="send.endTime"
          type="datetime"
          placeholder="选择截止日期时间"
          value-format="yyyy-MM-dd HH:mm:ss"
      >
      </el-date-picker>
      <div v-if="reportType===1" class="inputInfo" style="margin-top: 5px;">
<!--        <el-input placeholder="请输入项目id" clearable v-model="send.pid" size="small" style="width: 220px"></el-input>-->
          <el-select v-model="send.pid" @change="showProject" placeholder="请选择项目" filterable class="inputInfo"
                     style="margin-right: 100px" size="small" clearable>
            <el-option
                v-for="item in project"
                :key="item.id"
                :label="item.name"
                :value="item.id"
            >
              <span style=" color: #8492a6; font-size: 13px">{{ item.name }}</span>
            </el-option>
          </el-select>


      </div>
    </div>
    <div>
      <img :src="this.src" alt="">
    </div>
    <div>
      <el-row>
        <el-col :span="4">
          <div class="grid-content"></div>
        </el-col>
        <el-col :span="5">
          <div class="grid-content"></div>
        </el-col>
        <el-col :span="5">
          <div class="grid-content"></div>
        </el-col>
        <el-col :span="5">
          <div class="grid-content"></div>
        </el-col>
        <el-col :span="5">
          <el-button round @click="renderReport">确认生成报告</el-button>
<!--          <el-button type="primary" icon="el-icon-download" circle @click="download"></el-button>
          <el-button type="success" icon="el-icon-share" circle @click="share"></el-button>-->
        </el-col>
      </el-row>
    </div>
  </div>
</template>
<script>
export default {
  name: "Em_Reprot",
  data() {
    return {
      project:[],
      src: '',
      reportType: '',
      send: {
        uid:this.$store.state.employee.uid,
        startTime: '',
        endTime: '',
        pid: ''
      },
      options: [
        {
          label: '工作小时数',
          value: 0
        },
        {
          label: '项目总工时',
          value: 1
        },
        {
          label: '休假',
          value: 2
        },
        {
          label: '总收入',
          value: 3
        }
      ]

    }

  },
  methods: {
    showProject() {
      this.$notify.closeAll();
      this.postRequest('/project/get?cid=' + this.$store.state.company.cid + '&pid=' + this.send.pid).then(async (resp) => {
        if (resp) {
          let data = ['项目编号:' + resp.result.id,
            '项目名称:' + resp.result.name,
            '项目描述:' + resp.result.description];
          const h = this.$createElement;
          await this.$notify({
            title: '项目信息',
            message: h('i', null, data
            )
          });

        }
      })


    },
    getProject() {
      this.postRequest('/project/getAll?cid=' + this.$store.state.company.cid).then(resp => {
        if (resp && resp.result.length && resp.result.length > 0) {
          this.project = resp.result;


        }
      })
    },
    empPid() {
      this.send.pid = '';

    },
    renderReport() {
      console.log(this.send.startTime && this.send.endTime && this.reportType!==null)
      if (this.send.startTime && this.send.endTime && this.reportType!==null) {

        if (this.reportType !== '项目总工时' || (this.reportType === '项目总工时' && this.send.pid)) {
          this.postRequest('/report/draw?cid=' + this.$store.state.company.cid + '&code=' + this.reportType,this.send).then(resp => {
            if (resp) {

              this.src = resp.result;
            } else {
              this.$message.error('填写的项目编号有误');

            }
          })
        } else {
          this.$message.error('请填写项目id');

        }

      } else {
        this.$message.error('请选择并填写完整信息');

      }


    },
    download() {

    },
    share() {

    },
  },
  mounted() {
    this.getProject();
  }

}
</script>

<style scoped>
.inputInfo {
  margin: 3px 6px 3px 6px;
}

.border {
  box-shadow: 0 2px 12px 0 rgba(0, 0, 0, 0.1);
  padding: 10px 10px 10px 10px;
  width: auto;
  height: 450px;
}

.grid-content {
  border-radius: 4px;
  min-height: 36px;
}

</style>