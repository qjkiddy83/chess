<template>
  <div class="bg">
    <div class="form">
      <h1>登录</h1>
      <label>姓名：<input type="text" name="name" v-model="name"></label>
      <router-link :to="'/index/'+name">登录</router-link>
    </div>
  </div>
</template>
<script type="text/javascript">
  export default {
  name: 'Login',
  data () {
    return {
      name : ''
    }
  },
  methods:{
  }
}

</script>
<style lang="scss">
  .form{
    height:300px;
    transform: translateY(-50%);
    top: 50%;
    position: relative;
  }
  a{
    display: block;
  }
</style>
