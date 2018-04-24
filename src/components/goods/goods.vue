<template>
  <div class="goods">
    <div class="meun-wrapper" ref="meunWrapper">
      <ul>
        <li v-for="(item,index) in goods" class="meun-item" :class="{'current':currentIndex===index}"
        :key='item.id' @click="selectMenu(index,$event)">
          <span class="text border-1px">
            <span class="icon" v-show='item.type>0' :class="classMap[item.type]"></span>
            {{item.name}}
          </span>
        </li>
      </ul>
    </div>
    <div class="foods-wrapper" ref="foodsWrapper">
      <ul>
        <li v-for="item in goods" class="food-list food-list-hook" :key='item.id'>
          <h1 class="title">{{item.name}}</h1>
          <ul>
            <li @click="selectFood(food,$event)" v-for="food in item.foods" class="food-item" :key="food.id">
              <div class="icon">
                <img :src="food.icon" alt=""  width="57">
              </div>
              <div class="content">
                <h2 class="name">{{food.name}}</h2>
                <p class="desc">{{food.description}}</p>
                <div class="extra">
                  <span class="count">月售{{food.sellCount}}份</span><span>好评率{{food.rating}}%</span>
                </div>
                <div class="price">
                  <span class="now">￥{{food.price}}</span><span class="old" v-show="food.oldPrice">￥{{food.oldPrice}}</span>
                </div>
                <div class="cartcontrol-wrapper">
                  <cartcontrol @add="addFood" :food="food"></cartcontrol>
                </div>
              </div>
            </li>
          </ul>
        </li>
      </ul>
    </div>
    <shopcart ref="shopcart" :select-foods="selectFoods" :delivery-price="seller.deliveryPrice" :min-price="seller.minPrice"></shopcart>
    <food @add="addFood" :food="selectedFood"  ref="food"></food>
  </div>
</template>

<script type="text/ecmascript-6">
  import BScroll from 'better-scroll';
  import shopcart from 'components/shopcart/shopcart';
  import cartcontrol from 'components/cartcontrol/cartcontrol';
  import food from 'components/food/food';

  const ERR_OK = 0;
  export default {
    props: {
      seller: {
        type: Object
      }
    },
    data() {
      return {
        goods: [],
        listHight: [],
        scrollY: 0,
        selectedFood: {}
      };
    },
    computed: {
      currentIndex() {
        for (let i = 0; i < this.listHight.length; i++) {
          let height1 = this.listHight[i];
          let height2 = this.listHight[i + 1];
          if (!height2 || (this.scrollY >= height1 && this.scrollY < height2)) {
            return i;
          }
        }
        return 0;
      },
      selectFoods() {
        let foods = [];
        this.goods.forEach((good) => {
          good.foods.forEach((food) => {
            if (food.count) {
              foods.push(food);
            }
          });
        });
        return foods;
      }
    },
    created() {
      this.classMap = ['decrease', 'discount', 'special', 'invoice', 'guarantee'];
      this.$http.get('/api/goods').then((response) => {
        response = response.body;
        if (response.errno === ERR_OK) {
          this.goods = response.data;
          console.log(this.goods);
          // 在修改数据之后立即使用这个方法，获取更新后的 DOM。
          this.$nextTick(() => {
            this._initScroll();
            this._calculateHeight();
          });
        }
      });
    },
    methods: {
      selectMenu(index) {
        let foodList = this.$refs.foodsWrapper.getElementsByClassName('food-list-hook');
        let el = foodList[index];
        this.foodsScroll.scrollToElement(el, 300);
        console.log(index);
      },
      selectFood(food) {
        this.selectedFood = food;
        this.$refs.food.show();
      },
      addFood(target) {
        this._drop(target);
      },
      _drop(target) {
        // 体验优化,异步执行下落动画
        this.$nextTick(() => {
          this.$refs.shopcart.drop(target);
        });
      },
      _initScroll() {
        this.meunScroll = new BScroll(this.$refs.meunWrapper, {
          click: true
        });
        this.foodsScroll = new BScroll(this.$refs.foodsWrapper, {
          click: true,
          probeType: 3
          // 当 probeType 为 3 的时候，不仅在屏幕滑动的过程中，而且在 momentum 滚动动画运行过程中实时派发 scroll 事件
        });
        this.foodsScroll.on('scroll', (pos) => {
          // abs() 方法可返回数的绝对值。
          // 判断滑动方向，避免下拉时分类高亮错误（如第一分类商品数量为1时，下拉使得第二分类高亮）
          // if (pos.y <= 0) {
            this.scrollY = Math.abs(Math.round(pos.y));
          // }
        });
      },
      _calculateHeight() {
        let foodList = this.$refs.foodsWrapper.getElementsByClassName('food-list-hook');
        let height = 0;
        this.listHight.push(height);
        for (let i = 0; i < foodList.length; i++) {
          let item = foodList[i];
          height += item.clientHeight;
          this.listHight.push(height);
        }
      }
    },
    components: {
      shopcart,
      cartcontrol,
      food
    }
  };
</script>

<style lang="stylus" rel="stylesheet/stylus">
  @import '../../common/stylus/mixin.styl'
  .goods
    display: flex;
    position: absolute;
    top: 174px
    width: 100%
    bottom: 46px
    overflow: hidden;
    .meun-wrapper
      flex: 0 0 80px
      width: 80px
      background: #f3f5f7
      .meun-item
        display: table;
        height: 54px
        width: 56px
        padding: 0 12px
        line-height: 14px
        &.current
          position: relative;
          margin-top: -1px
          z-index: 10
          background: #fff
          font-weight: 700
          .text
            border-none()
        .icon
          display: inline-block
          vertical-align: top
          width: 12px
          height: 12px
          margin-right: 2px
          background-size: 12px 12px
          background-repeat: no-repeat
          &.decrease
            bg-image('decrease_3')
          &.discount
            bg-image('discount_3')
          &.guarantee
            bg-image('guarantee_3')
          &.invoice
            bg-image('invoice_3')
          &.special
            bg-image('special_3')
        .text
          display: table-cell;
          width: 56px
          vertical-align: middle;
          border-1px(rgba(7, 17, 27, 0.1))
          font-size: 12px
    .foods-wrapper
      flex: 1
      .title
        padding-left: 14px
        height: 26px
        line-height: 26px
        border-left: 2px solid #d9dde1
        font-size: 12px
        color: rgb(147, 153, 159);
        background: #f3f5f7
      .food-item
        display: flex;
        padding: 18px
        border-1px(rgba(7, 17, 27, 0.1))
        &:last-child
         border-none()
         margin-bottom: 0
       .icon
         flex: 0 0 57px
         margin-right: 10px
      .content
        flex: 1
        .name
          margin: 2px 0 8px 0
          height: 14px
          line-height: 14px
          font-size: 14px
          color: rgb(7, 17, 27);
        .desc, .extra
          color: rgb(147, 153, 159);
          line-height: 10px
          font-size: 10px
        .desc
          margin-bottom: 8px
          line-height: 12px
        .extra
          .count
            margin-right: 12px
        .price
          font-weight: 700
          line-height: 24px
          .now
            margin-right: 18px
            font-size: 14px
            color: rgb(240, 20, 20);
          .old
            text-decoration: line-through;
            font-size: 10px
            color: rgb(147, 153, 159);
        .cartcontrol-wrapper
          position: absolute;
          right: 0
          bottom: 12px

</style>
