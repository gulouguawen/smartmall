<template>
    <div>
        <nav-header></nav-header>
        <nav-bread>
            <span>Goods</span>
        </nav-bread>
        <div class="accessory-result-page accessory-page">
            <div class="container">
                <div class="filter-nav">
                    <span class="sortby">Sort by:</span>
                    <a href="javascript:void(0)" class="default cur">
                        Default</a>
                    <a @click="sortGoods()" href="javascript:void(0)" class="price">
                        Price
                        <svg class="icon icon-arrow-short">
                            <use xlink:href="#icon-arrow-short"></use>
                        </svg>
                    </a>
                    <a href="javascript:void(0)"
                       class="filterby stopPop"
                       @click="showFilterPop">
                        Filter by
                    </a>
                </div>
                <div class="accessory-result">
                    <!-- filter -->
                    <div class="filter stopPop"
                         id="filter"
                         :class="{'filterby-show':filterBy}">
                        <dl class="filter-price">
                            <dt>Price:</dt>
                            <dd>
                                <a href="javascript:void(0)"
                                   :class="{ 'cur' : priceChecked==='all'}"
                                   @click="setPriceFilter('all')">All
                                </a>
                            </dd>
                            <dd v-for="(price,index) in priceFilter"
                                :key="index">
                                <a href="javascript:void(0)"
                                   :class="{'cur':priceChecked===index}"
                                   @click="setPriceFilter(index)"
                                >
                                    {{price.startPrice}}- {{price.endPrice}}
                                </a>
                            </dd>
                        </dl>
                    </div>
                    <!-- search result accessories list -->
                    <div class="accessory-list-wrap">
                        <div class="accessory-list col-4">
                            <ul>
                                <li v-for="(item,index) in goodsList" :key="index">
                                    <div class="pic">
                                        <a href="#">
                                            <img v-lazy="'/static/'+item.productImage" alt="">
                                        </a>
                                    </div>
                                    <div class="main">
                                        <div class="name">{{item.productName}}</div>
                                        <div class="price">{{item.salePrice}}</div>
                                        <div class="btn-area">
                                            <a href="javascript:;" class="btn btn--m" @click="addCart(item.productId)">加入购物车</a>
                                        </div>
                                    </div>
                                </li>
                            </ul>
                            <div class="view-more-normal"
                                 v-infinite-scroll="loadMore"
                                 infinite-scroll-disabled="busy"
                                 infinite-scroll-distance="20">
                                <img src="./../../static/loading-svg/loading-spokes.svg" v-show="loading"
                                     alt="loading...">
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
        <div class="md-overlay" v-show="overLayFlag" @click="closeFilterPop"></div>
        <nav-footer></nav-footer>
    </div>
</template>

<script>
import '../assets/css/base.css'
import './../assets/css/product.css'
import NavHeader from '../components/NavHeader'
import NavFooter from '../components/NavFooter'
import NavBread from '../components/NavBread'
import axios from 'axios'

export default {
  name: 'GoodsList',
  data () {
    return {
      goodsList: [],
      sortFlag: true,
      page: 1,
      pageSize: 8,
      busy: true,
      priceLevel: this.priceChecked,
      loading: true,
      priceFilter: [
        {
          startPrice: '0',
          endPrice: '100'
        },
        {
          startPrice: '100',
          endPrice: '500'
        },
        {
          startPrice: '500',
          endPrice: '1000'
        },
        {
          startPrice: '1000',
          endPrice: '2000'
        },
        {
          startPrice: '2000',
          endPrice: '5000'
        }
      ],
      priceChecked: 'all',
      filterBy: false,
      overLayFlag: false
    }
  },
  components: {
    NavHeader,
    NavFooter,
    NavBread
  },
  methods: {
    getGoodsList (flag) {
      // axios.get('/static/mock/goods.json').then((result) => {
      var param = {
        page: this.page,
        pageSize: this.pageSize,
        sort: this.sortFlag ? 1 : -1,
        priceLevel: this.priceChecked
      }
      this.loading = false
      axios.get('/goods', {params: param}).then((result) => {
        let res = result.data
        this.loading = true
        if (res.status === '0') {
          if (flag) {
            this.goodsList = this.goodsList.concat(res.result.list)
            this.busy = res.result.count === 0
          } else {
            this.goodsList = res.result.list
            this.busy = false
          }
        } else {
          this.goodsList = []
        }
      })
    },
    showFilterPop () {
      this.filterBy = true
      this.overLayFlag = true
    },
    closeFilterPop () {
      this.filterBy = false
      this.overLayFlag = false
    },
    setPriceFilter (index) {
      this.priceChecked = index
      this.page = 1
      this.getGoodsList()
    },
    sortGoods () {
      this.sortFlag = !this.sortFlag
      this.page = 1
      this.getGoodsList()
    },
    loadMore () {
      this.busy = true
      setTimeout(() => {
        this.page++
        this.getGoodsList(true)
      }, 400)
    },
    addCart (productId) {
      axios.post('/goods/addCart', {
        productId: productId
      }).then((res) => {
        if (res.status === 1) {
          alert('add success')
        } else {
          alert('msg:' + res.message)
        }
      })
    }
  },
  mounted () {
    this.getGoodsList()
  }
}
</script>
