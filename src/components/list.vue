<template>
  <div class="list">
    <h1>机构名称：{{ Jname }}</h1>
    <h1>客户名称：{{ Kname }}</h1>
    <Button class="back-btn" type="primary" size="small" @click="goBack()">返回</Button>
  </div>
</template>
<script>
  export default {
    name: "list",
    data(){
      return{
        id:sessionStorage.deviceId,  //全局获取sessionStorage存储里面的值，这样每个方法事件都能读取到这些参数
        Jname:sessionStorage.Jname,
        Kname:sessionStorage.Kname,
      }
    },
    methods:{
      goBack(){
        this.$router.go(-1);
      }
    },
    mounted(){
      console.log(this.id);
      console.log(this.Jname);
      console.log(this.Kname);
      //获取sessionStorage存储里面的值两种方法都可以，全局和局部
      let devceId = sessionStorage.deviceId;  //局部获取sessionStorage存储里面的值，这样，只能在需要的事件、方法中使用
      console.log(devceId);
    }
  }
</script>
<style scoped>
  .list{
    width:100%;
    height:100%;
    padding:20px;
  }
  .back-btn{
    float:right;
  }
</style>
