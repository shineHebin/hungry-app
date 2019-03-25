<template>
  <div class="goods">
    <div
      class="menu-wrapper"
      ref="menuWrapper"
    >
      <ul>
        <li
          v-for="(item,index) in goods"
          :key="item.id"
          @click="selectMenu(index,$event)"
          :class="{'current':currenIndex===index}"
          class="menu-item"
        >
          <span class="text border-1px"> <span
              v-show="item.type>0"
              class="icon"
              :class="classMap[item.type]"
            ></span>{{item.name}}</span>
        </li>
      </ul>
    </div>
    <div
      class="foods-wrapper"
      ref="foodsWrapper"
    >
      <ul>
        <li
          v-for="(item,index) in goods"
          :key="index"
          class="food-list food-list-hook"
        >
          <h1 class="title">{{item.name}}</h1>
          <ul>
            <li
              v-for="(food,index) in item.foods"
              :key="index"
              @click="selectFood(food,$event)"
              class="food-item border-1px"
            >
              <div class="icon">
                <img
                  width="57"
                  height="57"
                  :src="food.icon"
                />
              </div>
              <div class="content">
                <h2 class="name">{{food.name}}</h2>
                <p class="desc">{{food.description}}</p>
                <div class="extra">
                  <span class="count">月售{{food.sellCount}}份</span><span>好评率{{food.rating}}%</span>
                </div>
                <div class="price">
                  <span class="new">￥{{food.price}}</span><span
                    class="old"
                    v-show="food.oldPrice"
                  >{{food.oldPrice}}</span>
                </div>
                <div class="cartcontrol-wrapper">
                  <cartcontrol
                    :food="food"
                    @add_to_cart="addToCart"
                  ></cartcontrol>
                </div>
              </div>
            </li>
          </ul>
        </li>
      </ul>
    </div>
    <shopcart
      :select-foods="selectFoods"
      :delivery-price="seller.deliveryPrice"
      :min-price="seller.minPrice"
      ref="shopcart"
    ></shopcart>
    <food
      :food="selectedFood"
      ref="food"
      @add_to_cart="addToCart"
    ></food>
  </div>
</template>

<script>
// import food from "../food/food";
import axios from "axios";
import BScroll from "better-scroll";
import cartcontrol from "../../components/cartcontrol/cartcontrol";
import shopcart from "../../components/shopcart/shopcart";
import food from "../../components/food/food";

const ERR_OK = 0; // 返回成功标志
export default {
  name: "goods",
  components: {
    cartcontrol,
    shopcart,
    food
  },
  props: {
    seller: {
      type: Object
    }
  },
  data() {
    return {
      goods: [],
      listHeight: [], //存储区块的高度
      foodsScrollY: 0,
      selectedFood: {}
    };
  },
  created() {
    this.classMap = ["decrease", "discount", "guarantee", "invoice", "special"];
    axios
      .get("/api/goods")
      .then(response => {
        this.goods = response.data.data;
        this.$nextTick(() => {
          this._initScroll(); // 初始化scroll
          this._calculateHeight(); // 初始化列表高度列表
        });
      })
      .catch(error => {
        console.log(error);
      });
  },
  computed: {
    // 比较scroll 和 右边分栏高度，返回index
    currenIndex() {
      for (let i = 0; i < this.listHeight.length; i++) {
        let height1 = this.listHeight[i];
        let height2 = this.listHeight[i + 1];
        if (
          !height2 ||
          (this.foodsScrollY >= height1 && this.foodsScrollY < height2)
        ) {
          return i;
        }
      }
      return 0;
    },
    selectFoods() {
      let foods = [];
      this.goods.forEach(good => {
        good.foods.forEach(food => {
          if (food.count) {
            foods.push(food);
          }
        });
      });
      return foods;
    }
  },
  methods: {
    _initScroll() {
      this.menuWrapper = new BScroll(this.$refs.menuWrapper, {
        click: true
      });
      this.foodsScroll = new BScroll(this.$refs.foodsWrapper, {
        click: true,
        probeType: 3
      });
      // 监控滚动事件
      this.foodsScroll.on("scroll", pos => {
        this.foodsScrollY = Math.abs(Math.round(pos.y));
      });
    },
    _calculateHeight() {
      let foodList = this.$refs.foodsWrapper.querySelectorAll(
        ".food-list-hook"
      );
      let height = 0;
      this.listHeight.push(height);
      for (let i = 0, l = foodList.length; i < l; i++) {
        let item = foodList[i];
        height += item.clientHeight;
        this.listHeight.push(height);
      }
    },
    selectMenu(index, event) {
      //去掉自带的click事件点击，即pc端直接返回
      if (!event._constructed) {
        return;
      }
      let foodsList = this.$refs.foodsWrapper.getElementsByClassName(
        "food-list-hook"
      );
      let el = foodsList[index];
      //类似jump to的功能,通过这个方法,跳转到指定的dom
      this.foodsScroll.scrollToElement(el, 300);
    },
    addToCart(target) {
      //拿到traget(DOM对象)之后，将其传入shopcart组件中drop(target){}方法，
      //此处用this.$refs调用子组件,访问DOM时用的是ref="menuWrapper"
      this.$nextTick(() => {
        //回调函数异步执行，两个动画效果就不会卡顿了
        this.$refs.shopcart.drop(target);
      });
    },
    selectFood(food, event) {
      if (!event._constructed) {
        return;
      }
      this.selectedFood = food;
      this.$refs.food.show();
    }
  }
};
</script>

<style lang="scss" scoped>
@import "../../common/css/mixin.scss";
.goods {
  display: flex;
  position: absolute;
  top: 174px;
  bottom: 47px;
  width: 100%;
  overflow: hidden;
  .menu-wrapper {
    flex: 0 0 80px;
    width: 80px;
    background: #f3f5f7;
    .menu-item {
      display: table;
      height: 54px;
      width: 56px;
      padding: 0 12px;
      line-height: 14px;
      &.current {
        position: relative;
        z-index: 10;
        margin-top: -1px;
        background: #fff;
        .text {
          @include border-1px-none();
          font-weight: bold;
        }
      }
      .text {
        display: table-cell;
        vertical-align: middle;
        @include border-b-1px(rgba(7, 17, 27, 0.1));
        width: 56px;
        font-size: 12px;
        .icon {
          display: inline-block;
          vertical-align: top;
          width: 12px;
          height: 12px;
          margin-right: 2px;
          background-size: 12px 12px;
          background-repeat: no-repeat;
          &.decrease {
            @include bg-image("images/decrease_3");
          }
          &.discount {
            @include bg-image("images/discount_3");
          }
          &.guarantee {
            @include bg-image("images/guarantee_3");
          }
          &.invoice {
            @include bg-image("images/invoice_3");
          }
          &.special {
            @include bg-image("images/special_3");
          }
        }
      }
    }
  }
  .foods-wrapper {
    flex: 1;
    .title {
      padding-left: 14px;
      height: 26px;
      line-height: 26px;
      border-left: 2px solid #d9dde1;
      font-size: 12px;
      color: rgb(147, 153, 159);
      background: #f3f5f7;
    }
    .food-item {
      display: flex;
      margin: 18px;
      padding-bottom: 18px;
      @include border-b-1px(rgba(7, 17, 27, 0.1));
      &:last-child {
        @include border-1px-none();
        margin-bottom: 0;
      }
      .icon {
        flex: 0 0 57px;
        margin-right: 10px;
      }
      .content {
        flex: 1;
        .name {
          margin: 2px 0 8px 0;
          height: 14px;
          line-height: 14px;
          font-size: 14px;
          font-weight: normal;
          color: rgb(7, 17, 27);
        }
        .desc,
        .extra {
          line-height: 10px;
          font-size: 10px;
          color: rgb(147, 153, 159);
        }
        .desc {
          margin-bottom: 8px;
          line-height: 12px;
        }
        .extra {
          margin-bottom: 8px;
          .count {
            margin-right: 12px;
          }
        }
        .price {
          line-height: 24px;
          font-size: 10px;
          font-weight: 700;
          color: rgb(240, 20, 20);
          .new {
            margin-right: 8px;
            font-size: 14px;
            color: rgb(240, 20, 20);
          }
          .old {
            text-decoration: line-through;
            color: rgb(147, 153, 159);
          }
        }
        .cartcontrol-wrapper {
          position: absolute;
          right: 0;
          bottom: 12px;
        }
      }
    }
  }
}
</style>
