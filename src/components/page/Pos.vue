<template>
  <div class="pos">
    <el-row>
      <el-col :span="10" class="pos-order" id="order-list">
        <el-tabs>
          <el-tab-pane label="点餐">
            <el-table :data="tableDate" border style="width:100%;">
              <el-table-column prop="goodsName" label="商品名称"></el-table-column>
              <el-table-column prop="count" label="数量" style="width:50px"></el-table-column>
              <el-table-column prop="price" label="金额" style="width:70px"></el-table-column>
              <el-table-column label="操作" style="width:100px" fixed="right">
                <template scope="scope">
                  <el-button type="text" size="small" @click="delSingleGoods(scope.row)">删除</el-button>
                  <el-button type="text" size="small" @click="addOrderList(scope.row)">增加</el-button>
                </template>
              </el-table-column>
            </el-table>
            <div class="totalDiv">
              <small>数量： </small><span>{{totalCount}}</span>
              <small>金额：</small><span>{{totalMoney}}元</span>
            </div>
            <div class="divBtn">
              <el-button type="warning">挂单</el-button>
              <el-button type="danger" @click="delAllGoods">删除</el-button>
              <el-button type="success" @click="checkout">结账</el-button>
            </div>
          </el-tab-pane>
          <el-tab-pane label="挂单">挂单</el-tab-pane>
          <el-tab-pane label="外卖">外卖</el-tab-pane>
        </el-tabs>
      </el-col>
      <el-col :span="14">
        <div class="often-goods">
          <div class="title">常用商品</div>
          <div class="often-goods-list">
            <ul>
              <li v-for="goods in oftenGoods" @click="addOrderList(goods)">
                <span>{{goods.goodsName}}</span>
                <span class="o-price">{{goods.price | money("元","￥")}}</span>
              </li>
            </ul>
          </div>
        </div>
        <div class="goods-type">
          <el-tabs type="border-card">
            <el-tab-pane label="汉堡">
              <div>
                <ul class='cookList'>
                  <li v-for="goods in type0Goods" @click="addOrderList(goods)">
                    <span class="foodImg"><img :src="goods.goodsImg" width="100%"></span>
                    <span class="foodName">{{goods.goodsName}}</span>
                    <span class="foodPrice">{{goods.price | money("元","￥")}}</span>
                  </li>
                </ul>
              </div>
            </el-tab-pane>
            <el-tab-pane label="小食">
              <div>
                <ul class='cookList'>
                  <li v-for="goods in type1Goods" @click="addOrderList(goods)">
                    <span class="foodImg"><img :src="goods.goodsImg" width="100%"></span>
                    <span class="foodName">{{goods.goodsName}}</span>
                    <span class="foodPrice">{{goods.price | money("元","￥")}}</span>
                  </li>
                </ul>
              </div>
            </el-tab-pane>
            <el-tab-pane label="饮料">
              <div>
                <ul class='cookList'>
                  <li v-for="goods in type2Goods" @click="addOrderList(goods)">
                    <span class="foodImg"><img :src="goods.goodsImg" width="100%"></span>
                    <span class="foodName">{{goods.goodsName}}</span>
                    <span class="foodPrice">{{goods.price | money("元","￥")}}</span>
                  </li>
                </ul>
              </div>
            </el-tab-pane>
            <el-tab-pane label="套餐">
              <div>
                <ul class='cookList'>
                  <li v-for="goods in type3Goods" @click="addOrderList(goods)">
                    <span class="foodImg"><img :src="goods.goodsImg" width="100%"></span>
                    <span class="foodName">{{goods.goodsName}}</span>
                    <span class="foodPrice">{{goods.price | money("元","￥")}}</span>
                  </li>
                </ul>
              </div>
            </el-tab-pane>
          </el-tabs>
        </div>
      </el-col>
    </el-row>
  </div>
</template>
<script>
import axios from 'axios'
export default {
  name: "Pos",
  data() {
    return {
      tableDate: [],
      oftenGoods: [],
      type0Goods: [],
      type1Goods: [],
      type2Goods: [],
      type3Goods: [],
      totalMoney: 0,
      totalCount: 0
    }
  },
  created: function() {
    axios.get("http://jspang.com/DemoApi/oftenGoods.php")
      .then(reponse => {
        this.oftenGoods = reponse.data;
        // console.log(reponse);
      })
      .catch(error => {
        // console.log(error);
        alert("万络错误，不能访问");
      })
    axios.get("http://jspang.com/DemoApi/typeGoods.php")
      .then(reponse => {
        this.type0Goods = reponse.data[0];
        this.type1Goods = reponse.data[1];
        this.type2Goods = reponse.data[2];
        this.type3Goods = reponse.data[3];
        // for(var i=0;i<reponse.data.length;i++){
        // 	this.type0Goods =reponse.data[i];
        // }
      })
      .catch()
  },
  filters: {
    money: function(yuan, kind, symbol) {

      if (!yuan) {
        return "暂无价格"
      } else {
        return symbol + yuan + kind;
      }
    }
  },
  mounted: function() {
    var orderHeight = document.body.clientHeight;
    document.getElementById("order-list").style.height = orderHeight + "px";
  },
  methods: {
    addOrderList(goods) {
      this.totalMoney = 0;
      this.totalCount = 0;
      let isHave = false;
      //商品是否已经存在于订单列表中
      for (let i = 0; i < this.tableDate.length; i++) {
        if (this.tableDate[i].goodsId == goods.goodsId) {
          isHave = true;
        }
      }
      //根据判断的知缩写业务逻辑
      if (isHave) {
        //改变列表中商品数量
        let arr = this.tableDate.filter(o => o.goodsId == goods.goodsId);
        arr[0].count++;
      } else {
        let newGoods = { goodsId: goods.goodsId, goodsName: goods.goodsName, price: goods.price, count: 1 };
        this.tableDate.push(newGoods);
      }
      this.getAllMoney();
    },
    //删除单个商品
    delSingleGoods(goods) {
      this.tableDate = this.tableDate.filter(o => o.goodsId != goods.goodsId);
      this.getAllMoney();
    },
    //清空购物车
    delAllGoods() {
      this.tableDate = [];
      this.totalMoney = 0;
      this.totalCount = 0;
    },
    //模拟结账
    checkout() {
      if (this.totalCount != 0) {
        this.tableDate = [];
        this.totalCount = 0;
        this.totalMoney = 0;
        this.$message({
          message: "结账成功，谢谢您的购物。祝您用餐愉快",
          type: 'success'
        })
      } else {
        this.$message.error("不能空结。老板了解你急切的心情！")
      }
    },
    //汇总数量及金额
    getAllMoney() {
      this.totalMoney = 0;
      this.totalCount = 0;
      if (this.tableDate) {
        //计算汇总价格和数量
        this.tableDate.forEach((element) => {
          this.totalCount += element.count;
          this.totalMoney = this.totalMoney + (element.price * element.count);
        })
      }
    }
  }
}

</script>
<style scoped>
.pos-order {
  background-color: #f9fafc;
  border-right: 1px solid #c0ccda;
  text-align: center;
}

.divBtn {
  margin: 10px;
}

.title {
  height: 20px;
  border-bottom: 1px solid #D3DCE6;
  background-color: #F9FAFC;
  padding: 10px;
}

.often-goods {
  padding-bottom: 20px;
}

.often-goods-list ul li {
  list-style: none;
  float: left;
  border: 1px solid #E5E9F2;
  padding: 10px;
  margin: 5px;
  background-color: #fff;
  cursor: pointer;
}

.o-price {
  color: #58B7FF;
}

.goods-type {
  height: auto;
  overflow: hidden;
  border-top: 1px solid #D3dce6;
  clear: both;
}

.cookList li {
  list-style: none;
  width: 23%;
  border: 1px solid #E5E9F2;
  height: auot;
  overflow: hidden;
  background-color: #fff;
  padding: 2px;
  float: left;
  margin: 2px;
  cursor: pointer;
}

.cookList li span {

  display: block;
  float: left;
}

.foodImg {
  width: 40%;
}

.foodName {
  font-size: 15px;
  padding-left: 10px;
  color: brown;
}

.foodPrice {
  font-size: 16px;
  padding-left: 10px;
  padding-top: 10px;
}

.totalDiv {
  background-color: #fff;
  padding: 10px;
  border-bottom: 1px solid #D3dce6;
}

.totalDiv span:nth-child(2) {
  margin-right: 10px;
}

.totalDiv small:nth-child(3) {
  margin-left: 10px;
}

</style>
