<template>
  <div class="songs-container">
    <div class="tab-bar">
      <span class="item" :class="{active:tag=='0'}" @click="tag='0'">全部</span>
      <span class="item" :class="{active:tag=='7'}" @click="tag='7'">华语</span>
      <span class="item" :class="{active:tag=='96'}" @click="tag='96'">欧美</span>
      <span class="item" :class="{active:tag=='8'}" @click="tag='8'">日本</span>
      <span class="item" :class="{active:tag=='16'}" @click="tag='16'">韩国</span>
    </div>
    <!-- 底部的table -->
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
        <tr class="el-table__row" v-for="(item,index) in list" :key="index">
          <td>{{index+1}}</td>
          <td>
            <div class="img-wrap">
              <img :src="item.album.picUrl" alt="" />
              <span class="iconfont icon-play" @click="play(item.id)"></span>
            </div>
          </td>
          <td>
            <div class="song-wrap">
              <div class="name-wrap">
                <span>{{item.name}}</span>
                <span class="iconfont icon-mv"></span>
              </div>
              <span>{{item.album.name}}</span>
            </div>
          </td>
          <td>{{item.artists[0].name}}</td>
          <td>{{item.album.name}}</td>
          <td>{{item.duration}}</td>
        </tr>
       
      </tbody>
    </table>
  </div>
</template>

<script>
import axios from 'axios'
export default {
  name: 'songs',
  data() {
    return {
      list:[],
      tag:'96'
    };
  },
  //侦听器
  watch:{
    tag(){
      console.log(this.tag)
      this.songList();
    }
  },
  created(){
    //最新音乐数据
    this.songList();
  },
  methods:{
    //获取最新音乐数据
        songList(){
          axios({
            url: 'https://autumnfish.cn/top/song',
            method: 'get',
            params:{
              type:this.tag
            }
          }).then(res => {
            console.log(res);
            this.list=res.data.data;
            //处理时长
            //假定  350750  毫秒
            //秒    350750/1000  350秒
            //分    350/60 
            //秒    350 % 60   %取余 例如 60%7=4  7*8=56 余数为4
            for(let i=0;i<this.list.length;i++){
              let duration = this.list[i].duration;  
              let min = parseInt(duration/1000/60);
              if(min<10){
                min='0'+min
              }
              let sec = parseInt(duration/1000%60)
              if(sec<10){
                sec='0'+sec
              }
              //console.log(min+" "+sec)
              this.list[i].duration=`${min}:${sec}`
            }
          })
       },
       //播放歌曲
       play(id){
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
       }
  }
}
</script>

<style >

</style>
