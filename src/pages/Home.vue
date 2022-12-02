<template>
  <div>

    <nav>
      <div class="nav-container">
        <a href="index.html">
          <img src="imgs/DBook.png" class="logonav">
        </a>

        <div class="nav-profile">
          <p class="nav-profile-name">Cart</p>
          <div @click="openCart()" style="cursor: pointer;" class="nav-profile-cart">
            <i class="fas fa-cart-shopping"></i>
            <div id="cartcount" class="cartcount" style="display: none;">
              {{ cart.length }}
            </div>
          </div>
        </div>
      </div>
    </nav>

    <div class="container">
      <div class="sidebar">
        <input @keyup="search" id="txt_search" type="text" class="sidebar-search" placeholder="Search..." />

        <a @click="searchproduct('all')" class="sidebar-items">
          All product
        </a>

        <a @click="searchproduct('Manga')" class="sidebar-items"> Manga </a>
        <a @click="searchproduct('Movies')" class="sidebar-items"> Movies </a>
        <a @click="searchproduct('Games')" class="sidebar-items"> Games </a>
      </div>
      <div class="product">
      <div v-for="(p, index) in productFilter" :key="p.id" @click="openProductDetail(index)"
        :class="'product-items ' + p.type">
        <img class="product-img" :src="p.img" alt="" />
        <p style="font-size: 1.3vw">{{ p.name }}</p>
        <p style="font-size: 1.vw">
          {{ this.numberWithCommas(p.price) }}.- THB
        </p>
      </div>
    </div>
    </div>


    <div id="modalDesc" class="modal" style="display: none;">
      <div @click="closeModal()" class="modal-bg"></div>
      <div class="modal-page">
        <h2>Detail</h2>
        <br>
        <div class="modaldesc-content">
          <img id="mdd-img" class="modaldesc-img" src="./imgs/chainsaw man.jpg" alt="">
          <div class="modaldesc-detail">
            <p id="mdd-name" style="font-size: 1.5vw;">chainsaw man</p>
            <p id="mdd-price" style="font-size: 1.2vw;">xxxx THB</p>
            <br>
            <p id="mdd-desc" style="color: #adadad;">Lorem ipsum dolor, sit amet consectetur adipisicing elit. Molestiae
              voluptatem dolorum perspiciatis, rerum unde ullam obcaecati ex modi nemo exercitationem quae distinctio!
              Eligendi nisi laborum provident saepe. Quibusdam, quas dolor!</p>
            <br>
            <div class="btn-control">
              <button @click="closeModal()" class="btn">
                Close
              </button>
              <button @click="addtocart()" class="btn btn-buy">
                Add to cart
              </button>
            </div>
          </div>
        </div>
      </div>
    </div>

    <div id="modalCart" class="modal" style="display: none;">
      <div @click="closeModal()" class="modal-bg"></div>
      <div class="modal-page">
        <h2>My cart</h2>
        <br>
        <div id="mycart" class="cartlist">
          <div class="cartlist-items" v-for="(c, i) in cart" :key="c.id">
            <div class="cartlist-left">
              <img :src="c.img" alt="">
              <div class="cartlist-detail">
                <p style="font-size: 1.5vw;">{{ c.name }}</p>
                <p style="font-size: 1.2vw;">{{ numberWithCommas(
                    c.price * c.count
                  )
                }}.- THB</p>
              </div>
            </div>
            <div class="cartlist-right">
              <p @click="deinitems('-', i)" class="btnc">-</p>
              <p :id="'countitems' + i" style="margin: 0 20px;">{{ c.count
              }}</p>
              <p @click="deinitems('+', i)" class="btnc">+</p>
            </div>
          </div>
          <p v-if="cart?.length < 1">Not found product list</p>
        </div>
        <div class="btn-control">
          <button @click="closeModal()" class="btn">
            Cancel
          </button>
          <button class="btn btn-buy">
            Buy
          </button>
        </div>
      </div>
    </div>


  </div>
</template>

<script>
import Navbar from "../components/Navbar.vue";
import $ from "jquery";

export default {
  components: { Navbar },
  methods: {
    numberWithCommas(x) {
      x = x.toString();
      var pattern = /(-?\d+)(\d{3})/;
      while (pattern.test(x)) x = x.replace(pattern, "$1,$2");
      return x;
    },
    search(elem) {
      if (elem.target.value)
        this.productFilter = this.product.filter(x => x.name.indexOf(elem.target.value) > -1)
      else
        this.productFilter = this.product
    },

    searchproduct(parem) {
      console.log(parem);
      $(".product-items").css("display", "none");
      if (parem == "all") {
        $(".product-items").css("display", "block");
      } else {
        $("." + parem).css("display", "block");
      }
    },
    openProductDetail(index) {
      this.productindex = index;
      console.log(this.productindex);
      $("#modalDesc").css("display", "flex");
      $("#mdd-img").attr("src", this.product[index].img);
      $("#mdd-name").text(this.product[index].name);
      $("#mdd-price").text(
        this.numberWithCommas(this.product[index].price) + ".- THB"
      );
      $("#mdd-desc").text(this.product[index].description);
    },
    closeModal() {
      $(".modal").css("display", "none");
    },
    addtocart() {
      var pass = true;

      for (let i = 0; i < this.cart.length; i++) {
        if (this.productindex == this.cart[i].index) {
          console.log("found same product");
          this.cart[i].count++;
          pass = false;
        }
      }

      if (pass) {
        var obj = {
          index: this.productindex,
          id: this.product[this.productindex].id,
          name: this.product[this.productindex].name,
          price: this.product[this.productindex].price,
          img: this.product[this.productindex].img,
          count: 1,
        };
        // console.log(obj)
        this.cart.push(obj);
      }

      this.$swal.fire({
        icon: "success",
        title: "Add" + this.product[this.productindex].name + " to cart !",
      });
      $("#cartcount").css("display", "flex");
    },
    openCart() {
      $("#modalCart").css("display", "flex");
      rendercart();
    },

    rendercart() {
      if (this.cart.length > 1) {
        var html = "";
        for (let i = 1; i < this.cart.length; i++) {
          html += `<div class="cartlist-items">
            <div class="cartlist-left">
                <img src="${this.cart[i].img}" alt="">
                <div class="cartlist-detail">
                    <p style="font-size: 1.5vw;">${this.cart[i].name}</p>
                    <p style="font-size: 1.2vw;">${numberWithCommas(
            this.cart[i].price * this.cart[i].count
          )}.- THB</p>
                </div>
            </div>
            <div class="cartlist-right">
                <p @click="deinitems('-', ${i})" class="btnc">-</p>
                <p id="countitems${i}" style="margin: 0 20px;">${this.cart[i].count
            }</p>
                <p @click="deinitems('+', ${i})" class="btnc">+</p>
            </div>
        </div>`;
        }
        $("#mycart").html(html);
      } else {
        $("#mycart").html("<p>Not found product list</p>");
      }
    },
    deinitems(action, index) {
      if (action == "-") {
        if (this.cart[index].count > 0) {
          this.cart[index].count--;
          $("#countitems" + index).text(this.cart[index].count);

          if (this.cart[index].count <= 0) {
            Swal.fire({
              icon: "warning",
              title: "Are you sure to delete?",
              showConfirmButton: true,
              showCancelButton: true,
              confirmButtonText: "Delete",
              cancelButtonText: "cancel",
            }).then((res) => {
              if (res.isConfirmed) {
                this.cart.splice(index, 1);
                rendercart();

                if (this.cart.length <1) {
                  $("#cartcount").css("display", "none");
                } else {
                  $("#cartcount").css("display", "flex");
                }
              } else {
                this.cart[index].count++;
                $("#countitems" + index).text(this.cart[index].count);
              }
            });
          }
        }
      } else if (action == "+") {
        this.cart[index].count++;
        $("#countitems" + index).text(this.cart[index].count);
      }
    },
  },
  data() {
    return {
      cart: [],
      productindex: 0, productFilter: [],
      product: [
        {
          id: 1,
          img: "./imgs/chainsaw man.jpg",
          name: "Chainsaw man",
          price: 69,
          description:
            "Chainsaw man Lorem ipsum dolor sit amet consectetur adipisicing elit. Repellat mollitia ad harum provident, ut magnam.",
          type: "Manga",
        },
        {
          id: 2,
          img: "./imgs/john wick.jpg",
          name: "John wick",
          price: 190,
          description:
            "John wick Lorem ipsum dolor sit amet consectetur adipisicing elit. Corporis quam doloribus nisi laboriosam nobis dolorem!",
          type: "Movies",
        },
        {
          id: 3,
          img: "./imgs/the last of us.jpg",
          name: "The last of us",
          price: 1590,
          description:
            "The last of us Lorem ipsum dolor sit amet consectetur adipisicing elit. Corporis quam doloribus nisi laboriosam nobis dolorem!",
          type: "Games",
        },
        {
          id: 4,
          img: "./imgs/blue lock.jpg",
          name: "Blue lock",
          price: 90,
          description:
            "Blue lock Lorem ipsum dolor sit amet consectetur adipisicing elit. Corporis quam doloribus nisi laboriosam nobis dolorem!",
          type: "Manga",
        },
        {
          id: 5,
          img: "./imgs/spy x family.jpg",
          name: "Spy x Family",
          price: 89,
          description:
            "Spy x Family Lorem ipsum dolor sit amet consectetur adipisicing elit. Corporis quam doloribus nisi laboriosam nobis dolorem!",
          type: "Manga",
        },
        {
          id: 6,
          img: "./imgs/top-gun.jpg",
          name: "TOP-GUM Maverick",
          price: 289,
          description:
            "TOP-GUM Maverick Lorem ipsum dolor sit amet consectetur adipisicing elit. Corporis quam doloribus nisi laboriosam nobis dolorem!",
          type: "Movies",
        },
        {
          id: 7,
          img: "./imgs/cyberpunk.jpg",
          name: "Cyberpunk Edgerunners",
          price: 199,
          description:
            "Cyberpunk Edgerunners Lorem ipsum dolor sit amet consectetur adipisicing elit. Corporis quam doloribus nisi laboriosam nobis dolorem!",
          type: "Movies",
        },
        {
          id: 8,
          img: "./imgs/god of war.jpg",
          name: "God of War (Ragnarok)",
          price: 6490,
          description:
            "God of War (Ragnarok) Lorem ipsum dolor sit amet consectetur adipisicing elit. Corporis quam doloribus nisi laboriosam nobis dolorem!",
          type: "Games",
        },
        {
          id: 9,
          img: "./imgs/avartar.jpg",
          name: "Avatar",
          price: 260,
          description:
            "Avatar Lorem ipsum dolor sit amet consectetur adipisicing elit. Corporis quam doloribus nisi laboriosam nobis dolorem!",
          type: "Movies",
        },
        {
          id: 10,
          img: "./imgs/jujutsu.jpg",
          name: "Jujutsu Kaisen",
          price: 89,
          description:
            "Jujutsu Kaisen Lorem ipsum dolor sit amet consectetur adipisicing elit. Corporis quam doloribus nisi laboriosam nobis dolorem!",
          type: "Manga",
        },
        {
          id: 11,
          img: "./imgs/titanfall.jpg",
          name: "TitanFall 2",
          price: 390,
          description:
            "TitanFall 2 Lorem ipsum dolor sit amet consectetur adipisicing elit. Corporis quam doloribus nisi laboriosam nobis dolorem!",
          type: "Games",
        },
        {
          id: 12,
          img: "./imgs/stray.jpg",
          name: "Stray",
          price: 1059,
          description:
            "Stray Lorem ipsum dolor sit amet consectetur adipisicing elit. Corporis quam doloribus nisi laboriosam nobis dolorem!",
          type: "Games",
        },
        {
          id: 13,
          img: "./imgs/slam-dunk.jpg",
          name: "Slam Dunk",
          price: 290,
          description:
            "Slam Dunk Lorem ipsum dolor sit amet consectetur adipisicing elit. Corporis quam doloribus nisi laboriosam nobis dolorem!",
          type: "Manga",
        },
        {
          id: 14,
          img: "./imgs/battlefield.jpg",
          name: "Battlefield 1",
          price: 1290,
          description:
            "Battlefield 1 Lorem ipsum dolor sit amet consectetur adipisicing elit. Corporis quam doloribus nisi laboriosam nobis dolorem!",
          type: "Games",
        },
        {
          id: 15,
          img: "./imgs/pacific-rim.jpg",
          name: "Pacific-rim",
          price: 320,
          description:
            "Pacific-rim Lorem ipsum dolor sit amet consectetur adipisicing elit. Corporis quam doloribus nisi laboriosam nobis dolorem!",
          type: "Movies",
        },
      ],
    };
  },
  mounted() {
    this.productFilter = this.product
  },
};
</script>

<style>

</style>