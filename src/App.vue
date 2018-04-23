<template>
  <div id="app">
    <!--<img src="./assets/logo.png">-->
    <!--<router-view/>-->
    <v-header :seller='seller'></v-header>
    <div class="tab border-1px">
      <div class="tab-item">
        <router-link to="/goods">商品</router-link>
      </div>
      <div class="tab-item">
        <router-link to="/ratings">评论</router-link>
      </div>
      <div class="tab-item">
        <router-link to="/seller">商家</router-link>
      </div>
    </div>
    <router-view :seller="seller"></router-view>
  </div>
</template>

<script type="text/ecmascript-6">
  import header from 'components/header/header.vue';
  const ERR_OK = 0;
  export default {
//  name: 'App'
    data() {
      return {
        seller: {}
      };
    },
    created() {
      this.$http.get('/api/seller').then((response) => {
        response = response.body;
        console.log(response);
        if (response.errno === ERR_OK) {
          this.seller = response.data;
          console.log(this.seller);
        }
      });
    },
    components: {
      'v-header': header
    }
  };
</script>

<style lang="stylus" rel="stylesheet/stylus">
  @import './common/stylus/mixin.styl'
  #app
    .tab
        display flex
        width 100%
        height 40px
        line-height 40px
        // border-bottom: 1px solid rgba(7, 17, 17, 0.5);
        border-1px(rgba(7, 17, 27, 0.1))
        .tab-item
          flex 1
          text-align center
          & > a
            text-decoration: none;
            display block
            font-size 14px
            color rgb(77, 85, 93)
            &.active
              color: rgb(240, 20, 20)
</style>
