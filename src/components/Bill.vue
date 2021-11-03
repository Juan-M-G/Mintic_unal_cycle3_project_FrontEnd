<template>
  <div v-if="loaded" class="information">
    <h1>Información de su factura</h1>
    <h3>
      Número de factura: <span>{{ id_bill }}</span>
    </h3>
    <h3>
      Vendedor: <span>{{ user }}</span>
    </h3>
    <h3>
      Cliente: <span>{{ client_name }}</span>
    </h3>
    <h3>
      Fecha de compra: <span>{{ purchase_date }}</span>
    </h3>
    <!-- <h3>
      Productos: <span>{{ product }}</span>
    </h3> -->
    <table class="table">
        <tr>
            <th>Producto</th>
            <th>Cantidad</th>
            <th>Precio Unidad</th>
            <th>Precio total</th>
        </tr>
        <tr v-for="pro in product" v-bind:key = "pro">
            <td v-text="pro.product_name"> </td>
            <td v-text="pro.product_amount"> </td>
            <td v-text="pro.product_price"> </td>
            <td v-text="pro.sub_total_price"> </td>
        </tr>
    </table>
    <h3>
      Valor total a cancelar: <span>{{ total_bill }}</span>
    </h3>
  </div>
</template>
<script>
import axios from "axios";
export default {
  name: "Bill",
  data: function () {
    return {
      id_bill: "",
      user: "",
      client_name: "",
      purchase_date: "",
      product: "",
      total_bill: "",
      loaded: false,
    };
  },
  methods: {
    getData: async function () {
      if (
        localStorage.getItem("token_access") === null ||
        localStorage.getItem("token_refresh") === null
      ) {
        this.$emit("logOut");
        return;
      }
      //definir que recoja el id a consultar
      await this.verifyToken();
      let token = localStorage.getItem("token_access");
      //cambiar user_id por el bill_id
      let billId = localStorage.selectId
      axios
        .get(`https://misiontic--bankbe-grupo6-p67.herokuapp.com/bill/${billId}/`, {
          headers: { Authorization: `Bearer ${token}` },
        })
        .then((result) => {
          console.log(result.data);
          this.id_bill = result.data.id_bill;
          this.user = result.data.user;
          this.client_name = result.data.client_name;
          this.purchase_date = result.data.purchase_date;
          this.product = result.data.product;
          this.total_bill = result.data.total_bill;
          this.loaded = true;
        })
        .catch((error) => {
          if (error.response.status == "404")
            alert("ERROR 404: La factura que busca no extiste.");
            this.$router.push({ name: "home" });
        });
    },
    verifyToken: function () {
      return axios
        .post(
          "https://misiontic--bankbe-grupo6-p67.herokuapp.com/refresh/",
          { refresh: localStorage.getItem("token_refresh") },
          { headers: {} }
        )
        .then((result) => {
          localStorage.setItem("token_access", result.data.access);
        })
        .catch(() => {
          this.$emit("logOut");
        });
    },
  },
  created: async function () {
    console.log(localStorage)
    this.getData();
  },
};
</script>
<style>
.information {
  margin: 0;
  padding: 0%;
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}
.information h1 {
    
  font-size: 60px;
  color: #0f1316;
}
.information h3 {
  margin: 10px
}
.information span {
  color: crimson;
  font-weight: bold;
}
.table {
    text-align: center;
}

.table th{
    width: 200px;
}

</style>