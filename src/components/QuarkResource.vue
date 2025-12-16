<template>
  <div class="quarkResource">
    <div class="header">
      <div class="logo"></div>
      <div class="title">影视资源共享</div>
      <div class="info-title" style="font-weight: bold;color: red;text-shadow: 0 0 15px white">夸克或百度网盘 <br>
        <span>转存后免费观看</span>
      </div>
    </div>
    <div class="content">
      <div class="typeFilter">
        <div class="typeList">
          <div class="typeItem" style="font-weight: bold" @click="searchType='';searchInput=''">首页</div>
          <div class="typeItem" v-for="(type,index) in typeList" :key="index"
               @click="searchType=searchType===type?'':type" :class="{typeActive:searchType===type}">
            {{ type }}
          </div>
        </div>
      </div>
      <div class="listBox">
        <div class="row table-header">
          <div class="no">序号</div>
          <div class="name">名称</div>
          <div class="type">类型</div>
        </div>
        <div class="table-body">
          <div class="row" v-for="(i,index) in computedData" :key="index">
            <div class="no">{{ index + 1 }}</div>
            <div class="name" style="position: relative;padding-right: 30px">
              <a :href="i.share_url" target="_blank" rel="noopener noreferrer">{{ i.file_name }}</a>
              <span
                  style="font-size: 12px;color: deeppink;font-weight: bold;padding: 0 10px;position: absolute;right: 0">{{
                  i.shortlink ? '百度' : '夸克'
                }}</span>
            </div>
            <div class="type">{{ i._type }}</div>
          </div>
        </div>

      </div>
      <div class="searchBox">
        <el-input v-model="searchInput" placeholder="请输入剧集名称查询"></el-input>
      </div>
    </div>
    <div class="addToGroup">
      <div class="qqGroup" @click="qqVisible=true"></div>
      <div class="wechatGroup" @click="wxVisible=true"></div>
    </div>
    <div class="info">资源随时可能丢失，请尽快转存或下载!</div>

    <!--qq二维码-->
    <el-dialog
        custom-class="my-dialog"
        title="加入QQ群，实时推送最新资源"
        :visible.sync="qqVisible"
        width="80%"
    >
      <div class="qqQR"></div>
    </el-dialog>
    <!--微信二维码-->
    <el-dialog
        title="加入微信群，实时推送最新资源"
        :visible.sync="wxVisible"
        width="80%"
    >
      <div class="wxQR"></div>
    </el-dialog>
    <!--搜索对话框-->
    <el-dialog
        title="搜索"
        :visible.sync="searchVisible"
        width="80%"
    >
    </el-dialog>
  </div>
</template>

<script>
import dataQuark from '@/assets/json/quarkResource.json'
import dataBaidu from '@/assets/json/baiduShareList.json'
// eslint-disable-next-line no-unused-vars
import Fuse from "fuse.js"
// eslint-disable-next-line no-unused-vars
import baiduNetDictionary from "@/assets/js/baiduNetDictionary";

export default {
  name: 'QuarkResource',
  props: {
    msg: String
  },
  data() {
    return {
      dataQuark: dataQuark,
      dataBaidu: dataBaidu,
      qqVisible: false,
      wxVisible: false,
      searchVisible: false,
      searchInput: '',
      searchType: ''
    }
  },
  mounted() {

  },
  methods: {
    handleType(item) {
      let arr = item.currentPath.split('/')
      let type = arr[1]
      type = type.replace(/##\d+##/g, '')
      return type
    }
  },
  computed: {
    computedData() {
      this.dataQuark.forEach(item => {
        let arr = item.currentPath.split('/')
        let type = arr[1]
        type = type.replace(/##\d+##/g, '')
        item._type = type
      })
      this.dataBaidu.forEach(item => {
        let find = baiduNetDictionary.find(b => {
          return b.shareId === item.shareId
        })
        item._type = item._tag
        item.file_name = find ? find.name : item._name
        item.share_url = `${item.shortlink}?pwd=${item.passwd}`
      })
      /*夸克模糊查询*/
      const fuseOptions = {
        threshold: 0.9,
        keys: [
          "file_name",
        ]
      };
      let allData = [...this.dataQuark, ...this.dataBaidu]
      const fuse = new Fuse(allData, fuseOptions);
      let newData = []
      if (this.searchInput) {
        let searchResult = fuse.search(this.searchInput)
        searchResult.forEach(item => {
          item.item.refIndex = item.refIndex
          if (this.searchType === '') {
            newData.push(item.item)
          } else {
            if (item.item._type === this.searchType) {
              newData.push(item.item)
            }
          }
        })
      } else {
        if (this.searchType) {
          newData = allData.filter(a => {
            return a._type === this.searchType
          })
        } else {
          newData = [...allData]
        }
      }

      return newData
    },
    typeList() {
      let s = []
      this.dataQuark.forEach(item => {
        let type = this.handleType(item)
        s.push(type)
      })
      this.dataBaidu.forEach(item => {
        s.push(item._tag)
      })
      s = new Set(s)
      s = Array.from(s)
      return s
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.quarkResource {
  position: relative;
  height: 100%;
}

.header {
  height: 40px;
  background-color: deepskyblue;
  display: flex;
}

.title {
  line-height: 40px;
  color: white;
  font-size: 25px;
  font-weight: bold;
  text-align: center;
  text-decoration: underline;
}

.logo {
  content: '';
  display: block;
  width: 40px;
  height: 40px;

  background-image: url("../assets/images/logo.png");
  background-position: center;
  background-size: 70% 70%;
  background-repeat: no-repeat;
}

.info-title {
  font-size: 12px;
  flex: 1;
  text-align: right;
  line-height: 20px;
  color: white;
  padding-right: 10px;
}

.content {
  height: calc(100% - 40px);
}

.typeFilter {
  height: 100px;
  font-size: 14px;
  border-bottom: 1px solid rgba(0, 0, 0, 0.2);
}


.typeList {
  height: 100px;
  display: flex;
  flex-wrap: wrap;
  padding: 5px;
}

.typeItem {
  width: 20%;
  color: deepskyblue;
  cursor: pointer;
}

.listBox {
  height: calc(100% - 150px);
}

.searchBox {
  height: 50px;
  display: flex;
}

.searchBox > * {
  margin: auto;
}

.addToGroup {
  position: absolute;
  bottom: 55px;
  width: 40px;
  height: 90px;
  display: flex;
  justify-content: space-between;
  flex-direction: column;
}

.qqGroup, .wechatGroup {
  width: 40px;
  height: 40px;
  background-image: url("../assets/images/qq.png");
  background-size: 100% 100%;
  background-repeat: no-repeat;
  background-position: center;
  cursor: pointer;
}

.wechatGroup {
  background-image: url("../assets/images/微信.png");
  background-size: 90% 90%;
}

.row {
  height: 30px;
  display: flex;
  font-size: 14px;
  line-height: 30px;
}

.table-header {
  color: deepskyblue;
  font-weight: bold;
  justify-content: space-between;
}

.table-header .name {
  text-align: center;
}

.no {
  width: 50px;
  text-align: center;
}

.type {
  width: 100px;
  text-align: center;
}

.name {
  white-space: nowrap; /* 确保文本不会换行 */
  overflow: hidden; /* 隐藏超出容器的文本 */
  text-overflow: ellipsis; /* 当文本超出容器时显示省略号 */
  width: calc(100% - 160px);

}

.table-body {
  height: calc(100% - 30px);
  overflow-y: scroll;
}

.table-body .row:nth-child(odd) {
  background-color: rgba(0, 191, 255, 0.1);
}

.table-header {
  border-bottom: 1px solid rgba(0, 0, 0, 0.2);
}

.info {
  position: absolute;
  bottom: 380px;
  font-size: 14px;
  color: red;
  font-weight: bold;
  transform: rotateZ(90deg);
  left: 25px;
  transform-origin: left top;
}

.qqQR {
  background-image: url("../assets/images/qqQR.png");
  width: 200px;
  height: 200px;
  background-position: center;
  background-size: 100% 100%;
}

.wxQR {

  background-image: url("../assets/images/wxQR.png");

  width: 200px;
  height: 200px;
  background-position: center;
  background-size: 100% 100%;
}

.typeActive {
  color: red;
  animation: move 1s ease infinite;
  font-weight: bold;
  text-shadow: 0 0 15px red;
}

@keyframes move {
  0% {
    transform: translateX(-3px);
  }
  50% {
    transform: translateX(3px);
  }
  100% {
    transform: translateX(-3px);
  }

}
</style>
<style>
.el-dialog__title {
  color: deepskyblue;
  font-size: 12px;
}

.el-dialog__header {
  padding: 10px;
}

.el-dialog__headerbtn {
  top: 15px;
}

.searchBox input {
  box-shadow: 0 0 3px red inset;
}
</style>
