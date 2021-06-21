<template>
  <div class="mv-container">
    <div class="mv-wrap">
      <h3 class="title">mv详情</h3>
      <!-- mv -->
      <div class="video-wrap">
        <video
          controls
          autoplay
          loop
          :src="url"
        ></video>
      </div>
      <!-- mv信息 -->
      <div class="info-wrap">
        <div class="singer-info">
          <div class="avatar-wrap">
            <img :src="mvData.cover" />
          </div>
          <span class="name" v-if='mvData.artists!=undefined'>{{mvData.artists[0].name}}</span>
        </div>
        <div class="mv-info">
          <h2 class="title">{{mvData.name}}</h2>
          <span class="date">发布：{{mvData.publishTime}}</span>
          <span class="number">播放：{{mvData.playCount}} 次</span>
          <p class="desc">
            {{mvData.desc}}
          </p>
        </div>
      </div>
      <!-- 精彩评论 -->
      <div class="comment-wrap" v-if="hotComments!=undefined">
        <p class="title">精彩评论<span class="number">({{hotTotal}})</span></p>
        <div class="comments-wrap">
          <div class="item" v-for="(item,index) in hotComments" :key="index">
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
        <p class="title">最新评论<span class="number">({{total}})</span></p>
        <div class="comments-wrap">
          <div class="item" v-for="(item,index) in comments" :key="index">
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
                  <span class="name">{{item.beReplied[0].user.nickname}}：</span>
                  <span class="comment">{{item.beReplied[0].content}}</span>
                </div>
              <div class="date">{{item.time}}</div>
            </div>
          </div>
        
        
        </div>
      </div>
      <!-- 分页器 -->
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
    </div>
    <div class="mv-recommend">
      <h3 class="title">相关推荐</h3>
      <div class="mvs">
        <div class="items">
          <div class="item" v-for="(item,index) in mvs" :key="index" @click="toMv(item.id)">
            <div class="img-wrap">
              <img :src="item.cover" alt="" />
              <span class="iconfont icon-play"></span>
              <div class="num-wrap">
                <div class="iconfont icon-play"></div>
                <div class="num">{{item.playCount}}</div>
              </div>
              <span class="time">{{item.duration}}</span>
            </div>
            <div class="info-wrap">
              <div class="name">{{item.name}}</div>
              <div class="singer">{{item.artistName}}</div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios'
export default {
  name: 'mv',
  data() {
    return {
      // 总条数
      total:0,
      // 页码
      page: 1,
      // 页容量
      limit: 10,
      url:'',
      //mv详情
      mvData:{},
      //热门评论
      hotComments:[],
      hotTotal:0,
      //普通评论数
      comTotal:0,
      comments:[],
      //相关mv
      mvs:[]
    };
  },
  created(){
    //mv详情
     axios({
      url:'https://autumnfish.cn/mv/url',
      method:'get',
      params:{
        id:this.$route.query.id
      }
    }).then(res => {
     // console.log(res);
      this.url=res.data.data.url
    })
    //mv信息
     axios({
      url:'https://autumnfish.cn/mv/detail',
      method:'get',
      params:{
        mvid:this.$route.query.id
      }
    }).then(res => {
      //console.log(res);
      this.mvData=res.data.data
    })
    //mv评论
   axios({
      url:' https://autumnfish.cn/comment/mv',
      method:'get',
      params:{
        id:this.$route.query.id,
        limit:this.limit,
        offset:(this.page-1)*this.limit
      }
    }).then(res => {
      //console.log(res);
      this.hotComments=res.data.hotComments;
      this.hotTotal=this.hotComments.length;
      this.comments=res.data.comments;
      this.total=res.data.total;
    })
    //相关mv
     axios({
      url:' https://autumnfish.cn/simi/mv',
      method:'get',
      params:{
        mvid:this.$route.query.id
      }
    }).then(res => {
      console.log(res);
      this.mvs=res.data.mvs;
      for(let i=0;i<this.mvs.length;i++){
          if(this.mvs[i].playCount>100000){
              this.mvs[i].playCount=parseInt(this.mvs[i].playCount/10000)+'万'
            }
          let duration = this.mvs[i].duration;  
          let min = parseInt(duration/1000/60);
          if(min<10){
            min='0'+min
          }
          let sec = parseInt(duration/1000%60)
          if(sec<10){
            sec='0'+sec
          }
          //console.log(min+" "+sec)
          this.mvs[i].duration=`${min}:${sec}`
        }
    })
  },
  methods: {
     //去mv详情
    toMv(id){
      this.$router.push(`/mv?id=${id}`)
    },
     getMvList(){
     axios({
      url:' https://autumnfish.cn/comment/mv',
      method:'get',
      params:{
        id:this.$route.query.id,
        limit:this.limit,
        offset:(this.page-1)*this.limit
      }
    }).then(res => {
      console.log(res);
      this.hotComments=res.data.hotComments;
      this.comments=res.data.comments;
    })
    },
    handleCurrentChange(val) {
      console.log(`当前页: ${val}`);
      this.page=val;
      this.getMvList()
    }
  }
};
</script>

<style></style>
