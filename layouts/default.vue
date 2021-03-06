<template>
  <v-app>
    <v-navigation-drawer v-if="!showDrawer" v-model="drawer" :clipped="clipped" fixed app>
      <v-list>
        <v-list-item v-for="(item, i) in items" :key="i" :to="item.to" router exact>
          <v-list-item-action>
            <v-icon>{{ item.icon }}</v-icon>
          </v-list-item-action>

          <v-list-item-content>
            <v-list-item-title v-text="item.title" />
          </v-list-item-content>
        </v-list-item>

        <v-list-item v-if="user" to="/cart" router exact>
          <v-list-item-action>
            <v-badge v-if="cart.length" color="green" :content="cart.length" overlap bordered>
              <v-icon>mdi-cart-outline</v-icon>
            </v-badge>
            <v-icon v-else>mdi-cart-outline</v-icon>
          </v-list-item-action>

          <v-list-item-content>
            <v-list-item-title>Корзина</v-list-item-title>
          </v-list-item-content>
        </v-list-item>

        <v-list-item>
          <v-row align="center" justify="center">
            <ButtonFeedback />
          </v-row>
        </v-list-item>
      </v-list>
    </v-navigation-drawer>

    <v-app-bar :clipped-left="clipped" dense fixed app>
      <v-app-bar-nav-icon v-if="!showDrawer" @click.stop="drawer = !drawer" />
      <v-toolbar-title class="mr-1" v-text="title" />

      <template v-if="showDrawer">
        <v-btn v-for="(item, i) in items" :key="i" :to="item.to" class="mx-1" router text>
          <v-icon>{{ item.icon }}</v-icon>
          <h4>{{ item.title }}</h4>
        </v-btn>

        <ButtonFeedback />

        <v-btn v-if="user" to="/cart" class="mx-1" router text>
          <v-badge v-if="cart.length" class="mr-2" color="deep-purple lighten-2" :content="cart.length" overlap>
            <v-icon>mdi-cart-outline</v-icon>
          </v-badge>
          <v-icon v-else>mdi-cart-outline</v-icon>

          <h4>Корзина</h4>
        </v-btn>
      </template>

      <v-spacer />

      <div v-if="!user">
        <ButtonSignin />
        <ButtonSignup />
      </div>
      <div v-else class="text-center">
        <v-menu offset-y transition="slide-y-transition">
          <template #activator="{ on, attrs }">
            <v-btn color="secondary" text v-bind="attrs" v-on="on">
              <v-icon>mdi-account</v-icon>
              {{ user.username }}
            </v-btn>
          </template>
          <v-list>
            <v-list-item link router to="/profile">
              <v-list-item-title class="d-flex justify-start align-center">
                <v-icon>mdi-account</v-icon>
                Профиль
              </v-list-item-title>
            </v-list-item>
            <v-list-item link router to="/orders">
              <v-list-item-title class="d-flex justify-start align-center">
                <v-icon>mdi-clipboard</v-icon>
                Заказы
              </v-list-item-title>
            </v-list-item>
            <v-list-item v-if="user.isAdmin" link router to="/admin">
              <v-list-item-title class="d-flex justify-start align-center">
                <v-icon>mdi-shield-account</v-icon>
                Админ-панель
              </v-list-item-title>
            </v-list-item>

            <v-divider class="my-2" />

            <v-list-item link @click="signOut">
              <v-list-item-title class="d-flex justify-start align-center">
                <v-icon>mdi-logout</v-icon>
                Выйти
              </v-list-item-title>
            </v-list-item>
          </v-list>
        </v-menu>
      </div>
    </v-app-bar>

    <v-main>
      <Nuxt />

      <v-snackbar v-model="snackbar.enabled" :timeout="snackbar.timeout" :color="snackbar.color" rounded="lg">
        <v-row align="center">
          <v-icon class="mx-1">{{ snackbar.icon }}</v-icon>
          <h3>{{ snackbar.text }}</h3>
        </v-row>

        <template #action="{ attrs }">
          <v-btn color="white" icon v-bind="attrs" @click="snackbar.enabled = false">
            <v-icon>mdi-close</v-icon>
          </v-btn>
        </template>
      </v-snackbar>
    </v-main>

    <v-footer dark :absolute="!fixed" padless>
      <v-card flat tile class="indigo lighten-1 white--text text-center" width="100%">
        <v-card-text>
          <v-btn v-for="item in footerIcons" :key="item.icon" class="mx-4 white--text" router :href="item.url" target="_blank" icon>
            <v-icon size="24px">{{ item.icon }}</v-icon>
          </v-btn>
        </v-card-text>

        <v-card-text class="white--text pt-0">
          Lorem ipsum dolor sit amet consectetur, adipisicing elit. Eaque blanditiis neque ea non, quas perspiciatis vero deserunt fugit atque provident, officia mollitia libero doloremque ex cupiditate, iste sequi ad maiores.
        </v-card-text>

        <v-divider />

        <v-card-text class="white--text">
          {{ new Date().getFullYear() }} — <strong>Fades</strong>
        </v-card-text>
      </v-card>
    </v-footer>
  </v-app>
</template>

<script>
import { mapState } from "vuex";

export default {
  data: () => ({
    title: "Nuxt Store",
    clipped: true,
    drawer: false,
    fixed: true,
    snackbar: {
      enabled: false,
      timeout: 3000,
      color: "green",
      icon: "mdi-information",
      text: ""
    },
    items: [
      {
        icon: "mdi-home-outline",
        title: "Главная",
        to: "/"
      },
      {
        icon: "mdi-archive-outline",
        title: "Каталог",
        to: "/catalog"
      }
    ],
    footerIcons: [
      { icon: "mdi-discord", url: "https://discord.gg/ETrKUWmCN4" },
      { icon: "mdi-twitter", url: "https://twitter.com" },
      { icon: "mdi-vk", url: "https://vk.com" },
      { icon: "mdi-instagram", url: "https://www.instagram.com" }
    ]
  }),
  computed: {
    ...mapState({
      user: state => state.user,
      cart: state => state.userCart
    }),
    showDrawer() {
      const disabledSizes = ["xs", "sm", "md"];
      if (disabledSizes.includes(this.$vuetify.breakpoint.name)) return false;
      return true;
    }
  },
  created () {
    this.$nuxt.$on("snackbarCall", (text, color = "green", icon = "mdi-check") => {
      this.snackbar.enabled = true;
      this.snackbar.text = text;
      this.snackbar.color = color;
      this.snackbar.icon = icon;
    });
  },
  async mounted() {
    const { data: cartItems } = await this.$axios.get("/cart");
    this.$store.commit("setCart", cartItems);
  },
  methods: {
    async signOut() {
      await this.$axios.$get("/auth/signout");
      window.location.reload();
    }
  }
};
</script>
