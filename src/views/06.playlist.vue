<template>
  <div class="playlist-container">
    <div class="top-wrap">
      <div class="img-wrap">
        <img :src="detail.coverImgUrl" />
      </div>
      <div class="info-wrap">
        <p class="title">{{detail.name}}</p>
        <div class="author-wrap" v-if="detail.creator!=undefined">
          <img class="avatar" :src="detail.creator.avatarUrl"  />
          <span class="name">{{detail.creator.nickname}}</span>
          <span class="time">{{detail.createTime}}</span>
        </div>
        <div class="play-wrap">
          <span class="iconfont icon-circle-play"></span>
          <span class="text">播放全部</span>
        </div>
        <div class="tag-wrap">
          <span class="title">标签:</span>
          <ul>
            <li v-for="(item,index) in detail.tags" :key="index">{{item}}</li>
            <!-- <li>怀旧</li>
            <li>感动</li> -->
          </ul>
        </div>
        <div class="desc-wrap">
          <span class="title">简介:</span>
          <!-- 简介 -->
          <span class="desc">{{detail.description}}</span>
        </div>
      </div>
    </div>
    <el-tabs v-model="activeIndex">
      <el-tab-pane label="歌曲列表" name="1">
        <table class="el-table playlit-table">
          <thead>
            <th></th>
            <th></th>
            <th>音乐标题</th>
            <th>歌手</th>
            <th>专辑</th>
            <th>时长</th>
          </thead>
          <tbody>
            <tr class="el-table__row" v-for="(item,index) in detail.tracks" :key="index" @click="play(item.id)">
              <td>{{index+1}}</td>
              <td>
                <div class="img-wrap">
                  <img :src="item.al.picUrl" alt="" />
                  <span class="iconfont icon-play"></span>
                </div>
              </td>
              <td>
                <div class="song-wrap">
                  <div class="name-wrap">
                    <span>{{item.name}}</span>
                    <span class="iconfont icon-mv"></span>
                  </div>
                  <span></span>
                </div>
              </td>
              <td>{{item.ar[0].name}}</td>
              <td>{{item.al.name}}</td>
              <td>{{item.dt}}</td>
            </tr>
         
          </tbody>
        </table>
      </el-tab-pane>
      <el-tab-pane :label="'评论'+(TotalSum)" name="2">
        <!-- 精彩评论 -->
        <div class="comment-wrap" v-if="hotComments.length!=0">
          <p class="title">精彩评论<span class="number">({{hotTotal}})</span></p>
          <div class="comments-wrap">
            <div class="item" v-for="(item,index) in hotComments" :key="index">
              <div class="icon-wrap">
                <img :src="item.user.avatarUrl"  />
              </div>
              <div class="content-wrap">
                <div class="content">
                  <span class="name">{{item.user.nickname}}：</span>
                  <span class="comment">{{item.content}}</span>
                </div>
                <!-- 评论回复 -->
                <div class="re-content" v-if="item.beReplied.length!=0">
                  <span class="name">{{item.beReplied[0].user.nickname}}:</span>
                  <span class="comment">{{item.beReplied[0].content}}</span>
                </div>
                <div class="date">{{item.time}}</div>
              </div>
            </div>
          </div>
        </div>
        <!-- 最新评论 -->
        <div class="comment-wrap">
          <p class="title">最新评论<span class="number">({{comTotal}})</span></p>
          <div class="comments-wrap">
            <div class="item" v-for="(item,index) in comments" :key="index">
              <div class="icon-wrap">
                <img :src="item.user.avatarUrl" />
              </div>
              <div class="content-wrap">
                <div class="content">
                  <span class="name">{{item.user.nickname}}：</span>
                  <span class="comment">{{item.content}}</span>
                </div>
                <!-- 评论回复 -->
                <div class="re-content" v-if="item.beReplied.length!=0">
                  <span class="name">{{item.beReplied[0].user.nickname}}：</span>
                  <span class="comment">{{item.beReplied[0].content}}</span>
                </div>
                <div class="date">{{item.time}}</div>
              </div>
            </div>
         
          
          </div>
        </div>
        <!-- 分页器 -->
       <!--  <el-pagination
          @current-change="handleCurrentChange"
          background
          layout="prev, pager, next"
          :total="total"
          :current-page="page"
          :page-size="10"
        >
        </el-pagination> -->
        <el-pagination
        @current-change="handleCurrentChange"
        background
        layout="prev, pager, next"
        :total="total"
        :current-page="page"
        :page-size="10"
        :limit="limit"
      >
      </el-pagination>
      </el-tab-pane>
    </el-tabs>
  </div>
</template>

<script>
import axios from 'axios'
export default {
  name: 'playlist',
  data() {
    return {
      activeIndex: '1',
      // 总条数
      total: 0,
      // 页码
      page: 1,
      limit:10,
      //歌单详情数据
      detail:{},
      hotComments:[],
      hotTotal:0,
      //普通评论数
      comTotal:0,
      comments:[],
      //总评论数
      TotalSum:0
    };
  },
  created(){
    //歌单详情
     axios({
      url:'https://autumnfish.cn/playlist/detail',
      method:'get',
      params:{
        id:this.$route.query.id
      }
    }).then(res => {
      console.log(res);
      this.detail=res.data.playlist;
      let arr=this.detail.tracks;
       for(let i=0;i<arr.length;i++){ 
          let duration = arr[i].dt;
          //console.log(duration+"我是时间");  
          let min = parseInt(duration/1000/60);
          if(min<10){
            min='0'+min
          }
          let sec = parseInt(duration/1000%60)
          if(sec<10){
            sec='0'+sec
          }
          //console.log(min+" "+sec)
          this.detail.tracks[i].dt=`${min}:${sec}`       
        }

    })
    //热门评论
    axios({
      url:'https://autumnfish.cn/comment/hot',
      method:'get',
      params:{
        id:this.$route.query.id,
        type:2
      }
    }).then(res => {
      console.log(res);
      this.hotComments=res.data.hotComments
      this.hotTotal=res.data.total;
     
      for(let i=0;i<this.hotComments.length;i++){
          var unixTimestamp = new Date(this.hotComments[i].time) ;
          //重载方法
          Date.prototype.toLocaleString = function() {
                this.hour = this.getHours() < 10 ? "0" + this.getHours() : this.getHours();
                this.minute = this.getMinutes() < 10 ? "0" + this.getMinutes() : this.getMinutes();
                this.second = this.getSeconds() < 10 ? "0" + this.getSeconds() : this.getSeconds();
                return this.getFullYear() + "/" + (this.getMonth() + 1) + "/" + this.getDate() + "/ " + this.hour  + ":" + this.minute + ":" + this.second;
          };
          this.hotComments[i].time = unixTimestamp.toLocaleString();
        }

     
    })

  //最新评论
    this.getNewlist();
    
  },
  methods: {

    //分页评论数   最新评论
      getNewlist(){
   axios({
      url:' https://autumnfish.cn/comment/playlist',
      method:'get',
      params:{
        id:this.$route.query.id,
        limit:this.limit,
        offset:(this.page-1)*this.limit
      }
    }).then(res => {
      console.log(res);
      this.comTotal=res.data.total;
      this.total=res.data.total;
      
      this.TotalSum=this.comTotal+this.hotTotal;
      this.comments=res.data.comments;
      
       for(let i=0;i<this.comments.length;i++){
          var unixTimestamp = new Date(this.comments[i].time) ;
          //重载方法
          Date.prototype.toLocaleString = function() {
                this.hour = this.getHours() < 10 ? "0" + this.getHours() : this.getHours();
                this.minute = this.getMinutes() < 10 ? "0" + this.getMinutes() : this.getMinutes();
                this.second = this.getSeconds() < 10 ? "0" + this.getSeconds() : this.getSeconds();
                return this.getFullYear() + "/" + (this.getMonth() + 1) + "/" + this.getDate() + "/ " + this.hour  + ":" + this.minute + ":" + this.second;
          };
          this.comments[i].time = unixTimestamp.toLocaleString();
        }
    })
    },

     //播放
    play(id){
       axios({
          url: 'https://autumnfish.cn/song/url',
          method: 'get',
          params: {
            id // id:id
          }
        }).then(res => {
            //console.log(res)
          let url = res.data.data[0].url
          if(url==null||url==""||url==undefined){
            this.$message({
              message: '您没有取得权限！',
              type: 'warning'
            })
            
          }else{
            // console.log(this.$parent.musicUrl)
          // 设置给父组件的 播放地址
          this.$parent.musicUrl = url
          }
          
        })
    },


    handleCurrentChange(val) {
      console.log(`当前页: ${val}`);
      this.page=val;
      this.getNewlist()
    }
  }
};
</script>

<style >

</style>
