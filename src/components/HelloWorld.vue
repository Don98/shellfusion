<template>
  <div class="hello">
    <h1>{{ msg }}</h1>
    <div id = 'box'>
      <input v-model = "query_content" placeholder="输入查询内容" v-on:focus="focusCustomer" v-on:blur="blurCustomer" @keyup.enter="searchEnterFun"/>
      <button id = "query" v-on:click = "to_query">查询</button>
      <ul id = "list">
        <div class = "Result" v-on:click = "jump" v-for="(answer,index) in resultFromServer" :key="index">
          <div class = "title">{{ answer["Command"] }}</div>
          <div class = "content">{{ answer["Top-3 Scripts"] }}</div>
        </div>
      </ul>
    </div>
  </div>
</template>

<script>
export default {
  name: 'HelloWorld',
  data () {
    return {
      msg: 'shellFusion',
      // path: 'ws://211.66.130.46:20004/',
      path: 'ws://127.0.0.1:20004/',
      ws: {},
      resultFromServer: []
    }
  },
  methods: {
    to_query: function () {
      // 点击搜索时候的操作，
      if (this.query_content === undefined) {
        alert('请输入查询内容')
      }
      let queryContent = this.query_content.replace(/\s+/g, '')
      if (queryContent === '') {
        alert('请输入查询内容')
      }
      console.log(queryContent)
      // 开始查询
      this.resultFromServer = []
      this.ws = new WebSocket(this.path)
      this.ws.onopen = () => {
        console.log('ws连接状态：' + this.ws.readyState)
        // alert("连接成功")
        // 连接成功则发送数据
        this.ws.send(queryContent)
      }
      // 接听服务器发回的信息并处理展示
      this.ws.onmessage = (data) => {
        console.log('接收到来自服务器的消息：')
        console.log(data['data'])
        var tmp = JSON.parse(data['data'])
        console.log(tmp)
        console.log(typeof tmp)
        console.log(tmp['Answers'])
        console.log(typeof tmp['Answers'])
        for (let i in tmp['Answers']) {
          tmp['Answers'][i]['id'] = i
          this.resultFromServer.push(tmp['Answers'][i])
        }
      }
      // 监听连接关闭事件
      this.ws.onclose = () => {
      // 监听整个过程中websocket的状态
        console.log('ws连接状态：' + this.ws.readyState)
      }
      // 监听并处理error事件
      this.ws.onerror = function (error) {
        console.log(error)
      }

      document.getElementById('list').style.display = 'block'
    },
    focusCustomer: function () {
      // 点击搜索框时候对于搜索框关注做的事情
      // if (document.querySelector('input') === document.activeElement) {
      //  this.getSelectData(this.query_content.trim())
      // }
      this.showCustomer = true
    },
    blurCustomer: function () {
      // 不关注的时候做的事情
      this.showCustomer = false
    },
    jump: function () {
      // 点击时候跳转到要的页面
      window.open('https://www.baidu.com')
      // window.open('./one_testing_page')
    },
    // 回车键搜索
    searchEnterFun: function (e) {
      var keyCode = window.event ? e.keyCode : e.which
      var val = e.target.value
      if (keyCode === 13 && val) {
        this.to_query()
      }
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h1, h2 {
  font-weight: normal;
}

#box{
    width: 500;
    margin: 30px auto;
    font-family: 'Microsoft YaHei';
    font-size: 14px;
}

input{
    width: 410px;
    border: 1px solid #e2e2e2;
    height: 40px;
    padding: 0 0 0 20px;
    font-size: 16px;
}

button{
    width: 90px;
    height: 40px;
    background: black;
    color: white;
    text-align: center;
    line-height: 38px;
    cursor: pointer;
    font-size: 16px;
}

ul{
    width: 500;
    /** border: 1px solid #e2e2e2; **/
    font-family: 'Microsoft YaHei';
    display: none;
}

.Result{
    width: 600px;
    height: 60px;
    margin: 0 auto;
    font-family: 'Microsoft YaHei';
    font-size: 16px;
    padding: 5px 10px 5px 0px; /** 上右下左 **/
    text-align: left;
    cursor: pointer;
}

.Result:hover{
    border: 1px solid #e2e2e2;
    border-radius: 6px 6px 6px 6px;
}

.title{
    font-weight: bold;
    padding: 0px 10px 0px 10px; /** 上右下左 **/
    height: 16px;
}

.content{
    height: 28px;
    font-family: 'Microsoft YaHei';
    font-size: 12px;
    margin: 5px auto;
    padding: 5px 10px 5px 20px; /** 上右下左 **/
    text-overflow:ellipsis;
    display: -webkit-box; /** 将对象作为伸缩盒子模型显示 **/
    -webkit-box-orient: vertical; /** 设置或检索伸缩盒对象的子元素的排列方式 **/
    -webkit-line-clamp: 2; /** 显示的行数 **/
    overflow: hidden;  /** 隐藏超出的内容 **/
}
</style>
