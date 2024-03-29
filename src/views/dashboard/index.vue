<template>
  <div class="dashboard-container">
    <!-- 头部内容 -->
    <el-card class="header-card">
      <div>
        <div class="fl headL">
          <div class="headImg">
            <img v-imageerror="defaultImg" :src="staffPhoto" />
          </div>
          <div class="headInfoTip">
            <p class="firstChild">你好，{{ name }}</p>
            <p class="lastChild">
              {{ userInfo.departmentName }}
            </p>
          </div>
        </div>
        <div class="fr" />
      </div>
    </el-card>
    <!-- 主要内容 -->
    <el-row type="flex" justify="space-between">
      <!-- 左侧内容 -->
      <el-col :span="13" style="padding-right: 26px">
        <!-- 工作日历 -->
        <el-card class="box-card">
          <div slot="header" class="header headTit">
            <span>工作日历</span>
          </div>
          <!-- 放置日历组件 -->
          <WorkCalendar />
        </el-card>
        <el-card class="box-card">
          <div slot="header" class="header headTit">
            <span>流程申请</span>
          </div>
          <div class="sideNav">
            <el-button class="sideBtn" @click="$router.push('/attendances')"
              >考勤列表</el-button
            ><el-button class="sideBtn" @click="$router.push('/permission')"
              >权限申请</el-button
            >
            <el-button class="sideBtn" @click="$router.push('/users/approvals')"
              >审批列表</el-button
            >
            <el-button class="sideBtn" @click="showDialog = true"
              >加班离职</el-button
            >
          </div>
        </el-card>
      </el-col>
      <!-- 右侧内容 -->
      <el-col :span="11">
        <!-- 帮助连接 -->
        <el-card class="box-card">
          <div slot="header" class="header headTit">
            <span>流程进度</span>
          </div>
          <div>
            <el-steps
              :space="200"
              :active="1"
              finish-status="success"
              align-center
            >
              <el-step title="已完成" />
              <el-step title="进行中" />
              <el-step title="步骤 3" />
            </el-steps>
          </div>
        </el-card>
        <!-- 绩效指数 -->
        <el-card class="box-card">
          <div slot="header" class="header headTit">
            <span>绩效指数</span>
          </div>
          <!-- 放置雷达图 -->
          <Radar />
        </el-card>
      </el-col>
    </el-row>
    <!-- 弹出层 -->
    <el-dialog :visible="showDialog" title="离职申请" @close="btnCancel">
      <el-form
        ref="ruleForm"
        :model="ruleForm"
        status-icon
        label-width="110px"
        :rules="rules"
      >
        <!--离职表单-->
        <el-form-item label="离职时间" prop="exceptTime">
          <el-date-picker
            v-model="ruleForm.exceptTime"
            type="datetime"
            value-format="yyyy-MM-dd HH:mm:ss"
            placeholder="选择日期时间"
          />
        </el-form-item>
        <el-form-item label="离职原因" prop="reason">
          <el-input
            v-model="ruleForm.reason"
            type="textarea"
            :autosize="{ minRows: 3, maxRows: 8 }"
            style="width: 70%"
            placeholder="请输入内容"
          />
        </el-form-item>
      </el-form>
      <el-row slot="footer" type="flex" justify="center">
        <el-col :span="6">
          <el-button size="small" type="primary" @click="btnOK">确定</el-button>
          <el-button size="small" @click="btnCancel">取消</el-button>
        </el-col>
      </el-row>
    </el-dialog>
  </div>
</template>

<script>
import { mapGetters, createNamespacedHelpers } from 'vuex'
import WorkCalendar from './components/work-calendar.vue'
import Radar from './components/radar.vue'
import { startProcess } from '@/api/approvals'

const { mapState } = createNamespacedHelpers('user')

export default {
  name: 'Dashboard',
  components: {
    WorkCalendar,
    Radar
  },
  data () {
    return {
      ruleForm: {
        exceptTime: '',
        reason: '',
        processKey: 'process_dimission', // 特定的审批
        processName: '离职'
      },
      rules: {
        exceptTime: [{ required: true, message: '离职时间不能为空', trigger: 'blur' }],
        reason: [{ required: true, message: '离职原因不能为空', trigger: 'blur' }]
      },
      defaultImg: require('@/assets/common/bigUserHeader.png'),
      showDialog: false
    }
  },
  computed: {
    ...mapGetters([
      'name',
      'staffPhoto'
    ]),
    ...mapState(['userInfo'])
  },
  methods: {
    btnOK () {
      this.$refs.ruleForm.validate(async validate => {
        if (validate) {
          const data = { ...this.ruleForm, userId: this.userInfo.userId }
          await startProcess(data)
          this.$message.success('提交流程成功')
          this.showDialog = false
        }
      })
    },
    btnCancel () {
      this.ruleForm = {
        exceptTime: '',
        reason: '',
        processKey: 'process_dimission', // 特定的审批
        processName: '离职'
      }
      this.$refs.ruleForm.resetFields()
      this.showDialog = false
    }
  }
}
</script>

<style lang="scss" scoped>
.dashboard-container {
  margin: 10px;
  li {
    list-style: none;
  }
  .headImg {
    float: left;
    width: 100px;
    height: 100px;
    border-radius: 50%;
    background: #999;
    img {
      width: 100%;
      height: 100%;
      border-radius: 50%;
    }
  }

  .headInfoTip {
    padding: 25px 0 0;
    margin-left: 120px;
    p {
      padding: 0 0 15px;
      margin: 0;
      &.firstChild {
        font-size: 24px;
      }
      &.lastChild {
        font-size: 20px;
        color: #7f8c8d;
      }
    }
  }
}

.box-card {
  padding: 5px 10px;
  margin-top: 20px;
  .header {
    span {
      color: #2c3e50;
      font-size: 24px;
    }
    .item {
      color: #97a8be;
      float: right;
      padding: 3px 0;
    }
  }
  .headTit {
    span {
      border-bottom: 4px solid #409eff;
      padding-bottom: 10px;
    }
  }
}
.header-card {
  position: relative;
  .header {
    position: absolute;
    right: 20px;
    top: 15px;
    z-index: 1;
  }
}

.advContent {
  background: #fff;
  border-radius: 5px 5px 0px 0px;
  .title {
    font-size: 16px;
    padding: 20px;
    font-weight: bold;
    border-bottom: solid 1px #ccc;
  }
  .contentItem {
    padding: 0 30px;
    min-height: 350px;
    .item {
      display: flex;
      padding: 18px 0 10px;
      border-bottom: solid 1px #ccc;
      .col {
        color: #409eff;
      }
      img {
        width: 56px;
        height: 56px;
        border-radius: 50%;
        margin-right: 10px;
      }
      p {
        padding: 0 0 8px;
      }
    }
  }
}
.noticeList {
  margin: 0;
  padding: 0;
}
.sideNav,
.sideLink {
  padding: 30px 0 12px;
  .sideBtn {
    padding: 16px 26px;
    font-size: 16px;
    margin: 10px 5px;
  }
}
.sideLink {
  text-align: center;
  .icon {
    display: inline-block;
    width: 76px;
    height: 76px;
    // background: url('./../../assets/common/icon.png') no-repeat;
  }
  .iconGuide {
    background-position: 0 0;
  }
  .iconHelp {
    background-position: -224px 0;
  }
  .iconTechnology {
    background-position: -460px 0;
  }
}
</style>
