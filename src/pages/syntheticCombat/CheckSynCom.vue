<template>
  <div class="big-box">
    <div class="header-container">
      <el-row :gutter="20" class="el-row-style">
        <el-col :span="6" class="el-col-style">
          <div class="grid-content">
            <span class="header-title">合成作战审核</span>
            <a class="header-back" href="javascript:;" @click="$router.go(-1)">返回</a>
          </div>
        </el-col>
        <el-col :span="8" :offset="10">
          <div class="grid-content">
            <el-row class="el-row-right">
              <el-button type="primary" size="small" v-show="!isBack & btnShowFlag & isOnlyRead" @click="submit('instructForm')">提交</el-button>
              <el-button type="warning" size="small" v-show="isBack & btnShowFlag & isOnlyRead" @click="submit('instructForm')">退回给申请人</el-button>
              <el-button size="small" @click="checkProcess">查看流程</el-button>
            </el-row>
          </div>
        </el-col>
      </el-row>

      <instructions :infos="infos"></instructions>

      <el-form
        :inline="true"
        :model="instructForm"
        :rules="instructRules"
        ref="instructForm"
        class="instruct-form form-style"
        label-width="100px"
      >
        <div class="form-item" v-if="btnShowFlag & isOnlyRead">
          <div class="form-title">审核信息</div>
          <el-form-item label="审核人：" prop="createName">
            <div>{{instructForm.createName}}</div>
          </el-form-item>
          <el-form-item label="审核结果：" prop="result">
            <el-radio-group v-model="instructForm.result">
              <el-radio label="通过" @change="resultRadio"></el-radio>
              <el-radio label="退回" @change="resultRadio"></el-radio>
            </el-radio-group>
          </el-form-item>
          <!-- 占位 -->
          <el-form-item></el-form-item>

          <el-form-item label="审核意见：" prop="suggest" class="brief-case-content">
            <el-input
              type="textarea"
              :rows="4"
              placeholder="请输入审核意见"
              v-model="instructForm.suggest"
            ></el-input>
          </el-form-item>
        </div>

        <div class="form-btn-item" v-if="btnShowFlag & isOnlyRead">
          <el-form-item>
            <el-button type="primary" size="small" v-show="!isBack" @click="submit('instructForm')">提交</el-button>
            <el-button type="warning" size="small" v-show="isBack" @click="submit('instructForm')">退回给申请人</el-button>
          </el-form-item>
        </div>
      </el-form>
    </div>
    <!-- 流程弹窗 -->
    <div class="process-box" v-show="processIsShow">
      <div class="process-border" id="mountNode">
        <!-- 流程图 -->
      </div>
    </div>
  </div>
</template>

<script>
//  引入流程图插件 AntV/g6
import G6 from "@antv/g6";
//  引入流程图节点图片
import processNodeDefault from "../../../static/image/processNodeDefault.png";
import processNodeSuccess from "../../../static/image/processNodeSuccess.png";
import UtilService from '../../service/UtilService.js'

//  引入通用'指令'组件
import instructions from "../../components/SCInstructions.vue";
import { type } from 'os';
import { fail } from 'assert';

export default {
  name: 'checkSynCom',
  data() {
    return {
      url: {
        getInformationUrl: "/hcfight/cmdDetails",
        startCheckUrl: "/hcfight/fightStartDeptReview"
      },
      params: {
          id: '',
          createBy: ""
      },
      processIsShow: false, //  流程弹框显示与否
      accessoryPlacrholder: "请上传案情材料，案情列表，物证详情等",
      isBack: false, //  是否退回

      instructForm: {
        createName: "",
        result: "通过",
        suggest: ""
      },
      instructRules: {
        result: [
            { required: true, message: "请选择审核结果", trigger: "blur" }
        ],
        suggest: [
          { required: true, message: "请输入审核意见", trigger: "blur" },
          { min: 1, max: 200, message: "请输入1-200个字符", trigger: "blur" }
        ]
      },
      infos: {},
      jizhen: '0',
      wangzhen: '0',
      shipin: '0',
      btnShowFlag: false,
      isOnlyRead: false,
      deptType: '',
      taskId: '', // 退回id
      nodesData: [
        //  流程图节点数据
        {
          id: "node1",
          x: 265,
          y: 208,
          size: [40, 40],
          shape: "image",
          img: processNodeSuccess,
          label: `
                    申请
                    张三  发起申请
                    2019-04-09 17:10:20`,
          labelCfg: {
            position: "bottom",
            textAlign: "left"
          }
        },
        {
          id: "node2",
          x: 423,
          y: 208,
          size: [40, 40],
          shape: "image",
          img: processNodeDefault,
          label: `
                    发起部门领导审批
                    李四  待处理
                    2019-04-09 17:10:20`,
          labelCfg: {
            position: "bottom",
            textAlign: "left"
          }
        },
        {
          id: "node3",
          x: 670,
          y: 67,
          size: [40, 40],
          shape: "image",
          img: processNodeDefault,
          label: `
                    发起部门领导审批
                    李四  待处理
                    2019-04-09 17:10:20`,
          labelCfg: {
            position: "bottom",
            textAlign: "left"
          }
        },
        {
          id: "node4",
          x: 670,
          y: 208,
          size: [40, 40],
          shape: "image",
          img: processNodeDefault,
          label: `
                    发起部门领导审批
                    李四  待处理
                    2019-04-09 17:10:20`,
          labelCfg: {
            position: "bottom",
            textAlign: "left"
          }
        },
        {
          id: "node5",
          x: 670,
          y: 357,
          size: [40, 40],
          shape: "image",
          img: processNodeDefault,
          label: `
                    发起部门领导审批
                    李四  待处理
                    2019-04-09 17:10:20`,
          labelCfg: {
            position: "bottom",
            textAlign: "left"
          }
        },
        {
          id: "node6",
          x: 830,
          y: 67,
          size: [40, 40],
          shape: "image",
          img: processNodeDefault,
          label: `
                    发起部门领导审批
                    李四  待处理
                    2019-04-09 17:10:20`,
          labelCfg: {
            position: "bottom",
            textAlign: "left"
          }
        },
        {
          id: "node7",
          x: 830,
          y: 208,
          size: [40, 40],
          shape: "image",
          img: processNodeDefault,
          label: `
                    发起部门领导审批
                    李四  待处理
                    2019-04-09 17:10:20`,
          labelCfg: {
            position: "bottom",
            textAlign: "left"
          }
        },
        {
          id: "node8",
          x: 830,
          y: 357,
          size: [40, 40],
          shape: "image",
          img: processNodeDefault,
          label: `
                    发起部门领导审批
                    李四  待处理
                    2019-04-09 17:10:20`,
          labelCfg: {
            position: "bottom",
            textAlign: "left"
          }
        },
        {
          id: "node9",
          x: 990,
          y: 67,
          size: [40, 40],
          shape: "image",
          img: processNodeDefault,
          label: `
                    发起部门领导审批
                    李四  待处理
                    2019-04-09 17:10:20`,
          labelCfg: {
            position: "bottom",
            textAlign: "left"
          }
        },
        {
          id: "node10",
          x: 990,
          y: 208,
          size: [40, 40],
          shape: "image",
          img: processNodeDefault,
          label: `
                    发起部门领导审批
                    李四  待处理
                    2019-04-09 17:10:20`,
          labelCfg: {
            position: "bottom",
            textAlign: "left"
          }
        },
        {
          id: "node11",
          x: 990,
          y: 357,
          size: [40, 40],
          shape: "image",
          img: processNodeDefault,
          label: `
                    发起部门领导审批
                    李四  待处理
                    2019-04-09 17:10:20`,
          labelCfg: {
            position: "bottom",
            textAlign: "left"
          }
        }
      ]
    };
  },
  created() {
    if (localStorage.getItem('userName')) {
      this.params.createBy = localStorage.getItem('userName')
    }
    if (localStorage.getItem('realName')) {
      this.instructForm.createName = localStorage.getItem('realName')
    }
    if (this.$route.query.id) {
        this.params.id = this.$route.query.id
    }
    this.taskId = this.$route.query.taskId
    this.deptType = this.$route.query.deptType
    if (this.$route.query.statusType) {
        if (this.$route.query.statusType === '1' || this.$route.query.statusType === '2') {
          this.isOnlyRead = false
        } else {
          this.isOnlyRead = true
        }
    }
    this.getInformationData();
  },
  mounted() {
    //  显示流程图
    const data = {
      //  节点
      nodes: this.nodesData,
      //  连接线
      edges: [
        {
          source: "node1",
          target: "node2",
          style: {
            endArrow: true
          }
        },
        {
          source: "node2",
          target: "node3",
          shape: "polyline", //  折线
          controlPoints: [{ x: 423, y: 67 }], //  折线转折处坐标
          style: {
            endArrow: true //  连接线结尾箭头
          }
        },
        {
          source: "node2",
          target: "node4",
          style: {
            endArrow: true
          }
        },
        {
          source: "node2",
          target: "node5",
          shape: "polyline",
          controlPoints: [{ x: 423, y: 357 }],
          style: {
            endArrow: true
          }
        },
        {
          source: "node3",
          target: "node6",
          style: {
            endArrow: true
          }
        },
        {
          source: "node4",
          target: "node7",
          style: {
            endArrow: true
          }
        },
        {
          source: "node5",
          target: "node8",
          style: {
            endArrow: true
          }
        },
        {
          source: "node6",
          target: "node9",
          style: {
            endArrow: true
          }
        },
        {
          source: "node7",
          target: "node10",
          style: {
            endArrow: true
          }
        },
        {
          source: "node8",
          target: "node11",
          style: {
            endArrow: true
          }
        }
      ]
    };
    const graph = new G6.Graph({
      container: "mountNode",
      width: 1306,
      height: 480,
      nodeStyle: {
        default: {
          fill: "#f7faff", //  节点背景色
          stroke: "#c0ccd7" //  节点边框色
        }
      },
      edgeStyle: {
        default: {
          stroke: "#bfcbd6", //  连接线颜色
          lineWidth: 2
        }
      }
    });
    graph.read(data);
  },
  components: {
    instructions
  },
  methods: {
    // 获取信息数据
    getInformationData() {
      let that = this;
      this.$Ajax.get(that.url.getInformationUrl, that.params, true).then(data => {
        that.infos = data.data.cmdDetails;
        that.infos.coprDepts.forEach((item, index) => {
          that.$set(item, "showFlag", false);
          if (index === 0) {
            that.$set(item, "showFlag", true);
          }
          if(item.deptType === 0) {
              that.jizhen = '1'
          } 
          if(item.deptType === 1) {
              that.wangzhen = '1'
          } 
          if(item.deptType === 2) {
              that.shipin = '1'
          } 
        });
        if (data.data.btn.length > 0) {
            data.data.btn.forEach(item => {
              if (item.taskDefKey === that.$route.query.deptType) {
                that.btnShowFlag = true;
              }
            });
        }
        console.log("是数据啊", that.infos.coprDepts);
      });
    },
    // 提交
    submit(formName) {
      this.$refs[formName].validate((valid) => {
        if (valid) {
          this.startCheck()
        } else {
          console.log('error submit!!')
          return false
        }
      })
    },
    // 发起部门审核
    startCheck() {
      let that = this;
      let params = {
        createBy: that.params.createBy,
        id: that.params.id,
        statusId: that.deptType,
        taskId: that.taskId,
        opinionInfo: {
          deptTypeName: that.deptType,
          createName: that.instructForm.createName,
          suggest: that.instructForm.suggest,
          result: that.instructForm.result,
          jizhen: that.jizhen,
          wangzhen: that.wangzhen,
          shipin: that.shipin
        }
      };
      let successMsg = '部门审核成功'
      let failMsg = '部门审核失败'
      if (that.instructForm.result !== '通过') {
        successMsg = '退回成功'
        failMsg = '退回失败'
      }
      this.$Ajax.post(that.url.startCheckUrl, params, true).then(data => {
        if (data.data === "success") {
          that.$message({
            message: successMsg,
            type: "success"
          });
          that.$router.push({
              name: 'hczzManage'
          })
        } else {
          that.$message({
            message: failMsg,
            type: "warning"
          });
        }
      });
    },
    //  查看流程
    checkProcess() {
      this.processIsShow = !this.processIsShow;
      // console.log(this.instructForm)
    },
    // 结果切换
    resultRadio(e) {
      console.log(e);
      e == "退回" ? (this.isBack = true) : (this.isBack = false);
    }
  }
};
</script>

<style scoped lang="less">
@import "../../../static/css/CheckIns.less";
</style>
