<template>
  <v-dialog v-model="dialog" max-width="600px" :disabled="inShoppingCart">
    <template #activator="{ on, attrs }">
      <v-btn v-if="inShoppingCart" color="deep-purple lighten-2" to="/cart" nuxt text>
        В корзине
      </v-btn>
      <v-btn v-else color="deep-purple lighten-2" text v-bind="attrs" v-on="on">
        Добавить в корзину
      </v-btn>
    </template>
    <v-form ref="form" v-model="valid" lazy-validation>
      <v-card>
        <v-card-title>
          <span class="headline">Добавить в корзину</span>
        </v-card-title>
        <v-card-text>
          <v-container>
            <v-row>
              <v-col cols="12">
                <v-text-field
                  v-model="quantity"
                  :rules="[rules.required, ...rules.number, v => (v && v.length <= 9) || 'Длина не должна превышать 9 цифр.']"
                  counter="9"
                  label="Количество"
                  type="number"
                  required
                />
              </v-col>
            </v-row>
          </v-container>
          <h3 v-if="error" class="red--text">{{ error }}</h3>
        </v-card-text>
        <v-card-actions>
          <v-spacer />
          <v-btn color="blue darken-1" text @click.prevent="addToShoppingCart">
            Добавить
          </v-btn>
          <v-btn color="blue darken-1" text @click="dialog = false">
            Закрыть
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-form>
  </v-dialog>
</template>

<script>
import { mapState } from "vuex";

export default {
  props: {
    productId: {
      type: Number,
      required: true
    }
  },
  data: () => ({
    valid: true,
    dialog: false,
    quantity: "1",
    error: "",
    rules: {
      required: v => !!v || "Это поле обязательно для заполнения",
      number: [
        v => !(/-\D/.test(v)) || "Только цифры",
        v => v > 0 || "Число должно быть больше 0"
      ]
    }
  }),
  computed: {
    ...mapState({
      cart: state => state.userCart
    }),
    inShoppingCart() {
      if (!this.cart.length) return false;

      const product = this.cart.filter(product => product.id === this.productId)[0];
      if (product) return true;

      return false;
    }
  },
  methods: {
    async addToShoppingCart() {
      const isValid = this.$refs.form.validate();
      if (!isValid) return;

      await this.$axios.post("/cart", {
        productId: this.productId,
        quantity: this.quantity
      }).then(res => {
        if (!res.data.success) {
          this.error = res.data.error || "Что-то пошло не так";
          this.$nuxt.$emit("snackbarCall", "Не удалось добавить товар в корзину.", "red", "mdi-alert");

          return;
        }

        this.$refs.form.reset();
        this.$nuxt.$emit("snackbarCall", "Товар добавлен в корзину.");

        this.disabled = true;
        this.dialog = false;
      }).catch(err => {
        console.log(err);
        this.error = "Что-то пошло не так";
      });

      const { data: cartItems } = await this.$axios.get("/cart");
      this.$store.commit("setCart", cartItems);
    }
  }
};
</script>
