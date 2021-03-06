<template>
  <div
    class="ratings"
    ref="ratings"
  >
    <div class="ratings-content">
      <div class="overview">
        <div class="overview-left">
          <h1 class="score">{{seller.score}}</h1>
          <div class="title">綜合评分</div>
          <div class="rank">高于周边商家{{seller.rankRate}}%</div>
        </div>
        <div class="overview-right">
          <div class="score-wrapper">
            <span class="title">服务态度</span>
            <star
              :size="36"
              :score="seller.serviceScore"
            ></star>
            <span class="score">{{seller.serviceScore}}</span>
          </div>
          <div class="score-wrapper">
            <span class="title">商品评分</span>
            <star
              :size="36"
              :score="seller.foodScore"
            ></star>
            <span class="score">{{seller.foodScore}}</span>
          </div>
          <div class="delivery-wrapper">
            <span class="title">送达时间</span>
            <span class="delivery">{{seller.deliveryTime}}分钟</span>
          </div>
        </div>
      </div>
      <split></split>
      <ratingselect
        :select-type="selectType"
        :only-content="onlyContent"
        :desc="desc"
        :ratings="ratings"
        @ratingtype-select="changeSelectType"
        @content-toggle="toggleOnlyContent"
      ></ratingselect>

      <div class="rating-wrapper">
        <ul>
          <li
            v-for="(rating,index) in ratings"
            :key="index"
            class="rating-item border-b-1px"
            v-show="needShow(rating.rateType,rating.text)"
          >
            <div class="avatar">
              <img
                width="28"
                height="28"
                :src="rating.avatar"
              >
            </div>
            <div class="content">
              <h1 class="name">{{rating.username}}</h1>
              <div class="star-wrapper">
                <star
                  :size="24"
                  :score="rating.score"
                ></star>
                <span
                  class="delivery"
                  v-show="rating.deliveryTime"
                >{{rating.deliveryTime}}</span>
              </div>
              <p class="text">{{rating.text}}</p>
              <div
                class="recommend"
                v-show="rating.recommend && rating.recommend.length"
              >
                <span class="icon-thumb_up"></span>
                <span
                  class="item"
                  v-for="(item,index) in rating.recommend"
                  :key="index"
                >{{item}}</span>
              </div>
              <div class="time">
                {{rating.rateTime | formatDate}}
              </div>
            </div>
          </li>
        </ul>
      </div>

    </div>
  </div>
</template>

<script>
import BScroll from "better-scroll";
import axios from "axios";
import star from "../star/star";
import split from "../split/split";
import ratingselect from "../ratingselect/ratingselect";
import { formatDate } from "../../common/js/date.js";

const POSITIVE = 0;
const NEGATIVE = 1;
const ALL = 2;
const ERR_OK = 0;
export default {
  name: "ratings",
  components: {
    star,
    split,
    ratingselect
  },
  props: {
    seller: {
      type: Object
    }
  },
  data() {
    return {
      ratings: [],
      showDetails: false,
      selectType: ALL,
      onlyContent: false,
      desc: {
        all: "全部",
        positive: "推荐",
        negative: "吐槽"
      }
    };
  },
  created() {
    axios
      .get("/api/ratings")
      .then(res => {
        if (res.data.errno === ERR_OK) {
          this.ratings = res.data.data;
          this.$nextTick(() => {
            this.scroll = new BScroll(this.$refs.ratings, {
              click: true
            });
          });
        }
      })
      .catch(error => {
        console.log(error);
      });
  },
  filters: {
    formatDate(time) {
      let date = new Date(time);
      return formatDate(date, "yyyy-MM-dd hh:mm");
    }
  },
  methods: {
    // 判断 评论是否显示
    needShow(type, text) {
      // 先判断是否存在内容和是否只显示有内容的评价
      if (this.onlyContent && !text) {
        return false;
      }
      // 判断是否显示全部评价
      if (this.selectType === ALL) {
        return true;
      } else {
        // 显示选择类型对应的评价
        return type === this.selectType;
      }
    },
    //接收子組件  改变 selectType
    changeSelectType(type) {
      this.selectType = type;
      this.$nextTick(() => {
        this.scroll.refresh();
      });
    },
    // 改变 onlyContent
    toggleOnlyContent(onlyContent) {
      this.onlyContent = onlyContent;
      this.$nextTick(() => {
        this.scroll.refresh();
      });
    }
  }
};
</script>

<style lang="scss" scoped>
@import "../../common/css/mixin.scss";
.ratings {
  position: absolute;
  top: 174px;
  bottom: 0;
  left: 0;
  width: 100%;
  overflow: hidden;
  .overview {
    display: flex;
    padding: 18px 0;
    .overview-left {
      flex: 0 0 137px;
      width: 137px;
      border-right: 1px solid rgba(7, 17, 27, 0.1);
      padding: 6px 0;
      text-align: center;
      @media only screen and (max-width: 320px) {
        flex: 0 0 120px;
        width: 120px;
      }
      .score {
        line-height: 28px;
        font-size: 24px;
        color: rgb(255, 153, 0);
        margin-bottom: 6px;
      }
      .title {
        line-height: 12px;
        font-size: 12px;
        color: rgb(7, 17, 27);
        margin-bottom: 8px;
      }
      .rank {
        line-height: 10px;
        font-size: 10px;
        color: rgb(147, 153, 159);
      }
    }
    .overview-right {
      flex: 1;
      padding: 6px 0 6px 24px;
      @media only screen and (max-width: 320px) {
        padding-left: 6px;
      }
      .score-wrapper {
        margin-bottom: 8px;
        font-size: 0;
        .title {
          display: inline-block;
          line-height: 18px;
          vertical-align: top;
          font-size: 12px;
          color: rgb(7, 17, 27);
        }
        .star {
          display: inline-block;
          vertical-align: top;
          margin: 0 12px;
        }
        .score {
          display: inline-block;
          line-height: 18px;
          vertical-align: top;
          font-size: 12px;
          color: rgb(147, 153, 159);
        }
      }
      .delivery-wrapper {
        font-size: 0;
        .title {
          line-height: 18px;
          font-size: 12px;
          color: rgb(7, 17, 27);
        }
        .delivery {
          margin-left: 12px;
          font-size: 12px;
          color: rgb(147, 153, 159);
        }
      }
    }
  }
  .rating-wrapper {
    padding: 0 18px;
    .rating-item {
      display: flex;
      padding: 18px 0;
      @include border-b-1px(rgba(7, 17, 27, 0.1));
      .avatar {
        flex: 0 0 28px;
        width: 28px;
        margin-right: 12px;
        img {
          border-radius: 50%;
        }
      }
      .content {
        position: relative;
        flex: 1;
        .name {
          margin-bottom: 4px;
          line-height: 12px;
          font-size: 10px;
          color: rgb(7, 17, 27);
        }
        .star-wrapper {
          margin-bottom: 6px;
          font-size: 0;
          .star {
            display: inline-block;
            vertical-align: top;
            margin-right: 6px;
          }
          .delivery {
            display: inline-block;
            vertical-align: top;
            line-height: 12px;
            font-size: 10px;
            color: rgb(147, 153, 159);
            font-weight: 200;
          }
        }
        .text {
          margin-bottom: 8px;
          line-height: 18px;
          font-size: 12px;
          color: rgb(7, 17, 27);
          font-weight: 200;
        }
        .recommend {
          line-height: 16px;
          font-size: 0px;
          .icon-thumb_up,
          .item {
            display: inline-block;
            margin: 0 8px 4px 0;
            font-size: 9px;
          }
          .icon-thumb_up {
            color: rgb(0, 160, 220);
          }
          .item {
            padding: 0 6px;
            border: 1px solid rgba(7, 17, 27, 0.1);
            border-radius: 1px;
            color: rgb(147, 153, 159);
            background: #fff;
          }
        }
        .time {
          position: absolute;
          top: 0;
          right: 0;
          line-height: 12px;
          font-size: 10px;
          color: rgb(147, 153, 159);
          font-weight: 200;
        }
      }
    }
  }
}
</style>
