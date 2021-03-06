<template>
  <div
    class="seller"
    ref="seller"
  >
    <div class="seller-content">
      <div class="overview">
        <h1 class="title">{{seller.name}}</h1>
        <div class="desc border-1px">
          <star
            :size="36"
            :score="seller.score"
          ></star>
          <span class="text">({{seller.ratingCount}})</span>
          <span class="text">月售{{seller.sellCount}}单</span>
        </div>
        <ul class="remark">
          <li class="block">
            <h2>起送价</h2>
            <div class="content">
              <span class="stress">{{seller.minPrice}}</span>元
            </div>
          </li>
          <li class="block">
            <h2>商家配送</h2>
            <div class="content">
              <span class="stress">{{seller.deliveryPrice}}</span>元
            </div>
          </li>
          <li class="block">
            <h2>平均配送时间</h2>
            <div class="content">
              <span class="stress">{{seller.deliveryTime}}</span>元
            </div>
          </li>
        </ul>
        <div
          class="favorite"
          @click="toggleFavorite"
        >
          <span
            class="icon-favorite"
            :class="{active:favorite}"
          ></span>
          <span class="text">{{favoriteText}}</span>
        </div>
      </div>
      <split></split>
      <div class="bulletin">
        <h1 class="title">商家公告</h1>
        <div class="content-wrapper">
          <p class="content">{{seller.bulletin}}</p>
        </div>
        <ul class="supports">
          <li
            class="supports-item border-1px"
            v-for="(item,index) in seller.supports"
            :key="index"
          >
            <span
              class="icon"
              :class="classMap[seller.supports[index].type]"
            ></span>
            <span class="text">{{seller.supports[index].description}}</span>
          </li>
        </ul>
      </div>
      <split></split>
      <div class="pics">
        <h1 class="title">商家实景</h1>
        <div
          class="pic-wrapper"
          ref="picwrapper"
        >
          <ul
            class="pic-list"
            ref="piclist"
          >
            <li
              class="pic-item"
              v-for="(pic,index) in seller.pics"
              :key="index"
            >
              <img
                :src="pic"
                width="120"
                height="90"
                alt="商家实景"
              >
            </li>
          </ul>
        </div>
      </div>
      <split></split>
      <div class="info">
        <h1 class="title">商家信息</h1>
        <ul>
          <li
            class="info-item"
            v-for="(info,index) in seller.infos"
            :key="index"
          >{{info}}</li>
        </ul>
      </div>
    </div>
  </div>
</template>

<script>
import BScroll from "better-scroll";
import star from "../star/star";
import split from "../split/split";
import { saveToLocal, loadFromLocal } from "../../common/js/store.js";

export default {
  props: {
    seller: {
      type: Object
    }
  },
  data() {
    return {
      // localStorage
      favorite: (() => {
        return loadFromLocal(this.seller.id, "favorite", false);
      })()
    };
  },
  created() {
    this.classMap = ["decrease", "discount", "special", "invoice", "guarantee"];
  },
  watch: {
    seller() {
      this._initScroll();
      this._initPics();
    }
  },
  mounted() {
    this.$nextTick(() => {
      this._initScroll();
      this._initPics();
    });
  },
  components: {
    star,
    split
  },
  computed: {
    favoriteText() {
      return this.favorite ? "已收藏" : "收藏";
    }
  },
  methods: {
    //收藏商家
    toggleFavorite() {
      //去掉自带的click事件点击，即pc端直接返回
      if (!event._constructed) {
        return;
      }
      this.favorite = !this.favorite;
      saveToLocal(this.seller.id, "favorite", this.favorite);
    },
    //初始化better-scroll
    _initScroll() {
      if (!this.scroll) {
        this.scroll = new BScroll(this.$refs.seller, {
          click: true
        });
      } else {
        this.scroll.refresh();
      }
    },
    // 初始化图片滚动
    _initPics() {
      if (this.seller.pics) {
        let picWidth = 120;
        let margin = 6;
        let width = (picWidth + margin) * this.seller.pics.length - margin;
        this.$refs.piclist.style.width = width + "px";
        this.$nextTick(() => {
          if (!this.picScroll) {
            this.picScroll = new BScroll(this.$refs.picwrapper, {
              scrollX: true, // 横向滚动
              eventPassthrough: "vertical" // 不影响竖向滚动
            });
          } else {
            this.picScroll.refresh();
          }
        });
      }
    }
  }
};
</script>

<style lang="scss" scoped>
@import "../../common/css/mixin.scss";
.seller {
  position: absolute;
  top: 174px;
  bottom: 0;
  left: 0;
  width: 100%;
  overflow: hidden;
  .overview {
    position: relative;
    padding: 18px;
    .title {
      margin-bottom: 8px;
      line-height: 14px;
      font-size: 14px;
      color: rgb(7, 17, 27);
    }
    .desc {
      padding-bottom: 18px;
      @include border-b-1px(rgba(7, 17, 27, 0.1));
      font-size: 0;
      .star {
        display: inline-block;
        vertical-align: top;
        margin-right: 8px;
      }
      .text {
        display: inline-block;
        vertical-align: top;
        line-height: 18px;
        margin-right: 12px;
        font-size: 10px;
        color: rgb(77, 85, 93);
      }
    }
    .remark {
      display: flex;
      padding-top: 18px;
      .block {
        flex: 1;
        text-align: center;
        border-right: 1px solid rgba(7, 17, 27, 0.1);
        &:last-child {
          border-right: none;
        }
        h2 {
          margin-bottom: 4px;
          line-height: 10px;
          font-size: 10px;
          color: rgb(147, 153, 159);
        }
        .content {
          line-height: 24px;
          font-size: 10px;
          color: rgb(7, 17, 27);
          .stress {
            font-size: 24px;
          }
        }
      }
    }
    .favorite {
      position: absolute;
      width: 50px;
      right: 11px;
      top: 18px;
      text-align: center;
      .icon-favorite {
        margin-bottom: 4px;
        display: block;
        line-height: 24px;
        font-size: 24px;
        color: #d4d6d9;
        &.active {
          color: rgb(240, 20, 20);
        }
        .text {
          line-height: 10px;
          font-size: 10px;
          color: rgb(77, 85, 93);
        }
      }
    }
  }
  .bulletin {
    padding: 18px 18px 0 18px;
    .title {
      margin-bottom: 8px;
      line-height: 14px;
      font-size: 14px;
      color: rgb(7, 17, 27);
    }
    .content-wrapper {
      padding: 0 12px 16px 12px;
      @include border-b-1px(rgba(7, 17, 27, 0.1));
      .content {
        line-height: 24px;
        font-size: 12px;
        color: rgb(240, 20, 20);
      }
    }
    .supports {
      .supports-item {
        padding: 16px 12px;
        @include border-b-1px(rgba(7, 17, 27, 0.1));
        font-size: 0px;
        &:last-child {
          @include border-1px-none;
        }
        .icon {
          display: inline-block;
          vertical-align: top;
          width: 16px;
          height: 16px;
          margin-right: 6px;
          background-size: 16px 16px;
          background-repeat: no-repeat;
          &.decrease {
            @include bg-image("images/decrease_4");
          }
          &.discount {
            @include bg-image("images/discount_4");
          }
          &.guarantee {
            @include bg-image("images/guarantee_4");
          }
          &.invoice {
            @include bg-image("images/invoice_4");
          }
          &.special {
            @include bg-image("images/special_4");
          }
        }
        .text {
          display: inline-block;
          vertical-align: top;
          line-height: 16px;
          font-size: 12px;
          color: rgb(7, 17, 27);
        }
      }
    }
  }
  .pics {
    padding: 18px;
    .title {
      margin-bottom: 12px;
      line-height: 14px;
      font-size: 14px;
      color: rgb(7, 17, 27);
    }
    .pic-wrapper {
      width: 100%;
      overflow: hidden;
      white-space: nowrap;
      .pic-list {
        font-size: 0;
        .pic-item {
          display: inline-block;
          margin-right: 6px;
          width: 120px;
          height: 90px;
          &:last-child {
            margin: 0;
          }
        }
      }
    }
  }
  .info {
    padding: 18px 18px 0 18px;
    color: rgb(7, 17, 27);
    .title {
      padding-bottom: 12px;
      line-height: 14px;
      @include border-b-1px(rgba(7, 17, 27, 0.1));
      font-size: 14px;
    }
    .info-item {
      padding: 16px 12px;
      line-height: 16px;
      @include border-b-1px(rgba(7, 17, 27, 0.1));
      font-size: 12px;
      &:last-child {
        @include border-1px-none;
      }
    }
  }
}
</style>
