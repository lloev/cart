<template>
  <div class="app-container">
    <Header></Header>
    <!-- 循环渲染每个商品信息 -->
    <Goods
      v-for="item in list"
      :key="item.id"
      :id="item.goods_id"
      :title="item.goods_name"
      :price="item.goods_price"
      :pic="item.goods_img"
      :state="item.goods_state"
      :count="item.goods_count"
      @state-change="getNewState"
    ></Goods>
    <Footer
      :isfull="fullState"
      :amout="amt"
      :all="total"
      @full-change="getFullState"
    ></Footer>
  </div>
</template>

<script>
// 导入 axios 请求库
import axios from 'axios'
import bus from '@/components/eventBus'
import Header from '@/components/Header/Header'
import Goods from '@/components/Goods/Goods'
import Footer from '@/components/Footer/Footer'

export default {
  components: {
    Header,
    Goods,
    Footer,
  },

  data() {
    return {
      list: [],
    }
  },

  methods: {
    // 封装请求列表数据的方法
    async initCartList() {
      const { data: res } = await axios.get(
        'https://applet-base-api-t.itheima.net/api/cart'
      )
      // 请求回来的数据，在页面渲染期间用到的，必须转存到data中
      console.log(res)
      if (res.status === 200) {
        this.list = res.list
      }
    },
    // 接收 Goods 组件传递过来的按钮的状态
    getNewState(e) {
      console.log(e)
      this.list.some((item) => {
        if (item.goods_id === e.id) {
          item.goods_state = e.value
          // 终止后续的循环
          return true
        }
      })
    },
    // 接收 Footer 组件传递过来的全选按钮的状态
    getFullState(e) {
      this.list.forEach((item) => (item.goods_state = e))
    },
  },

  computed: {
    // 动态计算出全选的状态
    fullState() {
      return this.list.every((item) => item.goods_state)
    },
    // 已勾选商品的总价
    amt() {
      return this.list
        .filter((item) => item.goods_state)
        .reduce(
          (total, item) => (total += item.goods_price * item.goods_count),
          0
        )
    },
    // 已勾选商品的总数量
    total() {
      return this.list
        .filter((item) => item.goods_state)
        .reduce((t, item) => (t += item.goods_count), 0)
    },
  },
  created() {
    this.initCartList()
    bus.$on('share', (val) => {
      this.list.some((item) => {
        if (item.goods_id === val.id) {
          item.goods_count = val.value
          return true
        }
      })
    })
  },
}
</script>

<style lang="less" scoped>
.app-container {
  padding-top: 45px;
  padding-bottom: 50px;
}
</style>
