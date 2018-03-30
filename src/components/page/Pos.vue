<template>
  <div class="pos">
    <el-row>
      <el-col :span='7' class='order' id="orderList">
        <el-tabs class="tab">
          <el-tab-pane label="点餐">
            <el-table :data="tableData" border style="width: 100%">
              <el-table-column prop="goodsName" label="商品"></el-table-column>
              <el-table-column prop="count" label="数量" width="50"></el-table-column>
              <el-table-column prop="price" label="金额" width="70"></el-table-column>
              <el-table-column label="操作" width="100" fixed="right">
                <template slot-scope="scope">
                  <el-button type="text" size="small" @click="delSinge(scope.row)">删除</el-button>
                  <el-button type="text" size="small" @click="addOrderList(scope.row)">增加</el-button>
                </template>
              </el-table-column>
            </el-table>
            <div class="tatalRow">
              <small>数量：</small>{{totalCount}} &nbsp;&nbsp;&nbsp;&nbsp;
              <small>金额：</small>{{totalMoney}}
            </div>
            <div class="btn">
              <el-button type="warning">挂单</el-button>
              <el-button type="danger" @click="delAll()">删除</el-button>
              <el-button type="success" @click="checkout()">结账</el-button>
            </div>
          </el-tab-pane>
          <el-tab-pane label="挂单">
            挂单
          </el-tab-pane>
          <el-tab-pane label="外卖">
            外卖
          </el-tab-pane>
        </el-tabs>
      </el-col>
      <el-col :span='17'>
        <div class="title">常用商品</div>
        <div class="often-goods-list">
          <ul>
            <li v-for="goods in oftenGoods" v-bind:key="goods.id" @click="addOrderList(goods)">
              <span>{{goods.goodsName}}</span>
              <span class="o-price">￥{{goods.price}}元</span>
            </li>
          </ul>
        </div>
        <div class="goods-type">
          <el-tabs>
            <el-tab-pane label="汉堡">
              <ul class='cookList'>
                <li v-for="goods in type0Goods" v-bind:key="goods.goodsId" @click="addOrderList(goods)">
                  <span class="foodImg"><img :src="goods.goodsImg" width="100%"></span>
                  <span class="foodName">{{goods.goodsName}}</span>
                  <span class="foodPrice">￥{{goods.price}}元</span>
                </li>
              </ul>
            </el-tab-pane>
            <el-tab-pane label="小食">
              <ul class='cookList'>
                <li v-for="goods in type1Goods" v-bind:key="goods.goodsId" @click="addOrderList(goods)">
                  <span class="foodImg"><img :src="goods.goodsImg" width="100%"></span>
                  <span class="foodName">{{goods.goodsName}}</span>
                  <span class="foodPrice">￥{{goods.price}}元</span>
                </li>
              </ul>
            </el-tab-pane>
            <el-tab-pane label="饮料">
              <ul class='cookList'>
                <li v-for="goods in type2Goods" v-bind:key="goods.goodsId" @click="addOrderList(goods)">
                  <span class="foodImg"><img :src="goods.goodsImg" width="100%"></span>
                  <span class="foodName">{{goods.goodsName}}</span>
                  <span class="foodPrice">￥{{goods.price}}元</span>
                </li>
              </ul>
            </el-tab-pane>
            <el-tab-pane label="套餐">
              <ul class='cookList'>
                <li v-for="goods in type3Goods" v-bind:key="goods.goodsId" @click="addOrderList(goods)">
                  <span class="foodImg"><img :src="goods.goodsImg" width="100%"></span>
                  <span class="foodName">{{goods.goodsName}}</span>
                  <span class="foodPrice">￥{{goods.price}}元</span>
                </li>
              </ul>
            </el-tab-pane>
          </el-tabs>
        </div>
      </el-col>
    </el-row>
  </div>
</template>

<script>
import axios from "axios";
export default {
  name: "Pos",

  data() {
    return {
      tableData: [],
      oftenGoods: [],
      type0Goods: [],
      type1Goods: [],
      type2Goods: [],
      type3Goods: [],
      totalMoney: 0,
      totalCount: 0
    };
  },
  created: function() {
    axios
      .get("http://jspang.com/DemoApi/oftenGoods.php")
      .then(Response => {
        //console.log(Response);
        this.oftenGoods = Response.data;
      })
      .catch(Error => {
        alert("网络错误，不能访问!");
      });

    axios
      .get("http://jspang.com/DemoApi/typeGoods.php")
      .then(Response => {
        //console.log(Response);
        this.type0Goods = Response.data[0];
        this.type1Goods = Response.data[1];
        this.type2Goods = Response.data[2];
        this.type3Goods = Response.data[3];
      })
      .catch(Error => {
        alert("网络错误，不能访问!");
      });
  },
  mounted: function() {
    var orderHeight = document.body.clientHeight;
    document.getElementById("orderList").style.height = orderHeight + "px";
  },
  methods: {
    addOrderList: function(goods) {
      //判断商品列表内是否存在
      let isH = false;
      for (let i = 0; i < this.tableData.length; i++) {
        if (goods.goodsName == this.tableData[i].goodsName) {
          isH = true;
        } else {
          isH = false;
        }
      }
      //根据商品存在状态修改订单列表
      if (isH) {
        let arr = this.tableData.filter(o => o.goodsId == goods.goodsId);
        arr[0].count++;
      } else {
        let newGoods = {
          goodsId: goods.goodsId,
          goodsName: goods.goodsName,
          count: 1,
          price: goods.price
        };
        this.tableData.push(newGoods);
      }
      this.sum();
    },
    delSinge: function(goods) {
      this.tableData = this.tableData.filter(o => o.goodsId != goods.goodsId);
      this.sum();
    },
    delAll: function() {
      //清零
      this.totalCount = 0;
      this.totalMoney = 0;
      this.tableData = [];
    },
    sum: function() {
      //清零
      this.totalCount = 0;
      this.totalMoney = 0;
      //合计
      this.tableData.forEach(function(element) {
        this.totalCount += element.count;
        this.totalMoney = this.totalMoney + element.price * element.count;
      }, this);
    },
    checkout: function() {
      if (this.totalCount != 0) {
        this.delAll();
        this.$message({
          message: "结账成功",
          type: "success"
        });
      } else {
        this.$message.error("订单列表为空，结账失败");
      }
    }
  }
};
</script>
<style scoped>
.order {
  background: #f9fafc;
  border-right: 1px solid #c0ccda;
}

.btn {
  margin-top: 10px;
}

.title {
  height: 20px;
  border-bottom: 1px solid #d3dce6;
  background-color: #f9fafc;
  padding: 10px;
}

.often-goods-list ul li {
  list-style: none;
  float: left;
  border: 1px solid #e5e9f2;
  padding: 10px;
  margin: 5px;
  background-color: #fff;
  cursor: pointer;
}

.o-price {
  color: #58b7ff;
}

.goods-type {
  clear: both;
}

.cookList li {
  list-style: none;
  width: 23%;
  border: 1px solid #e5e9f2;
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
  font-size: 18px;
  padding-left: 10px;
  color: brown;
  word-wrap: break-word;
  width: 50%;
  white-space: normal;
  text-align: left;
}

.foodPrice {
  font-size: 16px;
  padding-left: 10px;
  padding-top: 10px;
}

.tatalRow {
  background-color: #fff;
  padding: 10px;
  border-bottom: 1px solid #e5e9f2;
}
</style>