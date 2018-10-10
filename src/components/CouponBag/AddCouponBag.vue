<template lang="html">
  <div class="content-page">
      <div class="content-nav" >
          <el-breadcrumb class="breadcrumb" separator="/">
              <el-breadcrumb-item :to="{ path: '/dashboard' }">首页</el-breadcrumb-item>
              <el-breadcrumb-item >店铺运营</el-breadcrumb-item>
              <el-breadcrumb-item :to="{ path: '/dashboard/CouponPage' }">卡包管理</el-breadcrumb-item>
              <el-breadcrumb-item>添加卡包</el-breadcrumb-item>
          </el-breadcrumb>
          <div class="operation-nav">
              <el-button type="primary" @click="goBackPage" icon="arrow-left">返回列表</el-button>
          </div>
      </div>

      <div class="content-main">
<!-- ////////////////////////////////////////////////////////////////////////////////////////卡包基础信息开始 -->

        <el-form :model="ruleForm" ref="ruleForm" label-width="180px">
          <div class="tiptitle">
            卡包基础信息
          </div>
          <el-form-item label="卡包名称" >
            <el-input style="width:280px;" placeholder="请输入卡包名称" v-model="ruleForm.name"></el-input>
          </el-form-item>

          <el-form-item label="卡包类型" style="margin-bottom:15px;" >
            <el-radio-group v-model="cupfromradio" @change="cupfromradiochanged">
              <el-radio :label="0" :disabled="is_edit">新人卡包</el-radio>
              <el-radio :label="1" :disabled="is_edit">通用卡包</el-radio>
            </el-radio-group>
          </el-form-item>
<!-- ////////////////////////////////////////////////////////////////////////////////////////卡包基础信息结束 -->
<!-- ////////////////////////////////////////////////////////////////////////////////////////卡包基本规则开始 -->
          <div class="tiptitle" style="padding-top:10px;padding-bottom:10px;">
            卡包基本规则
            </div>
            <el-form-item label="有效期"  style="margin-top:-10px;">
              <el-radio-group class="cupable" v-model="cupableradio">
                <el-radio :label="6" style="color:#757575">固定日期</el-radio>
                <div class="dateblock" v-show="cupableradio == 6">
                    <el-date-picker
                      v-model="ruleForm.datelimit"
                      type="datetimerange"
                      align="left"
                      :placeholder="datepickerPlaceholder"
                      :default-time="['00:00:00', '00:00:00']">
                    </el-date-picker>
                  </div>
                <br/>
              </el-radio-group>
            </el-form-item>
          <el-form-item label="使用说明">
            <el-input type="textarea" style="width:45%;" :rows="2" placeholder="请输入使用说明" v-model="ruleForm.desc">
            </el-input>
          </el-form-item>
          <!-- ////////////////////////////////////////////////////////指定卡券 -->
          <el-form-item label="指定卡券" style="margin-top:12px;" >
            <el-radio-group  v-model="cuppointradio">
              <el-radio :label="10">指定卡券</el-radio>
              <el-button style="margin-left:20px" @click="showPointCouponPopup"
              v-show="cuppointradio == 10" type="primary" size="small" icon="plus">添加卡券</el-button>
            </el-radio-group>
            <div class="pointCouponArea" v-show="cuppointradio == 10">
              <div class="pointCouponTitle">
                <div class="pointCouponTitle_left"> 卡券名称 </div>
                <div class="pointCouponTitle_right"> 操作 </div>
              </div>
              <div class="pointCouponTip" v-if="pointCouponList.length == 0">
                您没有选择指定卡券
              </div>
              <div class="" v-if="pointCouponList.length > 0" v-for="item in pointCouponList">
                <div class="pointCouponTitle_left listItemName"> {{item.name}} </div>
                <div class="pointCouponTitle_right listItemAction ">
                  <el-button type="danger" size="mini" plain @click="delPointCoupon(item.id)">删除</el-button>
                </div>
              </div>
            </div>
          </el-form-item>
<!-- ////////////////////////////////////////////////////////指定卡券弹出层 -->
          <van-popup class="CouponPopover" v-model="selectCouponPopover">
            <el-row class="CouponPopoverTitle">
              <el-col :span="12" style="line-height:37px;font-weight:bold">添加指定卡券</el-col>
              <el-col :span="12" style="text-align:right">
                <el-input v-model="PointCouponInput" @keyup.13.native="SelectPointCoupon()" style="width: 70%;" placeholder="请输入卡券名称"></el-input>
                <el-button type="primary" @click="SelectPointCoupon">搜索</el-button>
              </el-col>
            </el-row>
            <el-table :data="dangerCouponData" border :default-sort = "{prop: 'date', order: 'descending'}" stripe style="width: 100%">
              <el-table-column prop="id" label="ID" align="center" width="100">
              </el-table-column>
              <el-table-column prop="name" label="卡券名称">
              </el-table-column>
              <el-table-column prop="action" align="center" label="操作" width="120">
                <template slot-scope="scope">
                  <el-button size="small" :type="scope.row.btnType"
                  @click="handleSelectCoupon(scope.$index, scope.row)">{{scope.row.btnText}}</el-button>
                </template>
              </el-table-column>
            </el-table>
            <div class="page-box">
                <el-pagination style="margin-right:10px;" @current-change="PointCouponPageChange" :current-page="couponpage" :page-size="5"
                layout="total, prev, pager, next, jumper" :total="coupontotal">
                </el-pagination>
            </div>
          </van-popup>
<!-- ////////////////////////////////////////////////////////指定卡券结束 -->

<!-- ////////////////////////////////////////////////////////////////////////////////////////卡包基本规则结束 -->

          <el-form-item>
            <el-button type="primary" @click="submitForm('ruleForm')">{{is_edit ? '修改' : '立即创建'}}</el-button>
            <el-button @click="resetForm('ruleForm')" v-if="!is_edit">重置</el-button>
          </el-form-item>
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
        .post("couponbag/findcouponbaginfoById", {
          id: this.id
        })
        .then(res => {
          // this.datepickerPlaceholder =
          //   this.timestampToTime(res.data.data.validity_create) +
          //   " 创建，有效期 " +
          //   res.data.data.validity_limit_day / 86400000 +
          //   " 天 ";
          this.ruleForm.name = res.data.data.name;
          this.cupfromradio = res.data.data.type;
          // this.typechangeTime = this.cupfromradio == 1 ? 3 : 2;
          // this.cuplimtradio = res.data.data.coupon_limit == 0 ? 4 : 5;
          this.ruleForm.desc = res.data.data.desc;
          let start_at = new Date(parseInt(res.data.data.start_at)).toLocaleString()
          let end_at = new Date(parseInt(res.data.data.end_at)).toLocaleString()
          this.ruleForm.datelimit = [start_at, end_at]
          this.datepickerPlaceholder = start_at + ' - ' + end_at
          let coupon_ids = res.data.data.coupon_ids.split(',')
          this.handlepointcoupon(coupon_ids)
        });
    },
    handlepointcoupon(coupon_ids) {
      this.axios
        .post("couponbag/findpointcoupon", {
          id: coupon_ids
        })
        .then(res => {
          // this.pointcouponData = res.data.data;
          let data = res.data.data
          for (let i = 0; i < data.length; i++) {
            let row = data[i]
            let obj = {};
            obj.id = row.id;
            obj.name = row.coupon_name;
            // obj.btnType = row.btnType;
            // obj.btnText = row.btnText;
            this.pointCouponList.push(obj);
          }
        });
    },
    ///////////////////////////////////////////////////以下为添加指定卡券
    //添加指定卡券弹出层列表内翻页按钮
    PointCouponPageChange(val) {
      this.couponpage = val;
      //保存到localStorage
      localStorage.setItem("couponpage", this.couponpage);
      this.showPointCouponPopup();
    },
    //添加指定卡券弹出层列表内选定按钮
    handleSelectCoupon(index, row) {
      if (row.btnType == "danger") {
        this.delPointCoupon(row.id);
        let NoData = this.dangerCouponData;
        this.dangerCouponData = [];
        for (var i = 0; i < NoData.length; i++) {
          if (NoData[i].id === row.id) {
            NoData[i].id = row.id;
            NoData[i].name = row.name;
            NoData[i].btnType = "success";
            NoData[i].btnText = "选中";
          }
        }
        this.dangerCouponData = NoData;
      } else {
        let NoData = this.dangerCouponData;
        this.dangerCouponData = [];
        for (var i = 0; i < NoData.length; i++) {
          if (NoData[i].id === row.id) {
            NoData[i].id = row.id;
            NoData[i].name = row.name;
            NoData[i].btnType = "danger";
            NoData[i].btnText = "已选中";
          }
        }
        this.dangerCouponData = NoData;
        let data = []; //已添加卡券；列表
        for (var i = 0; i < this.pointCouponList.length; i++) {
          if (this.pointCouponList[i].id === row.id) {
          } else {
            let obj = {};
            obj = this.pointCouponList[i];
            data.push(obj);
          }
        }
        this.pointCouponList = data;
        let obj = {};
        obj.id = row.id;
        obj.name = row.name;
        obj.btnType = row.btnType;
        obj.btnText = row.btnText;
        this.pointCouponList.push(obj);
      }
    },
    //添加指定卡券弹出层搜索按钮
    SelectPointCoupon() {
      this.showPointCouponPopup();
    },
    //打开指定卡券弹出层
    showPointCouponPopup() {
      this.selectCouponPopover = true;
      this.axios
        .get("couponbag/coupons", {
          params: {
            page: this.couponpage,
            couponname: this.PointCouponInput
          }
        })
        .then(res => {
          this.allCouponData = res.data.data.data;
          this.couponpage = res.data.data.currentPage;
          this.coupontotal = res.data.data.count;
          let NoCouponData = [];
          for (var i = 0; i < this.allCouponData.length; i++) {
            let obj = {};
            obj.id = this.allCouponData[i].id;
            obj.name = this.allCouponData[i].coupon_name;
            obj.btnType = "success";
            obj.btnText = "选中";
            NoCouponData.push(obj);
          }
          for (var i = 0; i < this.pointCouponList.length; i++) {
            for (var j = 0; j < NoCouponData.length; j++) {
              if (this.pointCouponList[i].id === NoCouponData[j].id) {
                NoCouponData[j].btnType = "danger";
                NoCouponData[j].btnText = "已选中";
              }
            }
          }
          this.dangerCouponData = NoCouponData;
        });
    },
    //删除指定卡券
    delPointCoupon(id) {
      let data = [];
      for (var i = 0; i < this.pointCouponList.length; i++) {
        if (this.pointCouponList[i].id === id) {
        } else {
          let obj = this.pointCouponList[i];
          data.push(obj);
        }
      }
      this.pointCouponList = data;
    },
    ///////////////////////////////////////////////////添加指定卡券结束
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
    cupfromradiochanged() {
      this.typechangeTime = Number(this.typechangeTime) + 1;
      if (this.typechangeTime > 3) {
        this.ruleForm.value = "";
      }
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
<!-- } -->
<!-- </script> -->

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
</style>
