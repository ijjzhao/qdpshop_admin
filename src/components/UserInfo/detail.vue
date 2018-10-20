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
<!-- ////////////////////////////////////////////////////////////////////////////////////////基础信息开始 -->

        <el-form :model="ruleForm" ref="ruleForm" label-width="180px">
          <div class="tiptitle">
            基础信息
          </div>
          <el-row>
            <el-col :span="12">
              <el-form-item label="昵称" >
                <el-input style="width:280px;" placeholder="" v-model="ruleForm.name" readonly></el-input>
              </el-form-item>
              <el-form-item label="称呼" >
                <el-input style="width:280px;" placeholder="" readonly></el-input>
              </el-form-item>
              <el-form-item label="职业" >
                <el-input style="width:280px;" placeholder="" v-model="ruleForm.occupation" readonly></el-input>
              </el-form-item>
              <el-form-item label="细节" >
                <el-input style="width:280px;" placeholder="" v-model="ruleForm.detail" readonly></el-input>
              </el-form-item>
            </el-col>
            <el-col :span="12">
              <el-form-item label="年龄" >
                <el-input style="width:280px;" placeholder="" v-model="ruleForm.age" readonly></el-input>
              </el-form-item>
              <el-form-item label="身高" >
                <el-input style="width:280px;" placeholder="" v-model="ruleForm.height" readonly></el-input>
              </el-form-item>
              <el-form-item label="体重" >
                <el-input style="width:280px;" placeholder="" v-model="ruleForm.weight" readonly></el-input>
              </el-form-item>
            </el-col>
          </el-row>

<!-- ////////////////////////////////////////////////////////////////////////////////////////基础信息结束 -->

          <div class="tiptitle" style="padding-top:10px;padding-bottom:10px;">
            颜色偏好
          </div>
          <el-form-item label="不喜欢的颜色" >
            <div class="pic-container" >
              <div class="color" v-for='item in ruleForm.colorDislikeArr' :key='item.color' :class='"color-" + item.value'>
              </div>
            </div>
          </el-form-item>

          <div class="tiptitle" style="padding-top:10px;padding-bottom:10px;">
            裁剪偏好
          </div>
          <el-form-item label="裁剪" >
            <el-input style="width:280px;" placeholder="" v-model="cuts[ruleForm.cut]" readonly></el-input>
          </el-form-item>

          <div class="tiptitle" style="padding-top:10px;padding-bottom:10px;">
            尺码信息
          </div>
          <el-form-item label="上装" >
            <el-input style="width:280px;" placeholder="" v-model="ruleForm.upsize" readonly></el-input>
          </el-form-item>
          <el-form-item label="下装" >
            <el-input style="width:280px;" placeholder="" v-model="ruleForm.downsize" readonly></el-input>
          </el-form-item>
          <el-form-item label="鞋子" >
            <el-input style="width:280px;" placeholder="" v-model="ruleForm.shoesize" readonly></el-input>
          </el-form-item>

          <div class="tiptitle" style="padding-top:10px;padding-bottom:10px;">
            风格偏好
          </div>
          <el-form-item label="喜欢" >
            <div class="pic-container" >
              <img class="style-pic" v-for='item, index in styles[1]' :key='index' :src='stylePics[item].url'>
              </img>
            </div>
          </el-form-item>
          <el-form-item label="不喜欢" >
            <div class="pic-container" >
              <img class="style-pic" v-for='item, index in styles[2]' :key='index' :src='stylePics[item].url'>
              </img>
            </div>
          </el-form-item>
          <el-form-item label="可以尝试" >
            <div class="pic-container" >
              <img class="style-pic" v-for='item, index in styles[3]' :key='index' :src='stylePics[item].url'>
              </img>
            </div>
          </el-form-item>
          <el-form-item label="没有选择" >
            <div class="pic-container" >
              <img class="style-pic" v-for='item, index in styles[0]' :key='index' :src='stylePics[item].url'>
              </img>
            </div>
          </el-form-item>

          <div class="tiptitle" style="padding-top:10px;padding-bottom:10px;">
            个人照片
          </div>
          <el-form-item label="半身照" >
            <div class="pic-container" >
              <img class="body-pic" :src='pics[0]' v-if='pics[0] != ""'>
              </img>
            </div>
          </el-form-item>
          <el-form-item label="全身照" >
            <div class="pic-container" >
              <img class="body-pic" :src='pics[1]' v-if='pics[1] != ""'>
              </img>
            </div>
          </el-form-item>

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
      ruleForm: {
      },
      id: 0,
      is_edit: false,
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
      cuts: ['修身', '正常', '宽松'],
      sizes: [
        { name: 'S', value: '165' },
        { name: 'M', value: '170' },
        { name: 'L', value: '175' },
        { name: 'XL', value: '180' },
        { name: 'XXL', value: '185' },
        { name: 'XXXL', value: '190' }
      ],
      stylePics:[
        { url: 'https://img.qingdapei.net/brief-deepColor.png', description: '简约' },
        { url: 'https://img.qingdapei.net/brief-sampleColor.png', description: '简约' },
        { url: 'https://img.qingdapei.net/fashion-HK-warmColor.png', description: '轻时尚' },
        { url: 'https://img.qingdapei.net/fashion-simpleColor.png', description: '轻时尚' },
        { url: 'https://img.qingdapei.net/light-deepColor.png', description: '休闲' },
        { url: 'https://img.qingdapei.net/light-middleColor.png', description: '休闲' },
        { url: 'https://img.qingdapei.net/light-sampleColor.png', description: '休闲' },
      ],
      styles: [
        [], // 没有选择
        [], // 喜欢
        [], // 不喜欢
        [], // 可以尝试
      ],
      pics: ['', '']
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
          let data = res.data.data
          this.ruleForm.name = data.nickname;
          this.ruleForm.age = data.age;
          this.ruleForm.height = data.height;
          this.ruleForm.weight = data.weight;
          if (data.detail) {
            this.ruleForm.detail = JSON.parse(data.detail).join(', ');
          }

          let colorDislikeArr = []
          if (data.color) {
            let colorArr = JSON.parse(data.color)
            for (let i in colorArr) {
              if (colorArr[i] == 2) {
                colorDislikeArr.push(this.colors[i])
              }
            }
          }
          this.ruleForm.colorDislikeArr = colorDislikeArr;

          this.ruleForm.cut = data.cut
          // switch (data.cut) {
          //   case -1:
          //     this.ruleForm.cutDesc = ''
          //     break;
          //   case 0:
          //     this.ruleForm.cutDesc = '修身'
          //     break;
          //   case 1:
          //     this.ruleForm.cutDesc = '正常'
          //     break;
          //   case 2:
          //     this.ruleForm.cutDesc = '宽松'
          //     break;
          // }

          this.ruleForm.upsize = ''
          this.ruleForm.downsize = ''
          this.ruleForm.shoesize = ''
          if (data.size) {
            this.ruleForm.size = JSON.parse(data.size)
            if (this.ruleForm.size[0] != -1) this.ruleForm.upsize = this.sizes[this.ruleForm.size[0]].name
            if (this.ruleForm.size[1] != -1) this.ruleForm.downsize = this.sizes[this.ruleForm.size[1]].name
          }

          if (data.style) {
            this.ruleForm.style = JSON.parse(data.style)
            for (let i in this.ruleForm.style) {
              this.styles[this.ruleForm.style[i]].push(i)
            }
          }

          if (data.pics) {
            this.ruleForm.pics = JSON.parse(data.pics)
            this.pics[0] = this.ruleForm.pics[0]
            this.pics[1] = this.ruleForm.pics[1]
          }

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
  }
};
</script>

<style lang="css" scoped>

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

.pic-container {
  /* display: flex; */
  /* justify-content: center; */
  max-width: 700px;
}

.style-pic {
  width: 150px;
  height: 150px;
  margin: 10px;
  border: 1px solid gray;
}

.body-pic {
  width: 350px;
  /* height: 350px; */
  margin: 10px;
  border: 1px solid gray;
}

.color {
  width: 40px;
  height: 40px;
  margin: 5px;
  border: 1px solid black;
  display: inline-block;
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
