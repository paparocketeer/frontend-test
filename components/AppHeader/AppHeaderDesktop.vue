<template>
  <div class="header">
    <div class="header__container" ref="headerContainer">
      <div class="header__row">
        <AppHeaderLogo class="header__logo" />
        <ul class="header__list header__list--menu">
          <li class="header__item">
            <ExternalLink
              class="header__link"
              href="https://gdematerial.ru/blog/"
            >
              Блог
            </ExternalLink>
          </li>
          <li class="header__item">
            <nuxt-link class="header__link" to="/info/contacts/">
              Контакты
            </nuxt-link>
          </li>
          <li class="header__item">
            <nuxt-link class="header__link" to="/info/payments/">
              Оплата
            </nuxt-link>
          </li>
          <li class="header__item">
            <nuxt-link class="header__link" to="/info/delivery/">
              Доставка
            </nuxt-link>
          </li>
          <li class="header__item">
            <ExternalLink class="header__link" href="https://partner.gdml.ru">
              Поставщикам
            </ExternalLink>
          </li>
        </ul>
        <ul class="header__list header__list--user-menu">
          <li class="header__item">
            <button class="header__link">
              <Icon
                class="header__icon header__icon--box"
                name="gm-header-box"
              />
              Отследить заказ
            </button>
          </li>
          <li class="header__item">
            <button class="header__link">
              Вход и регистрация
            </button>
          </li>
        </ul>
      </div>
      <div class="header__row">
        <AppHeaderCatalogBtn
          class="header__btn header__btn--catalog"
          :class="{ 'header-catalog-btn--open': catalogShow }"
          @click.native="toggleCatalog"
        />
        <AppSearch class="header__search" />
        <nuxt-link class="header__btn header__btn--cart" to="/cart/1/">
          <Icon class="header__icon header__icon--cart" name="gm-cart" />
          <span class="header__text header__text--cart">Мой заказ</span>
          <client-only>
            <span
              class="header__counter"
              v-if="itemsCount"
              v-text="itemsCount"
            />
          </client-only>
        </nuxt-link>
      </div>
      <div class="header__wrapper header__wrapper--menu" />
    </div>
    <div class="header__container">
      <div
        class="header__catalog"
        v-show="catalogShow"
        :class="{ 'header-catalog--open': catalogShow }"
      >
        <ul
          v-on-outside="closeCatalogOnOutside"
          class="header__catalog--list"
          ref="catalogList"
        >
          <li v-if="selectedGroup" class="header-catalog-item header-catalog-item--prev">
            <button class="header-catalog-item__link header-catalog-item__link--prev" @click="rootUp(selectedGroup)">
              <svg
                version="1.1"
                role="presentation"
                width="8"
                height="16"
                viewBox="0 0 6 12"
                class="header-catalog-item__icon header-catalog-item__icon--prev"
              >
                <path
                  d="M0.75 10.5L5.25 6L0.75 1.5"
                  stroke="currentColor"
                  stroke-width="1.5"
                  stroke-linecap="round"
                  stroke-linejoin="round"
                  fill="none"
                ></path>
                <!---->
              </svg>
              <span class="header-catalog-item__text">{{ selectedGroup.name }}</span>
            </button>
          </li>
          <li
            v-for="(item, index) in selectedGroup ? selectedGroup.submenu : products"
            :key="index"
            class="header-catalog-item"
            :class="{ 'header-catalog-item--first': index==0 }"
          >
            <a
              class="header-catalog-item__link"
              @click.prevent="openCategory(item)"
            >
              <span class="header-catalog-item__text">{{ item.name }}</span>
              <svg
                version="1.1"
                role="presentation"
                width="8"
                height="16"
                viewBox="0 0 6 12"
                class="header-catalog-item__icon"
              >
                <path
                  d="M0.75 10.5L5.25 6L0.75 1.5"
                  stroke="currentColor"
                  stroke-width="1.5"
                  stroke-linecap="round"
                  stroke-linejoin="round"
                  fill="none"
                ></path>
              </svg>
            </a>
          </li>
          <li class="header-catalog-item header-catalog-item--all">
            <nuxt-link :to="selectedGroup ? selectedGroup.absolute_url : '/catalog/'" class="header-catalog-item__link header-catalog-item__link--all">
              {{ selectedGroup ? "Все товары в категории" : "Все товары" }}
            </nuxt-link>
          </li>
        </ul>
      </div>
    </div>
  </div>
</template>

<script>
import AppHeaderLogo from '~/components/AppHeader/AppHeaderLogo.vue';
import ExternalLink from '~/components/ExternalLink.vue';
import AppSearch from '~/components/AppSearch/AppSearch.vue';
import AppHeaderCatalogBtn from '~/components/AppHeader/buttons/AppHeaderCatalogBtn.vue';

export default {
  components: {
    AppHeaderLogo,
    ExternalLink,
    AppSearch,
    AppHeaderCatalogBtn,
  },
  data() {
    return {
      items: 0,
      catalogShow: false,
      headerHeight: '0px',
      products: null,
      selectedGroup: null,
      prevGroup: null,
    };
  },
  watchQuery: ['page'],
  computed: {
    itemsCount() {
      if (process.server) return 0;

      return this.items.length > 99 ? 99 : this.items.length;
    },
  },
  mounted() {
    const h = this.$refs.headerContainer.clientHeight;
    this.$refs.catalogList.style.height = `${window.innerHeight - h}px`;
    this.getProducts();
  },
  methods: {
    toggleCatalog(e) {
      e.preventDefault();
      this.catalogShow = !this.catalogShow;
      document.body.classList.remove('body--fixed');
      if (this.catalogShow) {
        document.body.classList.add('body--fixed');
      }
    },
    closeCatalogOnOutside(e) {
      if (e && e.target.className !== 'header-catalog-btn__label') {
        this.catalogShow = false;
      }
    },
    openCategory(item) {
      if (item.submenu) {
        this.prevGroup = this.selectedGroup;
        this.selectedGroup = item;
      } else {
        this.$router.push(item.absolute_url);
      }
    },
    rootUp(current) {
      const found = this.products.some((product) => product === current);
      this.selectedGroup = !found ? this.prevGroup : null;
    },
    async getProducts() {
      const products = await this.$axios.get('/categories_tree');
      this.products = products.data;
    },
  },
};
</script>

<style lang="scss" scoped>
.header {
  &__container {
    max-width: $container-xl;
    padding: 16px;
    margin: 0 auto;

    @media (min-width: $screen-xl) {
      padding: 16px 84px;
    }
  }

  &__catalog {
    margin-top: -$gap;
    background-color: $white;

    &--list {
      padding: 0;
      overflow-y: scroll;
      list-style: none;
    }
  }

  .header-catalog-item {
    --background-color: #{$white};
    --link-color: #{$black};
    --icon-color: #{$black};

    &--prev {
      border-bottom: 1px solid $gray-light;
    }

    &--first {
      margin-top: 10px;
    }

    &--all {
      --all-link-color: #{$primary-color};
      --all-background-color: #{$white};
    }

    &:hover {
      --background-color: #{$primary-color};
      --link-color: #{$white};
      --icon-color: #{$white};
    }

    &__link {
      display: flex;
      justify-content: space-between;
      align-items: center;
      width: 100%;
      padding: 6px 8px;
      font-family: Montserrat, Arial, sans-serif;
      font-size: 0.8125rem;
      font-weight: 500;
      line-height: 1.25rem;
      color: var(--link-color);
      cursor: pointer;
      background-color: var(--background-color);
      border: 0;
      border-radius: 0;
      outline: none;
      -webkit-tap-highlight-color: transparent;

      &--prev {
        justify-content: flex-start;
        padding: 15px 20px;
      }

      &--all {
        color: var(--all-link-color);
        background-color: var(--all-background-color);
      }
    }

    &__icon {
      width: 6px;
      height: 20px;
      margin-left: 8px;
      color: var(--icon-color);

      &--prev {
        width: 10px;
        margin-right: 15px;
        transform: rotate(180deg);
      }
    }
  }

  &__row {
    display: flex;
    align-items: center;
    padding-top: 16px;
  }

  &__logo {
    flex-shrink: 0;
    margin-right: 20px;
  }

  &__list {
    display: flex;
    padding: 0;
    list-style: none;

    &--menu {
      flex-grow: 1;
      justify-content: center;
      margin-right: 20px;
    }

    &--user-menu {
      flex-shrink: 0;
    }
  }

  &__item {
    margin-right: 20px;

    &:last-child {
      margin-right: 0;
    }
  }

  &__link {
    display: flex;
    align-items: center;
    padding: 0;
    font-family: $font-family;
    font-size: 14px;
    line-height: 18px;
    color: hsl(198, 23%, 19%);
    cursor: pointer;
    background-color: transparent;
    border: none;
    outline: none;
    -webkit-tap-highlight-color: transparent;

    &:hover {
      color: $primary-color;
    }
  }

  &__search {
    flex-grow: 1;
  }

  &__btn {
    &--catalog {
      margin-right: 12px;
    }

    &--cart {
      display: flex;
      align-items: center;
      padding: 10px 8px 10px 14px;
      margin-left: 12px;
      font-family: $font-family;
      font-weight: 500;
      color: hsl(198, 23%, 19%);
      cursor: pointer;
      background-color: hsl(215, 38%, 94%);
      border: none;
      border-radius: 3px;
      outline: none;
      -webkit-tap-highlight-color: transparent;
    }
  }

  &__icon {
    flex-shrink: 0;

    &--box {
      width: 14px;
      height: 16px;
      margin-right: 4px;
    }

    &--cart {
      width: 20px;
      height: 20px;
    }
  }

  &__text {
    &--cart {
      margin: 0 8px;
    }
  }

  &__counter {
    display: flex;
    align-items: center;
    justify-content: center;
    width: 22px;
    height: 22px;
    font-size: 12px;
    font-weight: 600;
    line-height: 1;
    color: hsl(0, 0%, 100%);
    background-color: hsl(229, 78%, 50%);
    border-radius: 50%;
  }
}
</style>
