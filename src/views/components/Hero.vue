<template>
    <section class="section section-shaped my-0">
        <div class="shape-primary">
            <span class="span-150"></span>
            <span class="span-50"></span>
            <span class="span-50"></span>
            <span class="span-75"></span>
            <span class="span-100"></span>
            <span class="span-75"></span>
            <span class="span-50"></span>
            <span class="span-100"></span>
            <span class="span-50"></span>
            <span class="span-100"></span>
        </div>
        <div class="container shape-container d-flex align-items-center">
            <div class="col px-0">
                <div class="row justify-content-center align-items-center">
                    <div class="col-lg-7 text-center pt-lg" style="height: 800px">
                        <img src="img/brand/3.png" style="width: 200px;" class="img-fluid">
                        <p class="lead text-black-50 mt-4 mb-5">It's Free and Open Source.</p>
                          <div class="input-group-append text-center">
                            <base-input placeholder="What is the correct way to completely remove an application?" style="width: 100%"
                                        addon-left-icon="ni ni-zoom-split-in" v-model = 'query_content'
                                        v-on:focus="focusCustomer" v-on:blur="blurCustomer">
                            </base-input>
                            <base-button type="primary" style="height:45px" id = 'query' v-on:click="to_query">Search</base-button>
                          </div>
                      <div style="clear: both;overflow: auto;height:700px;width: 105%;position: relative;right: 35px">
                          <ul id = "list">
                            <div class = "Result1"  v-for="(answer,index) in resultFromServer.slice(0, 1)" :key="index">
                              <div class = "title1">The Top-1 Answer:======</div>
                              <br />
                              <div class = "content1" v-highlight>Command: <code class="language-vim" style="color: #28a745;font-size: 15px">{{ answer["Command"] }}<br /></code>
                                MP Summary: <code class="language-vim">{{ answer["MP Summary"] }}<br /></code>
                                Most Similar TLDR Task: <code class="language-vim">{{ answer["Most Similar TLDR Task"] }}<br /></code>
                                Most Similar TLDR Script: <pre><code v-highlight>{{ answer["Most Similar TLDR Script"] }}<br /></code></pre>
                                <br />
                                <span style="font-size: 16px;font-weight: bold">### Top-3 Similar Questions with Accepted Scripts ###<br /></span>
                                <div v-for="(a, index) in answer['Top-3 Similar Questions']" :key="index">
                                  Question: <a href="javascript:;" v-on:click="getUrl(a.split(':')[0])"><code>{{a.split(':')[1]}}<br /></code></a>
                                Accepted Script: <pre><code v-highlight>{{answer['Top-3 Scripts'][index].split(':')[1].replace(' ', '')}}<br /></code></pre>
                                </div>
                                <br />
                                <span style="font-weight: bold;font-size: 16px">### Explanations about Options ###<br /></span>
                                <div v-for="(value, key) in answer['Explanations about Options']">
                                  <code>{{key}}:</code>
                                  <span style="font-weight: lighter">{{value}}<br /></span>
                                </div>
                                <br />
                                <div class = "title">Other Answers:======</div>
                                <br />
                                <div class = "Result" v-on:click = "jump" v-for="(answer,index) in resultFromServer.slice(1, resultFromServer.length)" :key="index">
                                  <div class = "content" v-highlight style="background-color: rgba(106,107,100,0.16)">Command:<code v-highlight style="color: #28a745;font-size: 15px">{{ answer["Command"] }}<br /></code>
                                    MP Summary: <code v-highlight>{{ answer["MP Summary"] }}<br /></code>
                                    Most Similar TLDR Task: <code v-highlight>{{ answer["Most Similar TLDR Task"] }}<br /></code>
                                    <Modals :data1="answer"></Modals>
                                </div>
                              </div>
                            </div>
                            </div>
                          </ul>
                      </div>
                    </div>
                </div>
                <div class="row align-items-center justify-content-around stars-and-coded">
                    <div class="col-sm-12 mt-4 mt-sm-0 text-right">
                        <span class="text-black-50 alpha-7"></span>
                    </div>
                </div>
            </div>
        </div>
    </section>
</template>
<script>
import Modals from "./JavascriptComponents/Modals";
export default {
  components :{
    Modals
  },
  data () {
    return {
      msg: 'shellFusion',
      path : 'ws://211.66.130.46:20004/',
      // path: 'ws://localhost:8088',
      ws: {},
      resultFromServer: new Array(),
      queryUrl_dict : {"ul":"https://unix.stackexchange.com/questions/","so":"https://stackoverflow.com/questions/","su":"https://superuser.com/questions/","au":"https://askubuntu.com/questions/"}
    }
  },
  methods: {
    to_query: function () {
      // 点击搜索时候的操作，
      if (this.query_content === undefined) {
        this.query_content = 'What is the correct way to completely remove an application?'
      }
      let queryContent = this.query_content.replace(/\s+/g, '')
      if (queryContent === '') {
        this.query_content = 'What is the correct way to completely remove an application?'
      }
      console.log(queryContent)
      // 开始查询
      this.resultFromServer = new Array()
      this.ws = new WebSocket(this.path);
      this.ws.onopen = () => {
        console.log('ws连接状态：' + this.ws.readyState)
        // alert("连接成功")
        // 连接成功则发送数据
        this.ws.send(queryContent)
      }
      // 接听服务器发回的信息并处理展示
      this.ws.onmessage = (data) => {
        console.log('接收到来自服务器的消息：')
        // console.log(data['data'])
        var tmp = JSON.parse(data['data'])
        // console.log(tmp)
        // console.log(tmp['Answers'][0])
        for(let i in tmp['Answers']){
          this.resultFromServer.push(tmp['Answers'][i])
          // console.log(answer)
        }
        console.log(this.resultFromServer[0])
        // console.log(this.resultFromServer[1])
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
    getUrl: function (e) {
      e = e.split("_")
      window.open(this.queryUrl_dict[e[0]] + e[1] + "/")
    },
  }
};
</script>
<style>
ul{
  width: 100%;
  /** border: 1px solid #e2e2e2; **/
  font-family: 'Microsoft YaHei';
  display: none;
}

.Result{
  width: 100%;
  height: 130px;
  margin: 0 auto;
  font-family: 'Microsoft YaHei';
  font-size: 16px;
  /*padding: 5px 10px 5px 0px; !** 上右下左 **!*/
  text-align: justify;
  color: #191e23;
}


.Result1{
  width: 100%;
  height: 100%;
  margin: 0 auto;
  font-family: 'Microsoft YaHei';
  text-align: left;
  font-size: 20px;
  color: #191e23;
}

.title{
  font-weight: bold;
  font-size: 20px;
  padding: 0px 10px 0px 10px; /** 上右下左 **/
  height: 16px;
}

.content{
  font-weight: bold;
  height: 130px;
  font-family: 'Microsoft YaHei';
  font-size: 14px;
  margin: 5px auto;
  /*padding: 5px 10px 5px 20px; !** 上右下左 **!*/
  text-overflow:ellipsis;
  display: -webkit-box; /** 将对象作为伸缩盒子模型显示 **/
  -webkit-box-orient: vertical; /** 设置或检索伸缩盒对象的子元素的排列方式 **/
  -webkit-line-clamp: 8; /** 显示的行数 **/
  overflow: hidden;  /** 隐藏超出的内容 **/
}

.title1{
  font-weight: bold;
  /*padding: 0px 10px 0px 10px; !** 上右下左 **!*/
  height: 16px;
}

.content1{
  font-weight: bold;
  height: 40px;
  font-family: 'Microsoft YaHei';
  font-size: 14px;
  margin: 5px auto;
}

</style>
