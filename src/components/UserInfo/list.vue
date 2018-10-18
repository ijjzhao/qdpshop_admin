<template>
	<div class="content-page">
		<div class="content-nav">
			<el-breadcrumb class="breadcrumb" separator="/">
				<el-breadcrumb-item :to="{ path: '/dashboard' }">首页</el-breadcrumb-item>
				<el-breadcrumb-item>用户管理</el-breadcrumb-item>
				<el-breadcrumb-item>档案列表</el-breadcrumb-item>
			</el-breadcrumb>
			<div class="operation-nav">
				<router-link to="/dashboard/user/add">
					<!--<el-button type="primary" icon="plus">添加会员</el-button>-->
				</router-link>
			</div>
		</div>
		<div class="content-main">
			<div class="filter-box" style="float:left;">
				<el-form :inline="true" :model="filterForm" class="demo-form-inline">
					<el-form-item label="用户名称">
						<el-input v-model="filterForm.name" placeholder="用户名称"></el-input>
					</el-form-item>
					<el-form-item>
						<el-button type="primary" @click="onSubmitFilter">查询</el-button>
					</el-form-item>
				</el-form>
			</div>
			<div class="form-table-box">
				<el-table :data="tableData" :default-sort = "{prop: 'date', order: 'descending'}" style="width: 100%" border stripe>
					<el-table-column prop="id" label="ID" width="90" sortable align="center">
					</el-table-column>
					<el-table-column prop="nickname" label="昵称" align="center">
					</el-table-column>
					<!-- <el-table-column label="手机号" sortable align="center">
						<template slot-scope="scope">
							<el-tooltip placement="right">
								<div slot="content">
								  {{tableData[scope.$index].mobile_country_e}} - {{tableData[scope.$index].mobile_country}}  +{{tableData[scope.$index].mobile_code}}
								</div>
								<div>{{tableData[scope.$index].mobile}}</div>
							</el-tooltip>
						</template>
					</el-table-column> -->
          <el-table-column label="称呼" sortable width="180" align="center">
					</el-table-column>
          <el-table-column prop="age" label="年龄" sortable width="180" align="center">
					</el-table-column>
          <el-table-column prop="height" label="身高" sortable width="180" align="center">
					</el-table-column>
          <el-table-column prop="weight" label="体重" sortable width="180" align="center">
					</el-table-column>
          <el-table-column prop="create_at" label="创建时间" sortable width="180" align="center">
          </el-table-column>
          <el-table-column prop="update_at" label="更新时间" sortable width="180" align="center">
					</el-table-column>

					<el-table-column label="操作" width="80" align="center">
						<template slot-scope="scope">
							<el-button size="small" @click="handleRowDetail(scope.$index, scope.row)">详情</el-button>
						</template>
					</el-table-column>
				</el-table>
			</div>
			<div class="page-box">
				<el-pagination @current-change="handlePageChange" :current-page="page" :page-size="10" layout="total, prev, pager, next, jumper" :total="total">
				</el-pagination>
			</div>
		</div>
	</div>
</template>

<script>
import { Toast } from 'vant'
import api from '@/config/api';
export default {
	data() {
		return {
			page: 1,
			total: 0,
			filterForm: {
				name: ''
			},
			tableData: [],
			row: [],
			inforow: [],
			CorporateName: '',
		}
	},
	methods: {
    handleRowDetail(index, row) {
      this.$router.push({
        name: "userinfo_detail",
        query: {
          id: row.id
        }
      });
    },
    handlePageChange(val) {
			this.page = val;
			//保存到localStorage
			localStorage.setItem('userinfoPage', this.page)
			localStorage.setItem('userinfoFilterForm', JSON.stringify(this.filterForm));
			this.getList()
		},
		onSubmitFilter() {
			this.page = 1
			this.getList()
		},
		getList() {
			this.axios.get('userinfo', {
				params: {
					page: this.page,
					name: this.filterForm.name
				}
			}).then((response) => {
        this.tableData = response.data.data.data.data
        let userMap = response.data.data.userMap
				this.inforow = []
				for (var i = 0; i < this.tableData.length; i++) {
					this.tableData[i].create_at = this.timestampToTime(this.tableData[i].create_at / 1000)
					this.tableData[i].update_at = this.timestampToTime(this.tableData[i].update_at / 1000)
					this.tableData[i].nickname = userMap[this.tableData[i].user_id]
					let obj = {}
					obj.isfockers = this.tableData[i].user_level_is_fockers == 0 ? false : true
					this.inforow.push(obj)
				}
        this.page = response.data.data.currentPage
        this.total = response.data.data.count
			})
		},
		timestampToTime(timestamp) {
        var date = new Date(timestamp * 1000);//时间戳为10位需*1000，时间戳为13位的话不需乘1000
        var Y = date.getFullYear() + '-';
        var M = (date.getMonth()+1 < 10 ? '0'+(date.getMonth()+1) : date.getMonth()+1) + '-';
				var D = (date.getDate() < 10 ? '0'+date.getDate() : date.getDate()) + '  ';
				var h = (date.getHours() < 10 ? '0'+date.getHours() : date.getHours()) + ':';
				var m = (date.getMinutes() < 10 ? '0'+date.getMinutes() : date.getMinutes()) + ':';
        var s = (date.getSeconds() < 10 ? '0'+date.getSeconds() : date.getSeconds());
        return Y+M+D+h+m+s;
    }
	},
	components: {

	},
	mounted() {
		this.CorporateName = api.CorporateName
		this.getList();
	}
}

</script>

<style scoped>
.tips {
	text-align: left;
	font-size: 0.75rem;
	width: 90%;
	margin: 10px auto;
	text-indent: 1.25rem;
	letter-spacing: 0.2px;
	/* color: #757575 */
}
.PointUserLevelPopover_user_info_left.border {
	height: 120px;
}
.PointUserLevelPopover_user_info_right.border {
	/* border: 1px solid black; */
	height: 120px;
	width: 100px;
	/* margin-left: 10px; */
	/* display: flex; */
	/* justify-content: center; */
	/* align-items: center; */
	overflow-y: auto;
}
.PointUserLevelPopover_user_info.point {
	margin-bottom: 12px;
	/* border: 1px solid black; */
	height: 120px;
}
.PointUserLevelPopover_user_info_left {
	float: left;
}
.PointUserLevelPopover_user_info_right {
	float: right;
	color: #ff5566;
	font-weight: bold;
}
.PointUserLevelPopover_user_info {
	width: 90%;
	margin: 0 auto;
	height: 32px;
	/* border: 1px solid black; */
	color: #757575;
	font-size: 14px;

}
.PointUserLevelPopover_btn {
  /* text-align: right; */
  /* border:1px solid black; */
  height: 50px;
  width: 90%;
  margin: 20px auto;
  margin-bottom: 10px;
  display: flex;
  justify-content: flex-end;
  align-items: center;
}
.PointUserLevelPopover_from {
  /* border:1px solid black; */
  width: 80%;
  margin: 50px auto;
}
.PointUserLevelPopover_title {
  margin: 10px 20px;
  /* border:1px solid black; */
  text-align: center;
  font-size: 20px;
  padding: 10px 0px;
  border-bottom: 1px solid #eee;

}
.PointUserLevelPopover {
  width: 800px;
  /* height: 300px; */
  border-radius: 5px;
}

.selectpopup_right_middle {
	/* border:1px solid black; */
	text-align: right;
	padding-right: 10px;
	height: 40px;
	line-height: 40px;
	font-size: 14px;
	color: #666;
	margin-bottom: 3px;
	margin-top: 3px;
}
.selectpopup_right_table {
	/* border: 1px solid black; */
	max-height: 395px;
	overflow-y: auto;
}
.selectpopup_right_top .small {
	color: #757575;
	font-size: 14px;
}
.selectpopup_right_top {
	/* border:1px solid black; */
	height: 32px;
	text-align: center;
	line-height: 32px;
	margin: 0px 15px;
	font-size: 20px;
	border-bottom: 1px solid #f4f4f4;
	padding: 10px 0px;
	padding-top: 18px;
}
.selectpopup_right {
	background: #eee;
	/* border:1px solid black; */
	width: 100%;
	height: 100%;
}
.selectpopup_top {
	/* border:1px solid black; */
	width: 100%;
	height: 90%;
}
.selectpopup {
	width: 800px;
	height: 600px;
}
</style>
