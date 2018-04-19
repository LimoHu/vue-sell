<template>
  <div class="goods">
    <div class="menu-wrapper" ref="menuWrapper">
      <ul>
        <li v-for="(item,index) in goods" :key="index" class="menu-item" :class="{'current':currentIndex===index}">
          <span class="text border-1px">
            <span v-show="item.type>0" class="icon" :class="classMap[item.type]"></span>{{item.name}}
          </span>
        </li>
      </ul>
    </div>
    <div class="foods-wrapper" ref="foodsWrapper">
      <ul>
        <li v-for="(item, index) in goods" :key="index" class="foods-list food-list-hook">
          <h1 class="title">{{item.name}}</h1>
          <ul>
            <li v-for="(food, index) in item.foods" :key="index" class="food-item border-1px">
              <div class="icon">
                <img width="57" height="57" :src="food.icon" alt="">
              </div>
              <div class="content">
                <h2 class="name">{{food.name}}</h2>
                <p class="desc">{{food.description}}</p>
                <div class="extra">
                  <span class="count">月售{{food.sellCount}}</span><span>好评率{{food.rating}}</span>
                </div>
                <div class="price">
                  <span class="now">¥{{food.price}}</span>
                  <span class="old" v-show="food.oldPrice">¥{{food.oldPrice}}</span>
                </div>
              </div>
            </li>
          </ul>
        </li>
      </ul>
    </div>
  </div>
</template>

<script type="text/ecmascript-6">
 import BScorll from 'better-scroll';
 const ERR_OK = 0;

 export default {
   props: {
     seller: {
       type: Object
     }
   },
   data() {
      return {
        goods: [], // 所有商品的大分类
        listHeight: [], // 用来存放右侧每一个分类/li的商品高度
        scrollY: 0 // 当期滚动的高度
     };
   },
   computed: {
     currentIndex() { // 计算当前右侧的距离，决定决定左侧哪一个分类高亮（选中的意思）
        for (let i = 0; i < this.listHeight.length; i++) {
          // 这两个高度就是一个li从上到下整个的高度
          let height1 = this.listHeight[i]; // 获取当前的上高度
          let height2 = this.listHeight[i + 1]; // 获取当前的下高度
          // if (this.scrollY > height1 && this.scrollY < height2)  这样子height2会超出listHeight的索引，需要加一个条件
          console.log('y:' + this.scrollY);
          console.log(height1 + '....' + height2);
          if (!height2 || (this.scrollY >= height1 && this.scrollY < height2)) {
            return i; // 返回当前索引
          }
        }
        return 0; // 如果什么的都没有就返回 0
      }
   },
   created() {
     this.classMap = ['decrease', 'discount', 'special', 'invoice', 'guarantee'];
     this.$http.get('/api/goods').then(response => {
        response = response.body;
        if (response.errno === ERR_OK) {
          this.goods = response.data;
          this.$nextTick(() => { // 异步更新，当数据加载完之后再去更新dom
            // 调用滚动的初始化方法
            this._initScroll();
            // this._calculateHeight();
          });
        }
      });
   },
   methods: {
     _initScroll() {
       this.menuScroll = new BScorll(this.$refs.menuWrapper, {});
       this.foodsScroll = new BScorll(this.$refs.foodsWrapper, {
         probeType: 3
       });
       this.foodsScroll.on('scroll', (pos) => {
         this.scrollY = Math.round(pos.y);
       });
     },
     _calculateHeight() {
       let foodList = this.$refs.foodsWrapper.getElementsByClassName('foor-list-hook');
       let height = 0;
       this.listHeight.push(height);
       for (let i = 0; i < foodList.length; i++) {
         let item = foodList[i];
         height += item.clientHeight;
         this.listHeight.push(height);
       }
     }
   }
 };
</script>

<style lang="stylus" rel="stylesheet/stylus">
  @import "../../common/stylus/mixin";
  .goods
    display: flex
    position: absolute
    top: 174px
    bottom: 46px
    width: 100%
    overflow: hidden
    .menu-wrapper
      flex: 0 0 80px
      width:80px;
      background: #f3f5f7
      .menu-item
        display: table
        height: 54px
        width: 56px
        line-height: 14px
        padding: 0 12px
        &.current
          position: relative
          margin-top: 1px
          z-index: 10
          background: #ffffff
          font-weight: 700
          .text
            border: none
        .icon
          display: inline-block
          width: 12px
          height: 12px
          vertical-align: top
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
          display: table-cell
          width: 56px
          vertical-align: middle
          font-size: 12px
          border-1px(rgba(7, 17, 27, 0.1))
    .foods-wrapper
      flex: 1
      .title
        padding-left: 14px
        height: 26px
        line-height: 26px
        border-left: 2px solid #d9dde1
        font-size: 12px
        color: rgb(147,153,159)
        background: #f3f5f7
      .food-item
        display: flex
        margin: 18px
        border-1px(rgba(7,17,27,0.1))
        &:last-child
          border-none()
          margin-bottom: 0
        .icon
          flex: 0 0 57px
          margin-right: 10px
        .content  // 右侧内容
          flex: 1
          .name
            margin: 2px 0 8px 0
            height: 14px
            line-height: 14px
            font-size: 14px
            color: rgb(7,17,27)
          .desc,.extra
            line-height: 10px
            font-size: 10px
            color: #93999f
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
              margin-right: 8px
              font-size: 14px
              color: rgb(240,20,20)
            .old
              text-decoration: line-through
              font-size: 10px
              color: rgb(147,153,159)
</style>