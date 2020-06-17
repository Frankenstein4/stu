<template>
  <div class="top-container">
    <div class="left-box">
      <div class="icon-wrapper">
        <span class="iconfont icon-home"></span>
        <span class="iconfont icon-home"></span>
        <span class="iconfont icon-home"></span>
        <span class="iconfont icon-home"></span>
        <span class="iconfont icon-home"></span>
        <span class="iconfont icon-home"></span>
        <!-- <span class="iconfont icon-sami-select"></span>
        <span class="iconfont icon-full-screen"></span> -->
      </div>
      <!-- <div class="history-wrapper">
        <span class="iconfont icon-arrow-lift"></span>
        <span class="iconfont icon-arrow-right"></span>
      </div> -->
    </div>
    <div class="right-box">
      <div class="el-input el-input--small el-input--prefix">
        <!-- 搜索框 -->
        <input
          type="text"
          autocomplete="off"
          placeholder="搜索"
          class="el-input__inner"
          v-model="inputValue"
          @keyup.enter="toResult"
        />
        <span class="el-input__prefix">
          <i class="el-input__icon el-icon-search"></i>
        </span>
      </div>
    </div>
  </div>
</template>

<script>
  export default {
    name: 'top',
    data() {
      return {
        inputValue:''
      }
    },
    methods:{
      toResult(){
        if(this.inputValue==""){
           this.$message({
              message: '搜索内容不能为空！',
              type: 'warning'
            })
        }else{
          //this.$router.push(`/result?query=${this.inputValue}`)
          this.$router.push('/result?query='+this.inputValue)
        }
      }
    }
  }
</script>

<style scoped>
</style>
