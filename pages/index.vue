<template>
  <div class="p-5 app">
    <a-row :span="24" class="text-right">
      <span class="text-30 mr-3 cursor-pointer hover:text-ocean" @click="modeRegister = true">Register</span>
      <span class="text-30 mr-3 ">|</span>
      <span class="text-30 cursor-pointer hover:text-ocean" @click="modeLogin = true">Login</span>
    </a-row>
    <a-modal
      title="Login"
      :visible="modeLogin"
      @ok="login"
      @cancel="handleCancel"
      class="modal-login"
      :maskClosable="false"
    >
      <el-input placeholder="Email" v-model="emailLogin" class="mb-2 text-20"></el-input>
      <el-input placeholder="Password" v-model="passwordLogin" class="text-20"></el-input>
    </a-modal>
    <a-modal
      title="Register"
      :visible="modeRegister"
      @ok="register"
      @cancel="handleCancel"
      class="modal-login"
      :maskClosable="false"
    >
      <el-input placeholder="Email" v-model="emailRegister" class="mb-2 text-20"></el-input>
      <el-input placeholder="Password" v-model="passwordRegister" class="text-20"></el-input>
    </a-modal>
    <a-modal
      :title="!modeEdit? 'Add Product' : 'Edit Product'"
      :visible="modeProduct"
      @ok="addproduct"
      @cancel="handleCancel"
      class="modal-login"
      :maskClosable="false"
    >
      Sku:
      <el-input placeholder="Sku" v-model="form.sku" :disabled="modeEdit?true:false" class="mb-2 text-20"></el-input>
      Product Name:
      <el-input placeholder="Product Name" v-model="form.product_name" class="text-20 mb-2"></el-input>
      Qty:
      <el-input placeholder="Qty" v-model="form.qty" class="text-20 mb-2"></el-input>
      Price:
      <el-input placeholder="Price" v-model="form.price" class="text-20 mb-2"></el-input>
      Unit:
      <el-input placeholder="Unit" v-model="form.unit" class="text-20 mb-2"></el-input>
      Status:
      <el-input placeholder="Status" v-model="form.status" class="text-20 mb-2"></el-input>
    </a-modal>
    <a-row class="mt-3">
      <a-col :span="12">
        <el-input
          placeholder="Type something"
          prefix-icon="el-icon-search"
          v-model="searchCondition"
          class="mb-4 text-18"
          clearable
          @blur="getsearchList"
          @keyup.enter.native="onSubmit"
          :style="{ width: '300px' }"
        >
        </el-input>
      </a-col>
      <a-col :span="12" v-if="isLogin">
        <h3 class="text-20 text-right cursor-pointer" @click="setModeAdd">Add Product</h3>
      </a-col>
    </a-row>
    <el-table :data="tableData" style="width: 100%;font-size: 20px">
      <el-table-column prop="sku" label="Sku"> </el-table-column>
      <el-table-column prop="product_name" label="Product Name">
      </el-table-column>
      <el-table-column prop="qty" label="Qty"> </el-table-column>
      <el-table-column prop="price" label="Price"> </el-table-column>
      <el-table-column prop="unit" label="Unit"> </el-table-column>
      <el-table-column prop="status" label="Status"> </el-table-column>
      <el-table-column label="Action" v-if="isLogin">
        <template slot-scope="scope">
          <span class="mr-3 text-lightblue cursor-pointer" @click="setDataEdit(scope.row)">Edit</span>
          <span class="text-error cursor-pointer" @click="deleteProduct(scope.row)">Delete</span>
        </template>
      </el-table-column>
    </el-table>
  </div>
</template>

<script>
export default {
  name: "IndexPage",
  data() {
    return {
      isLogin: false,
      modeLogin: false,
      modeRegister: false,
      modeProduct: false,
      modeEdit: false,
      loading: true,
      emailLogin: "",
      passwordLogin: "",
      emailRegister: "",
      passwordRegister: "",
      searchCondition:"",
      token:"",
      tableData: [],
      form: {
        sku: "",
        product_name: "",
        qty: "",
        price: "",
        unit: "",
        status: "",
      }
    };
  },
  watch: {
    searchCondition: function (val) {
      if(val == "") {
        this.getList()
      }
    }
  },
  created(){
    this.getList()
  },
  methods: {
    handleOk() {

    },
    handleCancel() {
      this.modeLogin = false
      this.modeRegister = false
      this.modeProduct = false
      this.resetMail()
    },
    async login() {
      const res = await this.$axios.$post('https://hoodwink.medkomtek.net/api/auth/login', {
        email: this.emailLogin,
        password: this.passwordLogin
      })
        .then(res => {
          if(res.token) {
            this.modeLogin = false
            this.isLogin = true
            this.token = res.token
            alert("Loggin successfully")
          }
        })
        .catch(error => {
          alert(error.response.data.error)
        })
    },
    async register() {
      const res = await this.$axios.$post('https://hoodwink.medkomtek.net/api/register', {
        email: this.emailRegister,
        password: this.passwordRegister
      })
        .then(res => {
          if(res.success == true) {
            this.modeRegister = false
            alert("Register successfully")
          }
        })
        .catch(error => {
          alert(error.response.data.email)
        })
    },
    async getList() {
      const res = await this.$axios.$get('https://hoodwink.medkomtek.net/api/items')
        .then(res => {
          this.tableData = res
          this.loading = false
        })
        .catch(error => {
          this.loading = false
          alert(error.response)
        })
    },
    async getsearchList(){
      if(this.searchCondition != ""){
        const res = await this.$axios.$post('https://hoodwink.medkomtek.net/api/item/search',{
          sku: this.searchCondition
        })
          .then(res => {
            this.tableData = []
            if(res.id) {
              this.tableData.push(res)
            }
          })
          .catch(error => {
            alert(error.response)
          })
      }
    },
    addproduct(){
      if(this.modeEdit) {
        this.editProduct()
      } else {
        let config = {
          headers: {
            'Authorization': 'Bearer ' + this.token
          }
        }
        let params = {
          sku: this.form.sku,
          product_name: this.form.product_name,
          qty: this.form.qty,
          price: this.form.price,
          unit: this.form.unit,
          status: this.form.status,
        }
        const res = this.$axios.$post('https://hoodwink.medkomtek.net/api/item/add', params, config)
          .then(res => {
            alert("Add product successfully")
            this.modeProduct = false
            this.tableData.push(res)
          })
          .catch(error => {
            alert("Input value is not correct")
          })
      }
    },
    deleteProduct(value){
      let config = {
        headers: {
          'Authorization': 'Bearer ' + this.token
        }
      }
      let params = {
        sku: value.sku,
      }
      const res = this.$axios.$post('https://hoodwink.medkomtek.net/api/item/delete', params, config)
        .then(res => {
          alert("Delete product successfully")
          this.modeProduct = false
          this.getList()
        })
        .catch(error => {
          alert("Error")
        })
    },
    editProduct(){
      let config = {
        headers: {
          'Authorization': 'Bearer ' + this.token
        }
      }
      const res = this.$axios.$post('https://hoodwink.medkomtek.net/api/item/update', this.form, config)
        .then(res => {
          alert("Edit product successfully")
          this.modeProduct = false
          this.getList()
        })
        .catch(error => {
          alert("Error")
        })
    },
    setDataEdit(value){
      this.modeProduct = true
      this.form.sku = value.sku
      this.form.product_name = value.product_name
      this.form.qty = value.qty
      this.form.price = value.price
      this.form.unit = value.unit
      this.form.status = value.status
      this.modeEdit = true
    },
    setModeAdd(){
      this.modeProduct = true
      this.resetForm()
    },
    resetForm(){
      this.form.sku = ""
      this.form.product_name = ""
      this.form.qty = ""
      this.form.price = ""
      this.form.unit = ""
      this.form.status = ""
      this.modeEdit = ""
    },
    resetMail() {
      this.emailRegister = ""
      this.passwordRegister = ""
      this.emailRegister = ""
      this.passwordRegister = ""
    },
    onSubmit(){
      this.getsearchList()
    }
  }
};
</script>
<style scoped>
  .app {
    font-size: 20px;
  }
</style>
