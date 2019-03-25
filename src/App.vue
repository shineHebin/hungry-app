<template>
  <div id="app">
    <v-header
      class="header"
      :seller="seller"
    ></v-header>
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
    <keep-alive>
      <router-view :seller=seller></router-view>
    </keep-alive>
  </div>
</template>

<script>
import header from "./components/header/header";
import axios from "axios";
import { urlParse } from "./common/js/util.js";

const ERR_OK = 0;
export default {
  name: "App",
  components: {
    "v-header": header
  },
  data() {
    return {
      seller: {
        id: (() => {
          let queryParam = urlParse();
          return queryParam.id;
        })()
      }
    };
  },
  created() {
    axios
      .get("/api/seller?id=" + this.seller.id)
      .then(res => {
        if (res.data.errno === ERR_OK) {
          this.seller = Object.assign({}, this.seller, res.data.data);
          //this.seller = res.data.seller;
        }
      })
      .catch(error => {
        console.log(error);
      });
  }
};
</script>

<style lang="scss" scoped>
@import "../src/common/css/mixin.scss";
.tab {
  display: flex;
  width: 100%;
  height: 40px;
  line-height: 40px;
  font-size: 14px;
  .tab-item {
    flex: 1;
    text-align: center;
    @include border-b-1px(rgba(7, 17, 27, 0.1));
    a {
      font-size: 14px;
      color: rgb(77, 85, 93);
      display: block;
    }
    .active {
      color: rgb(240, 20, 20);
    }
  }
}
</style>

