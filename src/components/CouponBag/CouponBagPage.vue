<template lang="html">
  <div class="content-page">
      <div class="content-nav" >
          <el-breadcrumb class="breadcrumb" separator="/">
              <el-breadcrumb-item :to="{ path: '/dashboard' }">首页</el-breadcrumb-item>
              <el-breadcrumb-item >店铺运营</el-breadcrumb-item>
              <el-breadcrumb-item>卡包管理</el-breadcrumb-item>
              <!-- <el-breadcrumb-item>卡包管理</el-breadcrumb-item> -->
          </el-breadcrumb>
          <div class="operation-nav">
            <router-link to="/dashboard/addcouponbag">
              <el-button type="primary" icon="plus">添加卡包</el-button>
            </router-link>
          </div>
      </div>
      <div class="content-main">
          <div class="filter-box" style="float:left;">
              <el-form :inline="true" :model="filterForm" class="demo-form-inline">
                  <el-form-item label="卡包名称">
                      <el-input v-model="filterForm.coupon_bag_name" @keyup.13.native="onSubmitFilter()"  placeholder="请输入卡包名称"></el-input>
                  </el-form-item>
                  <el-form-item>
                      <el-button type="primary"@click="onSubmitFilter">查询</el-button>
                  </el-form-item>
              </el-form>
          </div>
          <div class="filter-box" style="float:right;">
            <el-select v-model="CouponCategory" placeholder="请选择卡包种类" @change="CouponCategorychange">
              <el-option v-for="item in CouponCategoryList" :key="item.id" :label="item.name" :value="item.id"></el-option>
            </el-select>
            <el-button type="info" plain @click="onClear">清空</el-button>
          </div>
          <div class="form-table-box">
              <el-table :data="tableData" style="width: 100%" border stripe>
                  <el-table-column align="center" label="名称" >
                    <template slot-scope="scope">
                      <div class="username_one">
                        {{tableData[scope.$index].name}}
                      </div>
                    </template>
                  </el-table-column>
                  <el-table-column align="center" label="描述">
                    <template slot-scope="scope">
                      <div class="username_one">
                        {{tableData[scope.$index].desc}}
                      </div>
                    </template>
                  </el-table-column>
                  <el-table-column prop="create_localtime" align="center" width="170" label="创建时间">
                    <template slot-scope="scope">
                      {{tableData[scope.$index].create_at}}
                    </template>
                  </el-table-column>
                  <el-table-column prop="" align="center" width="230" label="基本类型">
                    <template slot-scope="scope">
                      {{tableData[scope.$index].type == 0 ? '新人卡包' : '通用卡包'}}
                    </template>
                  </el-table-column>
                  <el-table-column prop="" align="center" width="200" label="有效期">
                    <template slot-scope="scope">
                      <span>{{tableData[scope.$index].end_localtime}}</span>
                    </template>
                  </el-table-column>
                  <el-table-column prop="" align="center" width="133" label="指定卡券">
                    <template slot-scope="scope">
                      <el-button style="" v-if="tableData[scope.$index].coupon_ids !== ''" size="small" @click="handlepointcoupon(scope.$index, scope.row)">卡券</el-button>
                      <div style="width:44px;height:28px;line-height:28px;text-align:center;" v-else >无</div>
                    </template>
                  </el-table-column>
                  <el-table-column prop="" align="center" width="92" label="启用">
                    <template slot-scope="scope">
                        <el-switch
                        v-model="row[scope.$index].isabled" @change="changeIsable(scope.$index, scope.row)">
                        </el-switch>
                    </template>
                  </el-table-column>
                  <el-table-column label="操作" width="140"  align="center">
                      <template slot-scope="scope">
                          <el-button size="small" @click="handleRowEdit(scope.$index, scope.row)">编辑</el-button>
                          <el-button size="small" type="danger" @click="handleRowDelete(scope.$index, scope.row)">删除</el-button>
                      </template>
                  </el-table-column>
              </el-table>
          </div>
          <div class="page-box">
              <el-pagination @current-change="handlePageChange" :current-page="page" :page-size="10" layout="total, prev, pager, next, jumper" :total="total">
              </el-pagination>
          </div>
      </div>
<!-- ////////////////////////////////////////////////////////////////查看指定商品弹层 -->
      <van-popup v-model="pointcoupon">
        <div class="Cuppointcoupon_area">
          <div class="Cuppointcoupon_area_title">
            指定卡券列表
          </div>
          <div class="Cuppointcoupon_area_table">
            <el-table :data="pointcouponData" :default-sort = "{prop: 'date', order: 'descending'}" style="width: 100%" border stripe>
                <el-table-column prop="coupon_name" align="center"  label="卡券名称" >
                </el-table-column>
            </el-table>
          </div>
        </div>
      </van-popup>
<!-- ////////////////////////////////////////////////////////////////查看指定用户弹层 -->

      <van-popup v-model="pointuser">
        <div class="Cuppointcoupon_area">
          <div class="Cuppointcoupon_area_title">
            指定用户列表
          </div>
          <div class="Cuppointcoupon_area_table">
            <el-table :data="pointuserData" :default-sort = "{prop: 'date', order: 'descending'}" style="width: 100%" border stripe>
                <el-table-column prop="nickname" align="center"  label="用户名称" >
                </el-table-column>
                <el-table-column prop="mobile" align="center"  label="手机号码" >
                </el-table-column>
            </el-table>
          </div>
        </div>
      </van-popup>
    </div>
</template>

<script>
export default {
  name: "CouponBagPage",
  data() {
    return {
      CouponCategory: "", //卡包
      CouponCategoryList: [], //卡包种类列表
      pointcoupon: false,
      pointuser: false,
      pointcouponData: [],
      pointuserData: [],
      page: 1,
      total: 0,
      tableData: [],
      row: [],
      filterForm: {
        coupon_id: ""
      },
      uploaderHeader: {
        "X-Nideshop-Token": localStorage.getItem("token") || ""
      }
    };
  },
  mounted() {
    this.getList();
  },
  methods: {
    handleRowEdit(index, row) {
      this.$router.push({
        name: "addcouponbag",
        query: {
          id: row.id
        }
      });
    },
    handlepointcoupon(index, row) {
      this.pointcoupon = true;
      let pointcouponList = row.coupon_ids.split(",");
      this.axios
        .post("couponbag/findpointcoupon", {
          id: pointcouponList
        })
        .then(res => {
          this.pointcouponData = res.data.data;
        });
    },
    getList() {
      this.axios
        .get("couponbag", {
          params: {
            page: this.page,
            couponbagname: this.filterForm.coupon_bag_name
          }
        })
        .then(response => {
          this.tableData = response.data.data.data;
          for (var i = 0; i < this.tableData.length; i++) {
            let obj = {};
            obj.isabled = this.tableData[i].isabled == 1 ? true : false;
            this.tableData[i].create_localtime = this.timestampToTime(
              this.tableData[i].validity_create
            );
            if (
              this.tableData[i].start_at == "" ||
              this.tableData[i].end_at == ""
            ) {
              obj.is_outof = false;
              this.tableData[i].limit_localtime = this.timestampToDay(
                this.tableData[i].validity_limit_day
              );
            } else {
              if (new Date().getTime() > this.tableData[i].end_at) {
                this.tableData[i].end_localtime = "已过期";
                this.axios
                  .post("couponbag/update", {
                    id: this.tableData[i].id,
                    form: {
                      isabled: 0
                    }
                  })
                  .then(response => {});
                obj.is_outof = true;
              } else {
                this.tableData[i].end_localtime =
                  this.timestampToTime(this.tableData[i].end_at) +
                  " 到期 ";
                obj.is_outof = false;
              }
            }
            // if (this.tableData[i].validity_type == 0) {
            //   obj.day_type = "固定日期" + "^";
            // } else if (this.tableData[i].validity_type == 1) {
            //   obj.day_type = "即时生效" + "^";
            // } else if (this.tableData[i].validity_type == 2) {
            //   obj.day_type = "次日生效" + "^";
            // }
            // if (this.tableData[i].coupon_limit == 0) {
            //   obj.limit_type = "通用" + "^";
            // } else if (this.tableData[i].coupon_limit == 1) {
            //   obj.limit_type =
            //     "满" + this.tableData[i].coupon_limit_value + "元" + "^";
            // }
            // if (this.tableData[i].coupon_type == 0) {
            //   obj.type =
            //     "减" + (this.tableData[i].coupon_value / 1).toFixed(2) + "券";
            // } else if (this.tableData[i].coupon_type == 1) {
            //   obj.type = this.tableData[i].coupon_value + "折券";
            // }
            this.row.push(obj);
            // this.tableData[i].disprice = (
            //   this.tableData[i].coupon_value / 1
            // ).toFixed(2);
          }
          this.page = response.data.data.currentPage;
          this.total = response.data.data.count;
        });
    },
    timestampToDay(timestamp) {
      if (timestamp >= 86400000) {
        var D = (timestamp / 86400000).toFixed(2);
        var DL = D.split(".");
        var Y = ((timestamp - DL[0] * 86400000) / 3600000).toFixed(2);
        var YL = Y.split(".");
        var H = (
          (timestamp - DL[0] * 86400000 - YL[0] * 3600000) /
          60000
        ).toFixed(2);
        var HL = H.split(".");
        var M = (
          (timestamp - DL[0] * 86000000 - YL[0] * 3600000 - HL[0] * 60000) /
          60000
        ).toFixed(2);
        var ML = M.split(".");
        return DL[0] + " 天 " + YL[0] + " 小时 " + HL[0] + " 分钟 ";
      }
      if (timestamp == 0) {
        return "有效期错误";
      }
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
    //启用禁用卡包
    changeIsable(index, row) {
      let dialogText= `是否修改卡包"${this.tableData[index].name}"的启用状态?`;

      this.$confirm(dialogText, "修改实时生效！", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      })
        .then(() => {
          this.axios
            .post("couponbag/update", {
              id: this.tableData[index].id,
              form: {
                isabled: this.row[index].isabled ? 1 : 0
              }
            })
            .then(response => {
              if (response.data.errno === 0) {
                this.$message({
                  type: "success",
                  message: "修改成功!"
                });
              }
            });
        })
        .catch(() => {
          this.row[index].isabled = !this.row[index].isabled;
          this.$message({
            type: "info",
            message: "已取消修改！"
          });
        });
    },
    //清空查询信息
    onClear() {},
    //卡包种类
    CouponCategorychange() {},
    //下一页卡包列表
    handlePageChange(val) {
      this.page = val;
      //保存到localStorage
      localStorage.setItem("page", this.page);
      this.getList();
    },
    //删除卡包
    handleRowDelete(index, row) {
      this.$confirm("此操作将永久删除该文件, 是否继续?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      })
        .then(() => {
          this.axios
            .post("couponbag/delete", {
              id: row.id
            })
            .then(res => {
              this.tableData = [];
              this.row = [];
              this.getList();
            });
          this.$message({
            type: "success",
            message: "删除成功!"
          });
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消删除"
          });
        });
    },
    // //查看卡包
    // handleRowEdit(){
    //
    // },
    //查询卡包
    onSubmitFilter() {
      this.getList();
    }
  }
};
</script>

<style lang="css" scoped>
.username_one {
  overflow: hidden;
  text-overflow: ellipsis;
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
}
.Cuppointcoupon_area {
  margin: 12px;
  width: 400px;
}
.Cuppointcoupon_area_title {
  text-align: center;
  font-weight: bold;
  padding: 15px 0 25px 0;
}
/* .el-upload--picture-card {
  width: 40px !important;
  height: 40px !important;
} */
</style>
