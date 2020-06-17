<template>
  <div class="result-container">
    <div class="title-wrap">
      <!-- 标题 -->
      <h2 class="title">{{$route.query.query}}</h2>
      <span class="sub-title">找到{{songCount}}个结果</span>
    </div>
    <el-tabs v-model="activeIndex">
      <el-tab-pane label="歌曲" name="songs">
        <table class="el-table">
          <thead>
            <th></th>
            <th>音乐标题</th>
            <th>歌手</th>
            <th>专辑</th>
            <th>时长</th>
          </thead>
          <tbody>
            <tr class="el-table__row" v-for="(item,index) in songList" :key="index" @dblclick="playMusic(item.id)">
              <td>{{index+1}}</td>
              <td>
                <div class="song-wrap">
                  <div class="name-wrap">
                    <span>{{item.name}}</span>
                    <!-- mv的图标 -->
                    <span v-if="item.mvid!=0" class="iconfont icon-mv"></span>
                  </div>
                  <span v-if="item.alias.length!=0">{{item.alias[0]}}</span>
                </div>
              </td>
              <td>{{item.artists[0].name}}</td>
              <td>{{item.album.name}}</td>
              <td>{{item.duration}}</td>
            </tr>
       

          </tbody>
        </table>
      </el-tab-pane>
      <el-tab-pane label="歌单" name="lists">
        <div class="items">
          <div class="item" v-for="(item,index) in playList" :key="index" @click="toPlayList(item.id)">
            <div class="img-wrap">
              <div class="num-wrap">
                播放量:
                <span class="num">{{item.playCount}}</span>
              </div>
              <img :src="item.coverImgUrl" alt="" />
              <span class="iconfont icon-play"></span>
            </div>
            <p class="name">{{item.name}}</p>
          </div>
     
        </div>
      </el-tab-pane>
      <el-tab-pane label="MV" name="mv">
        <div class="items mv">
          <div class="item" v-for="(item,index) in mvList" :key="index" @click="toMv(item.id)">
            <div class="img-wrap">
              <img :src="item.cover" alt="" />
              <span class="iconfont icon-play"></span>
              <div class="num-wrap">
                <div class="iconfont icon-play"></div>
                <div class="num">{{item.playCount}}</div>
              </div>
              <span class="time">02:43</span>
            </div>
            <div class="info-wrap">
              <div class="name">{{item.name}}</div>
              <div class="singer">{{item.artistName}}</div>
            </div>
          </div>
         
     
        </div>
      </el-tab-pane>
    </el-tabs>

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
</template>

<script>
import axios from 'axios'
export default {
  name: 'result',
  data() {
    return {
      activeIndex: 'songs',
      total: 0,
      page:1,
      limit:10,
      //歌曲查询结果
      songList:[],
      playList:[],
      mvList:[],
      //搜索结果总数
      songCount:0,
      type:1
    }
  },
  watch:{
    activeIndex(){
      this.page=1;
      
      switch (this.activeIndex) {
        case 'songs':
          this.type=1;
          break;
        case 'lists':
          this.type=1000;
          break;
        case 'mv':
          this.type=1004;
          break;
        default:
          break;
      }

       axios({
          url:' https://autumnfish.cn/search',
          method:'get',
          params:{
            keywords:this.$route.query.query,
            limit:this.limit,
            offset:(this.page-1)*this.limit,
            type:this.type
          }
        }).then(res => {
          console.log(res);
          if(this.type==1){
              this.songList=res.data.result.songs;
                this.songCount=res.data.result.songCount
                this.total=res.data.result.songCount
                for(let i=0;i<this.songList.length;i++){
                    let duration = this.songList[i].duration;  
                    let min = parseInt(duration/1000/60);
                    if(min<10){
                      min='0'+min
                    }
                    let sec = parseInt(duration/1000%60)
                    if(sec<10){
                      sec='0'+sec
                    }
                    //console.log(min+" "+sec)
                    this.songList[i].duration=`${min}:${sec}`
                  }
          }else if(this.type==1000){
                this.playList=res.data.result.playlists;
                this.total=res.data.result.playlistCount
                for(let i=0;i<this.playList.length;i++){
                  if(this.playList[i].playCount>100000){
                    this.playList[i].playCount=parseInt(this.playList[i].playCount/10000)+'万'
                  }
                }
          }else if(this.type==1004){
                this.mvList=res.data.result.mvs;
                this.total=res.data.result.mvCount
                for(let i=0;i<this.mvList.length;i++){
                  if(this.mvList[i].playCount>100000){
                    this.mvList[i].playCount=parseInt(this.mvList[i].playCount/10000)+'万'
                  }
                }
          }
          /* this.songList=res.data.result.songs;
          this.songCount=res.data.result.songCount
          for(let i=0;i<this.songList.length;i++){
                  let duration = this.songList[i].duration;  
                  let min = parseInt(duration/1000/60);
                  if(min<10){
                    min='0'+min
                  }
                  let sec = parseInt(duration/1000%60)
                  if(sec<10){
                    sec='0'+sec
                  }
                  //console.log(min+" "+sec)
                  this.songList[i].duration=`${min}:${sec}`
                } */
        })

    }
  },
  created(){
    //搜索接口
   this.search();
  },
  methods:{
    //搜索
       search(){
        axios({
          url:' https://autumnfish.cn/search',
          method:'get',
          params:{
            keywords:this.$route.query.query,
            limit:this.limit,
            offset:(this.page-1)*this.limit,
            type:1
          }
        }).then(res => {
          console.log(res);
          //判断搜索内容是否报错  例如搜索了不该搜索的内部
          if(res.data.hasOwnProperty('msg')){
            this.$message({
              message: '搜索内容异常！',
              type: 'warning'
            })
          }else{
          this.songList=res.data.result.songs;
          this.songCount=res.data.result.songCount
          this.total=res.data.result.songCount
          for(let i=0;i<this.songList.length;i++){
                  let duration = this.songList[i].duration;  
                  let min = parseInt(duration/1000/60);
                  if(min<10){
                    min='0'+min
                  }
                  let sec = parseInt(duration/1000%60)
                  if(sec<10){
                    sec='0'+sec
                  }
                  //console.log(min+" "+sec)
                  this.songList[i].duration=`${min}:${sec}`
                }
            }
        },err=>{
          console.log(err)
        })
      },
    //播放
    playMusic(id){
       axios({
          url: 'https://autumnfish.cn/song/url',
          method: 'get',
          params: {
            id // id:id
          }
        }).then(res => {
          // console.log(res)
          let url = res.data.data[0].url
          // console.log(this.$parent.musicUrl)
          // 设置给父组件的 播放地址
          this.$parent.musicUrl = url
        })
    },

    //点击歌单详情id
    toPlayList(id){
      this.$router.push(`/playlist?id=${id}`)
    },

    //去mv详情
    toMv(id){
      this.$router.push(`/mv?id=${id}`)
    },

    handleCurrentChange(val) {
      console.log(`当前页: ${val}`);
      this.page=val;
         axios({
          url:' https://autumnfish.cn/search',
          method:'get',
          params:{
            keywords:this.$route.query.query,
            limit:this.limit,
            offset:(this.page-1)*this.limit,
            type:this.type
          }
        }).then(res => {
          console.log(res);
          if(this.type==1){
              this.songList=res.data.result.songs;
                this.songCount=res.data.result.songCount
                this.total=res.data.result.songCount
                for(let i=0;i<this.songList.length;i++){
                    let duration = this.songList[i].duration;  
                    let min = parseInt(duration/1000/60);
                    if(min<10){
                      min='0'+min
                    }
                    let sec = parseInt(duration/1000%60)
                    if(sec<10){
                      sec='0'+sec
                    }
                    //console.log(min+" "+sec)
                    this.songList[i].duration=`${min}:${sec}`
                  }
          }else if(this.type==1000){
                this.playList=res.data.result.playlists;
                this.total=res.data.result.playlistCount
                for(let i=0;i<this.playList.length;i++){
                  if(this.playList[i].playCount>100000){
                    this.playList[i].playCount=parseInt(this.playList[i].playCount/10000)+'万'
                  }
                }
          }else if(this.type==1004){
                this.mvList=res.data.result.mvs;
                this.total=res.data.result.mvCount
                for(let i=0;i<this.mvList.length;i++){
                  if(this.mvList[i].playCount>100000){
                    this.mvList[i].playCount=parseInt(this.mvList[i].playCount/10000)+'万'
                  }
                }
          }
          /* this.songList=res.data.result.songs;
          this.songCount=res.data.result.songCount
          for(let i=0;i<this.songList.length;i++){
                  let duration = this.songList[i].duration;  
                  let min = parseInt(duration/1000/60);
                  if(min<10){
                    min='0'+min
                  }
                  let sec = parseInt(duration/1000%60)
                  if(sec<10){
                    sec='0'+sec
                  }
                  //console.log(min+" "+sec)
                  this.songList[i].duration=`${min}:${sec}`
                } */
        })
    }

  }
}
</script>

<style >

</style>
