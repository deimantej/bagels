<template>
  <v-container fluid >

    <v-snackbar
      v-model="addCheckoutItem"
    >
      {{ updatedStatus }}
      <v-btn
        color="pink"
        text
        @click="addCheckoutItem = false"
      >
        Close
      </v-btn>
    </v-snackbar>

    
<!---------  floating button  --------->
    <div id="shopping_cart" class="text-center">
    <v-menu
      v-model="menu"
      :close-on-content-click="false"
      :nudge-width="200"
      offset-x
    >
      <template v-slot:activator="{ on }">
        <v-btn

          fixed
          fab
          left
          mid
          color="#a25054"
          dark
          v-on="on"
        >
          <v-icon>mdi-cart</v-icon>
        </v-btn>
      </template>

      <v-card>
        <v-list>
          <v-list-item>
            <v-icon color="#a25054">mdi-cart</v-icon>

            <v-list-item-content>
              <v-list-item-title>Current basket</v-list-item-title>
            </v-list-item-content>
          </v-list-item>
        </v-list>
<div class="pa-2" id="info">
          <v-simple-table id="menu-table" v-if="basket.length > 0">
            <thead>
              <tr>
                <th class="text-left" style="width:30%">Number of travelers</th>
                <th class="text-left" style="width:50%">Destination</th>
                <th class="text-left" style="width:20%">Price</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="item in basket" :key="item.name">
                <td>
                  <v-icon color="#a25054" @click="increaseQtn(item)">add_box</v-icon>
                  {{ item.quantity }}
                  <v-icon color="#a25054" @click="decreaseQtn(item)">indeterminate_check_box</v-icon>
                </td>
                <td>{{ item.name }}</td>
                <td>{{item.price}}</td>
              </tr>
            </tbody>
          </v-simple-table>

          <v-simple-table v-else>
            <p>The basket is empty</p>
          </v-simple-table>

          <v-divider></v-divider>
          <v-row id="basket_checkout" class="mt-4" style="margin:0;">
            <v-col>
              <p>Subtotal:</p>
              <p>Tax for charity:</p>
              <p>Total amount:</p>
            </v-col>
            <v-col class="text-right">
              <p>{{subTotal}} DKK</p>
              <p>10 DKK</p>
              <p>
                <b>{{total}} DKK</b>
              </p>
            </v-col>
          </v-row>
          <v-row style="margin:0;">
            <v-spacer></v-spacer>
            <v-btn dark color="#a25054" @click="addCheckoutItem()" to="/about">Checkout</v-btn>
          </v-row>
        </div>

        
      </v-card>
    </v-menu>
  </div>


  <!------------------------  travel cards ------------------------------->
    <v-layout class="text-center" >
      <v-flex>
        <h1>Travel packages</h1>
      </v-flex>
    
    </v-layout>

    
    <v-layout>
    <v-row cols="12"
        justify="center">
        
        <v-card  id="item_box" class="ma-5" max-width="350" v-for="item in menuItems" :key="item.name">

          <v-img class="white--text align-end"
            height="200px" v-bind:src="item.image">
            <v-card-title> 
              <span id="td_name">{{ item.name }}</span> 
              </v-card-title>
          </v-img>



          <v-card-text class="text--primary pa-3">
             <span id="menu_item_description">{{ item.description }}</span>
          </v-card-text>

          <v-card-subtitle class="pa-3">
             <span id="menu_item_price"> price DKK: {{ item.price }}</span>
          </v-card-subtitle>

          <v-card-actions>
            <v-btn text small v-on:click="addToBasket(item)">
              <v-icon left color="#a25054">mdi-cart</v-icon> Add to cart
            </v-btn>
            <v-spacer></v-spacer>
            <v-btn text small v-on:click="addToWishlist(item)">
               <v-icon  color="red">mdi-heart</v-icon>
            </v-btn>

            





          </v-card-actions>
        </v-card>



    </v-row>
    </v-layout>


     

  </v-container>
</template>

<script>
import { dbMenuAdd } from "../../firebase";

export default {
  data() {
    return {
      basketDump: [],
      addCheckoutItem: false,
      updatedStatus: 'Travel package has been added'
    };
  },
  beforeCreate() {
    this.$store.dispatch("setMenuItems");
  },
  methods: {
    addCheckoutItem() {
      this.$store.dispatch("setCheckoutItem");
    },
    addToBasket(item) {
      /*    if (this.basket.find(itemInArray => item.name === itemInArray.name)) {
        item = this.basket.find(itemInArray => item.name === itemInArray.name);
        this.increaseQtn(item);
      } else {
        this.basket.push({
          name: item.name,
          price: item.price,
          quantity: 1
        });
      } */
      this.basketDump.push({
        name: item.name,
        price: item.price,
        quantity: 1
      });
      this.$store.commit("addBasketItems", this.basketDump);
      this.basketDump = [];
    },
/*    addToWishlist(item) {
      this.wishlistDump.push({
        name: item.name,
        price: item.price,
        quantity: 1
      });
      this.$store.commit("addWishlistItems", this.wishlistDump);
      this.wishlistDump = [];
    },   */
    increaseQtn(item) {
      item.quantity++;
    },
    decreaseQtn(item) {
      item.quantity--;

      if (item.quantity === 0) {
        this.basket.splice(this.basket.indexOf(item), 1);
      }
    }
  },
  computed: {
    basket() {
      // return this.$store.state.basketItems
      return this.$store.getters.getBasketItems;
    },
    menuItems() {
      return this.$store.getters.getMenuItems;
    },
    subTotal() {
      var subCost = 0;
      for (var items in this.basket) {
        var individualItem = this.basket[items];
        subCost += individualItem.quantity * individualItem.price;
      }
      return subCost;
    },
    total() {
      var deliveryPrice = 10;
      var totalCost = this.subTotal;
      return totalCost + deliveryPrice;
    }
  }
};
</script>




<style lang="scss" scoped>


#menu_item_price {
  font-style: italic;
  font-weight: 300;
  color: map-get($colorz, darkgrey);
  font-size: 13px;

}
#td_menuitem_img {
  max-width: 40px;
  max-height: 40px;
  padding: 0;
}
#basket_checkout {
  font-size: 13px;
}
#basket_checkout p:first-child {
  line-height: 2px;
}

#item_box{
  max-height: 6000px;
}
</style>