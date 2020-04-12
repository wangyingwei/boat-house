<template>
  <div class="container">
    <div class="row">
      <div class="col-md-8 orders-wrapper">
        <div class="order-title">您的订单</div>
        <div class="order-empty" v-show="orders.length === 0">您还没有创建订单</div>
        <div class="card" v-for="(order, index) in orders" :key="order.id">
          <div class="card-header" v-on:click="showOrderDetail(index)">
            <div class="row">
              <div class="col-sm-6 order-name">{{order.name}}</div>
              <div class="col-sm-4 order-time">{{order.createTime}}</div>
              <div class="col-sm-2 order-status">{{order.orderStatusDesc}}</div>
            </div>
          </div>
          <div class="order-detail collapse show" v-show="selectedOrder === order.id">
            <div class="card-body">
              <ul class="list-group">
                <li v-for="item in items" :key="item.id" class="list-group-item">
                  <div class="row">
                    <div class="col-sm-3">
                      <img :src="item.foodPicture" class="cooking-img" />
                    </div>
                    <div class="col-sm-5">
                      <div class="cooking-name">{{item.foodName}}</div>
                    </div>
                    <div class="quota col-sm-1">
                      x {{item.foodNum}}
                    </div>
                    <div class="col-sm-3 price">
                      ￥ {{formatMoney(item.foodPrice)}}
                    </div>
                  </div>
                </li>
              </ul>
              <div class="total-price">
                <span>总计：</span>
                <span class="price">￥ {{formatMoney(total)}}</span>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data () {
    return {
      orders: [],
      selectedOrder: -1
    }
  },
  computed: {
    items () {
      if (this.selectedOrder >= 0 && this.orders.length) {
        return this.orders[this.selectedOrder].itemsList
      }
      return []
    },
    total () {
      if (this.selectedOrder) {
        return this.items.reduce((acc, item) => acc + item.foodPrice * item.foodNum, 0)
      }
    }
  },
  mounted () {
    this.fetchOrder()
  },
  methods: {
    fetchOrder () {
      this.axios.get('/api/orders/currentUser').then(result => {
        if (result.status === 200) {
          const data = result.data
          this.orders = data.map(item => {
            const foods = item.itemsList
            if (foods.length) {
              if (foods.length === 1) {
                item.displayName = foods[0].foodName
              } else {
                item.displayName = foods[0].foodName + ' 等' + foods.length + '件'
              }
            }
            return item
          })
        }
      })
    },
    formatMoney (m) {
      if (m) {
        return m.toFixed(2)
      }
      return ''
    },
    showOrderDetail (index) {
      if (this.selectedOrder === index) {
        this.selectedOrder = undefined
      } else {
        this.selectedOrder = index
      }
    }
  }
}
</script>
<style scoped>
.orders-wrapper {
  margin-top: 24px;
  margin-bottom: 24px;
  min-height: calc(100vh - 490px);
}

.order-title {
  font-weight: bold;
  line-height: 30px;
}

.card {
  margin-bottom: 0;
}

.card-header {
  cursor: pointer;
}

.order-status {
  text-align: right;
}

.order-item > .row {
  align-items: center;
}

.order-detail .row {
  align-items: center;
}

.order-detail .cooking-img {
  width: 108px;
  height: 70px;
}

.order-detail .price {
  color: #f00;
  font-size: 16px;
  text-align: right;
}

.total-price {
  display: flex;
  flex-flow: row;
  justify-content: flex-end;
  margin-top: 12px;
}
</style>
