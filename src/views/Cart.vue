<template>
  <div class="cart m-4">
    <h3>
      Ваша корзина
      <span v-if="order.items.length == 0">пуста. Добавьте товары из каталога.</span>
      <span v-else>:</span>
    </h3>
    <div ref="orderTable">
      <table class="table" v-if="order.items.length > 0">
        <thead>
          <tr>
            <th scope="col">#</th>
            <th scope="col">Изображение</th>
            <th scope="col">Артикул</th>
            <th scope="col">Наименование</th>
            <th scope="col">Цена</th>
            <th scope="col">Кол-во</th>
            <th scope="col">Стоимость</th>
            <th scope="col">Удалить</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(item, index) in order.items" :key="index">
            <th scope="row">{{index + 1}}</th>
            <td>
              <img class="productPhoto" v-if="item.product.photos.length>0" :src="item.product.photos[0].server + item.product.photos[0].name" />
            </td>
            <td>{{ item.product.name }}</td>
            <td>{{ item.product.caption }}</td>
            <td>{{ item.product.price }}₽</td>
            <td>{{ item.quantity }}</td>
            <td>{{ item.product.price * item.quantity}}₽</td>
            <td>
              <a href="#" @click="deleteFromCart(index)">
                <font-awesome-icon icon="trash-alt" />
              </a>
            </td>
          </tr>
        </tbody>
        <thead>
          <tr>
            <th scope="col"></th>
            <th scope="col"></th>
            <th scope="col"></th>
            <th scope="col"></th>
            <th scope="col">Итого:</th>
            <th scope="col">{{cartCount}}</th>
            <th scope="col">{{cartSum}}₽</th>
            <th scope="col">
              <a href="#" @click="emptyCart()">
                <font-awesome-icon icon="trash-alt" class="text-primary" />
              </a>
            </th>
          </tr>
        </thead>
      </table>
    </div>
    <div class="container" v-if="order.items.length > 0">
      <div class="row">
        <div class="col-md-6 mb-4">
          <input
            class="form-control form-control-sm mr-3"
            type="text"
            placeholder="Ф.И.О."
            aria-label="Search"
            v-model="customerName"
          />
        </div>
        <div class="col-md-6 mb-4">
          <input
            placeholder="Номер телефона"
            type="number"
            id="exampleForm6"
            class="form-control"
            v-model="customerPhone"
          />
        </div>
      </div>
      <div class="row">
        <div class="col-md-12 text-center mb-4">
          <button @click="checkout()" class="btn btn-primary">Оформить заказ</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import pdfMake from "pdfmake/build/pdfmake";
import pdfFonts from "pdfmake/build/vfs_fonts";
pdfMake.vfs = pdfFonts.pdfMake.vfs;

export default {
  props: ["order", "apiPrefix"],
  data() {
    return {
      apiResponse: null,
      customerName: null,
      customerPhone: null
    };
  },
  methods: {
    checkout: function() {
      if (this.customerName != null && this.customerPhone != null) {
        let newOrder = {
          customerName: this.customerName,
          customerPhone: this.customerPhone
          //orderProducts: this.cartProducts
        };
        console.log(JSON.stringify(newOrder));
        this.axios
          .post(this.apiPrefix + "api/order/new", newOrder, {
            headers: { "Content-Type": "application/json" }
          })
          .then(response => {
            this.apiResponse = response.data;
          })
          .finally(() => {
            this.generateOrderDocument();
          });
      }
      this.generateOrderDocument();
    },
    generateOrderDocument: function() {
      if (this.apiResponse != null) {
        console.log(pdfMake);
        let tableContents = [];
        tableContents.push([
          "№",
          "Артикул",
          "Наименование",
          "Цена",
          "Кол-во",
          "Стоимость"
        ]);
        let i = 0;
        this.order.items.forEach(item => {
          tableContents.push([
            ++i,
            item.product["name"],
            item.product["caption"],
            item.product["price"],
            item.product["orderQuantity"],
            item.product["price"] * item["quantity"]
          ]);
        });

        let docDefinition = {
          content: [
            {
              text: `Ваш заказ №${this.apiResponse.idOrder} успешно зарегистрирован!`,
              fontSize: 30
            },
            {
              text:
                "Сохраните данный документ для предъявления продавцу при оплате заказа!",
              color: "red"
            },
            { text: `ФИО: ${this.customerName}`, fontSize: 15 },
            { text: `Номер телефона: ${this.customerPhone}`, fontSize: 15 },

            {
              layout: "lightHorizontalLines",
              table: {
                headerRows: 1,
                widths: [20, "auto", "auto", "auto", "auto", "auto"],
                body: tableContents
              }
            },
            { text: `Сумма заказа: ${this.cartSum}₽`, fontSize: 15 }
          ]
        };
        console.log(docDefinition);
        pdfMake
          .createPdf(docDefinition)
          .download(`Заказ${this.apiResponse.idOrder}.pdf`);
        this.emptyCart();
      }

      return;
    },
    deleteFromCart: function(index) {
      this.$emit("deleted-from-cart", index);
    },
    emptyCart: function() {
      this.$emit("cart-empty");
    }
  },
  computed: {
    cartSum: function() {
      let sum = 0;
      this.order.items.forEach(item => {
        sum += item.quantity * item.product.price;
      });
      return sum;
    },
    cartCount: function() {
      let count = 0;
      this.order.items.forEach(item => {
        count += item.quantity * 1;
      });
      return count;
    }
  }
};
</script>

<style>
.productPhoto {
  max-height: 50px;
}
</style>

