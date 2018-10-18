<template lang="html">
  <div class="content-page">
      <div class="content-nav" >
          <el-breadcrumb class="breadcrumb" separator="/">
              <el-breadcrumb-item :to="{ path: '/dashboard' }">首页</el-breadcrumb-item>
              <el-breadcrumb-item >用户管理</el-breadcrumb-item>
              <el-breadcrumb-item :to="{ path: '/dashboard/User/info/list' }">档案列表</el-breadcrumb-item>
              <el-breadcrumb-item>档案详情</el-breadcrumb-item>
          </el-breadcrumb>
          <div class="operation-nav">
              <el-button type="primary" @click="goBackPage" icon="arrow-left">返回列表</el-button>
          </div>
      </div>

      <div class="content-main">
<!-- ////////////////////////////////////////////////////////////////////////////////////////卡包基础信息开始 -->

        <el-form :model="ruleForm" ref="ruleForm" label-width="180px">
          <div class="tiptitle">
            基础信息
          </div>
          <el-form-item label="昵称" >
            <el-input style="width:280px;" placeholder="" v-model="ruleForm.name"></el-input>
          </el-form-item>
          <el-form-item label="称呼" >
            <el-input style="width:280px;" placeholder=""></el-input>
          </el-form-item>
          <el-form-item label="年龄" >
            <el-input style="width:280px;" placeholder="" v-model="ruleForm.age"></el-input>
          </el-form-item>
          <el-form-item label="职业" >
            <el-input style="width:280px;" placeholder="" v-model="ruleForm.occupation"></el-input>
          </el-form-item>
          <el-form-item label="身高" >
            <el-input style="width:280px;" placeholder="" v-model="ruleForm.height"></el-input>
          </el-form-item>
          <el-form-item label="体重" >
            <el-input style="width:280px;" placeholder="" v-model="ruleForm.weight"></el-input>
          </el-form-item>
          <el-form-item label="细节" >
            <el-input style="width:280px;" placeholder="" v-model="ruleForm.detail"></el-input>
          </el-form-item>
<!-- ////////////////////////////////////////////////////////////////////////////////////////基础信息结束 -->

          <div class="tiptitle" style="padding-top:10px;padding-bottom:10px;">
            颜色偏好
          </div>
          <el-form-item label="不喜欢的颜色" >
          </el-form-item>
          <div class="color-container" >
            <div class="color" v-for='item in ruleForm.colorDislikeArr' :key='item.color' :class='"color-" + item.value'>
            </div>
          </div>

          <!-- <el-form-item>
            <el-button type="primary" @click="submitForm('ruleForm')">{{is_edit ? '修改' : '立即创建'}}</el-button>
            <el-button @click="resetForm('ruleForm')" v-if="!is_edit">重置</el-button>
          </el-form-item> -->
        </el-form>

    </div>
  </div>
</template>

<script>
import { Toast } from "vant";
import api from "@/config/api";

export default {
  data() {
    return {
      couponpage: 1,
      coupontotal: 0,
      userpage: 1,
      usertotal: 0,
      cupfromradio: 1, //卡包形式默认单选按钮
      // cuplimtradio: 4, //使用门槛默认单选按钮
      cupableradio: 6, //有效期默认单选按钮
      cuppointradio: 10, //指定卡券默认单选按钮
      cupuserradio: 11, //指定用户默认单选按钮
      // cupusergetradio: 13,//是否需要用户主动领取默认值
      selectCouponPopover: false, //选择卡券的弹出层
      selectUserPopover: false, //选择用户的弹出层
      PointUserInput: "", //添加指定用户的输入框内容
      PointCouponInput: "", //添加指定卡券的输入框内容
      allCouponData: [], //数据库取出的五条数据
      allUserData: [], //数据库取出的五条数据
      dangerCouponData: [], //修改按钮之后的五条数据
      dangerUserData: [], //修改按钮之后的五条数据
      ruleForm: {
        name: "", //名称
        datelimit: "", //时间限制
        // datestart: "", //时间开始
        desc: "" //卡包使用说明
      },
      pointCouponList: [], //指定卡券可使用卡包列表
      pointUserList: [], //指定用户可使用卡包列表
      id: 0,
      is_edit: false,
      datepickerPlaceholder: "点击选择 开始日期 - 结束日期",
      typechangeTime: 0, //卡包形式改变过几次
      colors: [
        { color: '#000000', name: '黑色', value: '0' },
        { color: '#FFFFFF', name: '白色', value: '1'  },
        { color: '#C0C0C0', name: '灰色', value: '2'  },
        { color: '#082E54', name: '藏青色', value: '3'  },
        { color: '#A39480', name: '米色', value: '4'  },
        { color: '#8B4513', name: '棕色', value: '5'  },
        { color: '#F8F8FF', name: '米白色', value: '6'  },
        { color: '#FFFF00', name: '黄色', value: '7'  },
        { color: '#6B8E23', name: '军绿色', value: '8'  },
        { color: '#4169E1', name: '深蓝色', value: '9'  },
        { color: '#FF0000', name: '红色', value: '10'  },
        { color: '#9013FE', name: '紫色', value: '11'  }
      ],
    };
  },
  mounted() {
    this.id = this.$route.query.id || 0;
    if (this.id !== 0) {
      this.is_edit = true;
      this.getInfo();
    }
  },
  updated() {},
  methods: {
    ////////////////////////////////////////////////编辑时按id查找信息
    getInfo() {
      this.axios
        .post("userinfo/getById", {
          id: this.id
        })
        .then(res => {
          this.ruleForm.name = res.data.data.nickname;
          this.ruleForm.age = res.data.data.age;
          this.ruleForm.height = res.data.data.height;
          this.ruleForm.weight = res.data.data.weight;
          this.ruleForm.detail = JSON.parse(res.data.data.detail).join(', ');

          let colorArr = JSON.parse(res.data.data.color)
          let colorDislikeArr = []
          for (let i in colorArr) {
            if (colorArr[i] == 2) {
              colorDislikeArr.push(this.colors[i])
            }
          }
          this.ruleForm.colorDislikeArr = colorDislikeArr;

          // 以下是老代码 需要替换
          this.cupfromradio = res.data.data.type;
          this.ruleForm.desc = res.data.data.desc;
          let start_at = new Date(parseInt(res.data.data.start_at)).toLocaleString()
          let end_at = new Date(parseInt(res.data.data.end_at)).toLocaleString()
          this.ruleForm.datelimit = [start_at, end_at]
          this.datepickerPlaceholder = start_at + ' - ' + end_at
          let coupon_ids = res.data.data.coupon_ids.split(',')
        });
    },
    timestampToTime(timestamp) {
      var date = new Date(timestamp * 1);
      var Y = date.getFullYear() + "/";
      var M =
        (date.getMonth() + 1 < 10
          ? "0" + (date.getMonth() + 1)
          : date.getMonth() + 1) + "/";
      var D =
        (date.getDate() < 10 ? "0" + date.getDate() : date.getDate()) + "  ";
      var h =
        (date.getHours() < 10 ? "0" + date.getHours() : date.getHours()) + ":";
      var m =
        (date.getMinutes() < 10 ? "0" + date.getMinutes() : date.getMinutes()) +
        ":";
      var s =
        date.getSeconds() < 10 ? "0" + date.getSeconds() : date.getSeconds();
      return Y + M + D + h + m + s;
    },
    goBackPage() {
      this.$router.go(-1);
    },
    submitForm(formName) {
      if (this.id !== 0) {
        if (
          this.ruleForm.name == "" ||
          this.ruleForm.desc == ""
        ) {
          this.$message.error("数据不完整！");
          return false;
        }
        let form = {};
        form.name = this.ruleForm.name;
        form.desc = this.ruleForm.desc;
        form.type = this.cupfromradio;
        form.start_at = new Date(this.ruleForm.datelimit[0]).getTime();
        form.end_at = new Date(this.ruleForm.datelimit[1]).getTime();
        form.coupon_ids = this.pointCouponList.map((e) => {
          return e.id
        }).join(',')
        this.axios
          .post("couponbag/update", {
            id: this.id,
            form: form
          })
          .then(res => {
            if (res.data.errno === 0) {
              this.$message({
                type: "success",
                message: "更新成功!"
              });
              this.goBackPage();
            } else {
              this.$message.error("异常 ！");
            }
          });
      } else {
        //总体判断信息是否完整
        // let isComp = true
        //   if (this.cupusergetradio == 13) { //卡包实时到账
        //     if (this.ruleForm.name == '' || this.ruleForm.Instructions == ''|| this.ruleForm.value == '') { isComp = false }
        //   }else if(this.cupusergetradio == 14){
        if (
          this.ruleForm.name == "" ||
          this.ruleForm.desc == ""
        ) {
          // isComp = false }
          this.$message.error("数据不完整！");
          return false;
        }
        //卡包有效期判断
        let datelimit = true;
        if (this.cupableradio == 6) {
          if (this.ruleForm.datelimit == "") {
            datelimit = false;
          }
        }
        if (datelimit == false) {
          this.$message.error("卡包有效期未设置或设置为0！");
          return false;
        }
        // //判断指定用户
        // let pointuser = true;
        // if (this.cupuserradio == 12) {
        //   if (this.pointUserList.length == 0) {
        //     pointuser = false;
        //   }
        // }
        // if (pointuser == false) {
        //   this.$message.error("选择指定用户不能为空！");
        //   return false;
        // }

        //数据整合-------------------------------------------------------------------------------------------
        let CupTime = {}; //卡包时间
        if (this.cupableradio == 6) {
          CupTime.start = new Date(this.ruleForm.datelimit[0]).getTime();
          CupTime.end = new Date(this.ruleForm.datelimit[1]).getTime();
          CupTime.limit_day = CupTime.end - CupTime.start;
          CupTime.create = new Date().getTime();
        }

        // let CupState = {}; //状态
        // //卡包形式
        // if (this.cupfromradio == 1) {
        //   CupState.CupFrom = 0;
        // } else if (this.cupfromradio == 2) {
        //   CupState.CupFrom = 1;
        // }
        // if (this.cuplimtradio == 4) {
        //   CupState.CupLimit = 0;
        // } else if (this.cuplimtradio == 5) {
        //   CupState.CupLimit = 1;
        // }
        // if (this.cupableradio == 6) {
        //   CupState.CupTime = 0;
        // }
        // CupState.CupAble = 1;
        // if (this.cuppointradio == 9) {
        //   this.pointCouponList = [];
        // }
        // if (this.cupuserradio == 11) {
        //   this.pointUserList = [];
        // }
        let form = this.ruleForm
        form.type = this.cupfromradio
        form.start_at = CupTime.start
        form.end_at = CupTime.end
        form.coupon_ids = this.pointCouponList.map((e) => {
          return e.id
        }).join(',')
        //存入数据库
        this.axios
          .post("couponbag/save", {
            form: this.ruleForm,
            // CupTime: CupTime,
            // CupState: CupState,
            // UserList: this.pointUserList,
          })
          .then(res => {
            if (res.data.errno == 0) {
              this.$message({
                type: "success",
                message: "保存成功!"
              });
              this.goBackPage();
            } else {
              this.$message.error("异常 ！");
            }
          });
      }
    }
  }
};
</script>

<style lang="css" scoped>
.tipa {
  color: #888;
  font-size: 12px;
  line-height: 30px;
}
.page-box {
  margin-bottom: 15px;
}
.CouponPopoverTitle {
  margin: 20px 20px 0 20px;
  padding-bottom: 10px;
  /* border-bottom: 1px solid #ddd; */
}
.CouponPopover {
  position: fixed;
  /* border: 1px solid black; */
  width: 600px;
  /* height: 450px; */
  /* left: calc(50vw - 250px + 200px);
  top: calc(50vh - 150px); */
  background: rgba(255, 255, 255, 0.9);
}
/* .CouponPopoverArea {
  position: fixed;
  background: rgba(0, 0, 0, 0.1);
  left: 0;
  top: 0;
  z-index: 999;
  width: 100vw;
  height: 100vh;
} */
.listItemAction {
  border-bottom: 1px solid #eee;
}
.listItemName {
  /* border: 1px solid black; */
  border-bottom: 1px solid #eee;
  color: #333;
  overflow: hidden;
  text-overflow: ellipsis;
  display: -webkit-box;
  -webkit-line-clamp: 1;
  -webkit-box-orient: vertical;
  /* margin-top: 4px; */
}
.pointCouponTip {
  /* border: 1px solid black; */
  text-align: center;
  font-size: 12px;
  color: #8391a5;
}
.pointCouponTitle_right {
  /* border: 1px solid black; */
  float: left;
  height: 30px;
  width: 70px;
  text-align: center;
  line-height: 30px;
  /* padding-left: 10px; */
}
.pointCouponTitle_left {
  /* border: 1px solid black; */
  float: left;
  height: 30px;
  width: 198px;
  line-height: 30px;
  padding-left: 10px;
}
.pointCouponTitle {
  background: #f8f8f8;
  color: #333;
  font-size: 15px;
  width: 278px;
  height: 30px;
  /* border: 1px solid black */
}
.pointCouponArea {
  width: 278px;
  margin-top: 5px;
  height: auto;
  /* border: 1px solid black */
}
.radioselect {
  margin: 20px 0px;
}
.cupable {
  /* border: 1px solid black; */
  margin-top: 10px;
}
.dateblock {
  margin-top: 15px;
  margin-bottom: 0px;
  /* margin: 15px 0; */
}
.cupfromareatip {
  font-size: 16px;
  color: #8391a5;
}
.cupfromarea {
  font-size: 16px;
  color: #333;
}
.el-radio {
  color: #333;
}
.tiptitle {
  /* border: 1px solid black; */
  border-bottom: 1px solid #8391a5;
  /* display: inline-block; */
  font-weight: bold;
  margin-top: -5px;
  padding-left: 2px;
  padding-bottom: 5px;
  margin-bottom: 10px;
  color: #333;
  font-size: 16px;
}


.color-container {
  display: flex;
  justify-content: center;
}
.color {
  width: 40px;
  height: 40px;
  margin: 5px;
  border: 1px solid black;
}

.color-0 {
  background: #000000
}
.color-1 {
  background: #FFFFFF
}
.color-2 {
  background: #C0C0C0
}
.color-3 {
  background: #082E54
}
.color-4 {
  background: #A39480
}
.color-5 {
  background: #8B4513
}
.color-6 {
  background: #F8F8FF
}
.color-7 {
  background: #FFFF00
}
.color-8 {
  background: #6B8E23
}
.color-9 {
  background: #4169E1
}
.color-10 {
  background: #FF0000
}
.color-11 {
  background: #9013FE
}
</style>
