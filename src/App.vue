<template>
  <div id="app">
    <h1>Online Shop 後台</h1>
    <h2 v-if="firstButtonListForm === 'sellerList'">{{ loginInfo['Account'] }}</h2>
    <!-- 第一層 -->
    <div>
      <div>
        <!-- 登陸選單 -->
        <button v-on:click="setSecondButton('Login')" v-if="firstButtonListForm === 'LoginOrRegister'">登录</button>
        <button v-on:click="setSecondButton('Register')" v-if="firstButtonListForm === 'LoginOrRegister'">注册</button>
      </div>

      <div v-if="firstButtonListForm === 'sellerList'">
        <!-- 賣家選單 -->
        <button v-on:click="setSecondButton('Product')">商品</button>
        <button v-on:click="setSecondButton('Order')">訂單</button>
        <button v-on:click="setSecondButton('Discount')">折扣</button>
        <button v-on:click="setSecondButton('Payment')">金流</button>
        <button v-on:click="setSecondButton('Shipping')">物流</button>
        <button v-on:click="setSecondButton('Setting')">設定</button>
        <button v-on:click="logout">登出</button>
      </div>
    </div>

    <!-- 第二層 -->
    <div>
      <div v-if="secondButtonListForm === 'Login'">
        <!-- 登录表单 -->
        <input type="text" v-model="loginInfo['Account']" placeholder="请输入用户名" />
        <br />
        <input type="password" v-model="loginInfo['Password']" placeholder="请输入密码" />
        <br />
        <button v-on:click="login">確定</button>
      </div>

      <div v-if="secondButtonListForm === 'Register'">
        <!-- 注册表单 -->
        <input type="text" v-model="registerInfo['NewAccount']" placeholder="请输入新用户名" />
        <br />
        <input type="password" v-model="registerInfo['NewPassword']" placeholder="请输入新密码" />
        <br />
        <button v-on:click="register">確定</button>
      </div>

      <div v-if="secondButtonListForm === 'Setting'">
        <!-- 設定表单 -->
        <label for="account">帳號:</label>
        <input type="text" id="account" :value="settingInfo['SettingAccount']" readonly />
        <br />
        <label for="password">密碼:</label>
        <input type="text" id="password" v-model="settingInfo['SettingPassword']" />
        <br />
        <label for="shopTitle">商店標題:</label>
        <input type="text" id="shopTitle" v-model="settingInfo['SettingShopTitle']" />
        <br />
        <button>更新</button>
      </div>
      
      <div class="box" v-if="secondButtonListForm === 'Product'">
        <!-- 商品選單 -->
        <button v-on:click="setThirdData('ProductsList'); getProducts() ">獲得全部商品</button>
        <button v-on:click="setThirdData('CreateProduct')">新增商品</button>
      </div>

      <div v-if="secondButtonListForm === 'Discount'">
        <!-- 折扣 -->
      </div>

      <div v-if="secondButtonListForm === 'Payment'">
        <!-- 金流 -->
      </div>
    </div>
    
    <!-- 第三層 -->
    <div>
      <div v-if="thirdDataForm === 'CreateProduct' && secondButtonListForm === 'Product'">
        <table class="centered-table table-container">
          <thead>
            <th>商品名稱</th>
            <th>商品介紹</th>
          </thead>
        <!-- 新增商品 -->
        <tbody>
          <th>
            <input type="text" v-model="title" placeholder="輸入商品title" />
          </th>
          <th>
            <input type="text" v-model="description" placeholder="輸入商品介紹" />
          </th>
        </tbody>
        </table>
        <br />
        <button v-on:click="createProduct">確認</button>
      </div>

      <div v-if="thirdDataForm=== 'ProductInfo' && secondButtonListForm === 'Product'">
        <table class="centered-table table-container">
        <thead>
            <tr>
              <th>ProductID</th>
              <th>Title</th>
              <th>Description</th>
            </tr>
          </thead>
        <tbody>
          <th>{{ this.product.ProductID }}</th>
          <th>
            <input type="text" id="productTitle" v-model="this.product.Title" />
          </th>
          <th>
            <input type="text" id="productDescription" v-model="this.product.Description" />
          </th>
        </tbody>
      </table>
      <button v-on:click="updateProduct">修改</button>
      </div>

      <div v-if="thirdDataForm === 'ProductsList' && secondButtonListForm === 'Product'">
        <!-- 商品清單-->
        <table class="centered-table table-container">
          <thead>
            <tr>
              <th>
                <a>刪除</a>
              </th>
              <th>Title</th>
            </tr>
          </thead>
          <tbody>
            <tr
              v-for="(product, product_id) in this.products.data"
              :key="product_id"
            >
              <td>
                <label>
                  <button v-on:click="deleteProduct(product.ProductID)">刪除</button>
                </label>
              </td>
              <td>
                <a href="#" v-on:click="getProduct(product.ProductID)">{{ product.Title }}</a>
              </td>
            </tr>
          </tbody>
        </table>
      </div>

    </div>

    <div class="popup" v-if="showPopup">
      <!-- 彈跳視窗 -->
      <span class="close" @click="setPopup">x</span>
      <p>{{ popupMessage }}</p>
    </div>

  </div>
</template>

<script>
import axios from "axios";

export default {
  data() {
    return {
      //商品列表
      products: [],
      product: {},

      //彈跳視窗訊息
      popupMessage: "",

      //登陸
      loginInfo: [
        {name:'Account', value: ''},
        {name:'Password', value: ''},
      ],

      //註冊
      registerInfo: [
        {name: 'NewAccount', value: ''},
        {name: 'NewPassword', value: ''},
      ],

      //設定
      settingInfo: [
        {name: 'SettingAccount', value: ''},
        {name: 'SettingPassword', value: ''},
        {name: 'SettingShopTitle', value: ''},
      ],

      firstButtonListForm: 'LoginOrRegister',
      secondButtonListForm: '',
      thirdDataForm: '',

      showPopup: false
    };
  },

  methods: {
    initializeData(){
      for (let i = 0; i < this.loginInfo.length; i++){
        this.loginInfo[i].value = ''
      }
      for (let i = 0; i < this.registerInfo.length; i++){
        this.registerInfo[i].value = ''
      }
      for (let i = 0; i < this.settingInfo.length; i++){
        this.settingInfo[i].value = ''
      }
      this.products = []
    },
    setFirstButton(button){
      this.firstButtonListForm = button
    },
    setSecondButton(button){
      this.secondButtonListForm = button
    },
    setThirdData(data){
      this.thirdDataForm = data
    },
    login() {
      axios
        .get("http://localhost:1235/api/v1/onlineShop/user?", {
          params: {
            account:  this.loginInfo['Account'],
            password: this.loginInfo['Password'],
          }})
        .then((response) => {
          const { code, msg, data } = response.data;
          console.log('msg:',msg, 'data:',data)
          if (code == 200) {
            this.setFirstButton('sellerList');
            this.setSecondButton('');
          } else {
            console.log("登入失敗");
            (this.popupMessage = "登入失敗"), this.setPopup();
          }
        })
        .catch((error) => {
          console.error("error", error);
        });
    },
    register() {
      axios
        .post("http://localhost:1235/api/v1/onlineShop/user?", null, {
          params: { 
          account: this.registerInfo['NewAccount'],
          password: this.registerInfo['NewPassword'],
         }
        })
        .then((response) => {
          const { code, msg, data } = response.data;
          console.log('msg:',msg, 'data:',data)
          if (code == 200) {
            this.username = this.newUsername;
            this.setFirstButton('sellerList');
            this.setSecondButton('');
          }
          if (code != 200) {
            (this.popupMessage = "註冊失敗"), this.setPopup();
          }
        })
        .catch((error) => {
          console.error("error", error);
        });
    },
    logout(){
      this.setFirstButton('LoginOrRegister');
      this.setSecondButton('');
      this.setThirdData('');
      this.initializeData();
    },
    setPopup() {
      this.showPopup = !this.showPopup;
    },
    getProducts() {
      axios
        .get("http://localhost:1235/api/v1/onlineShop/shop/productsList", {
          params: {
            account: this.loginInfo['Account'],
          }
        })
        .then((response) => {
          const { code, msg, data } = response.data;
          console.log("msg", msg,"data", data);
          if (code == 200) {
            this.products = response.data;
            console.log("products", this.products);
          }
          if (code != 200) {
            (this.popupMessage = "獲取商品列表失敗"), this.setPopup();
          }
        });
    },
    createProduct() {
      axios
        .post("http://localhost:1235/api/v1/onlineShop/shop/product?",{
          title: this.title,
          description: this.description,
        }
        ,{
          params: {
            account: this.loginInfo['Account'],
        }
        })
        .then((response) => {
          const { code, msg, data } = response.data;
          console.log("msg", msg,"data", data);
          if (code == 200) {
            (this.popupMessage = "新增商品成功"), this.setPopup();
          }
          if (code != 200) {
            (this.popupMessage = "新增商品失敗"), this.setPopup();
          }
        })
        .catch((error) => {
          console.error("error", error);
        });
    },
    updateProduct(){
      axios.put("http://localhost:1235/api/v1/onlineShop/shop/product?", {
        product_id: this.product.ProductID,
        title: this.product.Title,
        description: this.product.Description,
      }, {
        params: {
          account: this.loginInfo['Account']
        }
      })
      .then((response)=>{
        const {code, msg, data} = response.data;
        console.log("msg", msg,"data", data);
        if (code == 200){
          this.setThirdData('ProductsList');
          this.getProducts();
          (this.popupMessage = "更新商品成功"), this.setPopup();
        }
        if (code != 200){
          (this.popupMessage = "更新商品失敗"), this.setPopup();
        }
      })
    },
    deleteProduct(product_id){
      axios.delete("http://localhost:1235/api/v1/onlineShop/shop/product?",{
        params:{
          account: this.loginInfo['Account']
        }, 
        data: {
          product_id: product_id,
        }
      })
      .then((response) => {
        const {code, msg, data} = response.data;
        console.log("msg", msg,"data", data);
        if (code == 200) {
          this.setThirdData('ProductsList');
          this.getProducts();
          (this.popupMessage = "刪除商品成功"), this.setPopup();
        }
        if (code != 200) {
            (this.popupMessage = "刪除商品失敗"), this.setPopup();
          }
      })
    },
    getProduct(product_id){
      console.log('id', product_id)
      axios.get("http://localhost:1235/api/v1/onlineShop/shop/product?", {
        params: {
          product_id: product_id,
        }
      })
      .then((response) => {
        const {code, msg, data} = response.data;
        console.log("msg", msg,"data", data);
        if (code == 200) {
            this.product.Title = data.Title,
            this.product.Description = data.Description
            this.product.ProductID = data.ProductID
            this.product.Created_at = data.Created_at
            console.log(this.product)
            this.setThirdData('ProductInfo')
        }
        if (code != 200) {
            (this.popupMessage = "連接商品失敗"), this.setPopup();
          }
      })
    },
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

.popup {
  position: fixed;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  background-color: #ffffff;
  padding: 20px;
  border: 1px solid #ccc;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
}

.close {
  position: absolute;
  top: 10px;
  right: 10px;
  cursor: pointer;
}

p {
  margin-top: 20px;
}
.input-container {
  display: flex;
  flex-direction: column;
  align-items: center;
}
.label {
  margin-right: 10px;
}
.centered-table {
  margin: 0 auto; /* 將表格置中 */
}
.table-container {
  width: 100%;
  border-collapse: collapse;
}

.table-container th,
.table-container td {
  border: 1px solid #000;
  padding: 8px;
}
</style>
