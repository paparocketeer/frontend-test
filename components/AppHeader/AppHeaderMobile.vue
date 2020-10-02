<template>
  <div class="header">
    <div class="header__container" ref="headerContainer">
      <div class="header__row">
        <button
          class="header__btn header__btn--menu"
          :class="{ 'header__btn--menu-open': menuShow }"
          aria-label="Меню"
          @click="toggleMenu"
        >
          <Icon
            :class="!menuShow ? 'header__icon--burger' : 'header__icon--close'"
            class="header__icon"
            :name="!menuShow ? 'nav-burger' : 'nav-burger-close'"
          />
        </button>
        <AppHeaderLogo class="header__logo" />
        <AppHeaderUserMenu class="header__user-menu" />
      </div>
      <div class="header__row">
        <AppHeaderCatalogBtn
          class="header__btn header__btn--catalog"
          @click.native="toggleCatalog"
          :class="{ 'header-catalog-btn--open': catalogShow }"
        />
        <AppSearch class="header__search" />
      </div>
    </div>
    <div class="header__wrapper header__wrapper--menu">
      <ul
        v-on-outside="closeMenuOnOutside"
        class="header-menu"
        v-show="menuShow"
        :class="{ 'header-menu--open': menuShow }"
        ref="menuList"
      >
        <li class="header-menu__item">
          <button class="header-menu__link">
            Прайс-лист
          </button>
        </li>
        <li class="header-menu__item">
          <button class="header-menu__link">
            Отслеживание заказа
          </button>
        </li>
        <li class="header-menu__item">
          <button class="header-menu__link">
            Каталог товаров
          </button>
        </li>
        <li class="header-menu__item">
          <nuxt-link class="header-menu__link" to="/info/delivery/">
            Доставка
            <Icon
              class="header-menu__icon header-menu__icon--angle-right"
              name="gm-angle-right"
            />
          </nuxt-link>
        </li>
        <li class="header-menu__item">
          <nuxt-link class="header-menu__link" to="/about/">
            О компании
            <Icon
              class="header-menu__icon header-menu__icon--angle-right"
              name="gm-angle-right"
            />
          </nuxt-link>
        </li>
        <li class="header-menu__item">
          <nuxt-link class="header-menu__link" to="/info/contacts/">
            Контакты
            <Icon
              class="header-menu__icon header-menu__icon--angle-right"
              name="gm-angle-right"
            />
          </nuxt-link>
        </li>
        <li class="header-menu__item">
          <ExternalLink
            class="header-menu__link"
            href="https://gdematerial.ru/blog/"
            target="_blank"
          >
            Блог
          </ExternalLink>
        </li>
        <li class="header-menu__item">
          <ExternalLink
            class="header-menu__link"
            href="https://partner.gdml.ru"
            target="_blank"
          >
            Поставщикам
          </ExternalLink>
        </li>
      </ul>
      <ul
        v-on-outside="closeCatalogOnOutside"
        v-show="catalogShow"
        class="header-menu header-menu--catalog"
        ref="catalogList"
      >
        <li
          v-if="selectedGroup"
          class="header-menu__item header-menu__item--prev"
        >
          <a
            class="header-menu__link header-menu__link--prev"
            @click.prevent="rootUp(selectedGroup)"
          >
            <Icon
              class="header-menu__icon header-menu__icon--prev header-menu__icon--angle-right"
              name="gm-angle-right"
            />
            {{ selectedGroup.name }}
          </a>
        </li>
        <li
          class="header-menu__item"
          v-for="(item, index) in selectedGroup
            ? selectedGroup.submenu
            : products"
          :key="index"
        >
          <a class="header-menu__link" @click.prevent="openCategory(item)">
            {{ item.name }}
            <Icon
              class="header-menu__icon header-menu__icon--angle-right"
              name="gm-angle-right"
            />
          </a>
        </li>
        <li class="header-menu__item header-menu__item--all">
          <nuxt-link
            class="header-menu__link header-menu__link--all"
            :to="selectedGroup ? selectedGroup.absolute_url : '/catalog/'"
          >
            {{ selectedGroup ? "Все товары в категории" : "Все товары" }}
          </nuxt-link>
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
import AppHeaderLogo from '~/components/AppHeader/AppHeaderLogo.vue';
import AppHeaderUserMenu from '~/components/AppHeader/mobile/AppHeaderUserMenu.vue';
import AppHeaderCatalogBtn from '~/components/AppHeader/buttons/AppHeaderCatalogBtn.vue';
import AppSearch from '~/components/AppSearch/AppSearch.vue';
import ExternalLink from '~/components/ExternalLink.vue';

export default {
  components: {
    AppHeaderLogo,
    AppHeaderUserMenu,
    AppHeaderCatalogBtn,
    AppSearch,
    ExternalLink,
  },
  data() {
    return {
      catalogShow: false,
      menuShow: false,
      headerHeight: '0px',
      products: null,
      selectedGroup: null,
      prevGroup: null,
    };
  },
  watchQuery: ['page'],
  mounted() {
    const h = this.$refs.headerContainer.clientHeight;
    this.$refs.catalogList.style.height = `${window.innerHeight - h}px`;
    this.$refs.menuList.style.height = `${window.innerHeight - h}px`;
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
    closeMenuOnOutside(e) {
      if (e && !e.target.ownerDocument.activeElement.className.includes('open')) {
        this.menuShow = false;
      }
    },
    toggleMenu() {
      this.menuShow = !this.menuShow;
      document.body.classList.remove('body--fixed');
      if (this.menuShow) {
        document.body.classList.add('body--fixed');
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
    padding: 10px 8px 10px 8px;
    margin: 0 auto;
  }

  &__row {
    display: flex;
    align-items: center;
    margin-bottom: 8px;

    &:last-child {
      margin-bottom: 0;
    }
  }

  &__user-menu {
    margin-left: auto;
  }

  &__search {
    flex-grow: 1;
  }

  &__btn {
    flex-shrink: 0;

    &--menu {
      display: flex;
      justify-content: center;
      align-items: center;
      padding: 0;
      margin-right: 18px;
      color: hsl(0, 0%, 0%);
      cursor: pointer;
      background-color: transparent;
      border: none;
      outline: none;
      -webkit-tap-highlight-color: transparent;
    }

    &--catalog {
      margin-right: 5px;
    }
  }

  &__icon {
    &--burger {
      width: 20px;
      height: 14px;
    }

    &--close {
      width: 20px;
      height: 14px;
    }
  }
}

.header-menu {
  --link-background-color: hsl(0, 0%, 100%);
  --link-border-radius: 0;
  --link-padding: 6px 8px;
  --link-color: hsl(198, 23%, 19%);
  --link-outline-color: transparent;
  --icon--angle-right-color: hsl(199, 22%, 77%);

  padding: 0;
  overflow-y: scroll;
  list-style: none;
  background-color: hsl(0, 0%, 100%);

  &__item {
    @media (min-width: $screen-lg) {
      --link-padding: 7px 20px;
    }

    &:hover,
    &--active {
      --link-background-color: hsl(230, 78%, 51%);
      --link-color: hsl(0, 0%, 100%);
      --link-outline-color: hsl(230, 78%, 51%);
      --icon--angle-right-color: hsl(0, 0%, 100%);
    }

    &--all {
      --link-color: hsl(230, 78%, 51%);
    }

    &:first-child {
      margin-top: 10px;

      @media (min-width: $screen-lg) {
        --link-padding: 16px 20px 10px 20px;
      }
    }

    &:last-child {
      @media (min-width: $screen-lg) {
        --link-padding: 10px 20px 16px 20px;
      }
    }
  }

  &__link {
    display: flex;
    justify-content: space-between;
    align-items: center;
    width: 100%;
    padding: var(--link-padding);
    font-family: $font-family-montserrat;
    font-size: 13px;
    font-weight: 500;
    line-height: 20px;
    color: var(--link-color);
    cursor: pointer;
    background-color: var(--link-background-color);
    border: 1px solid transparent;
    border-radius: var(--link-border-radius);
    outline: none;
    -webkit-tap-highlight-color: transparent;

    @media (min-width: $screen-lg) {
      outline: 3px solid var(--link-outline-color);
    }

    &--prev {
      justify-content: flex-start;
      padding: 15px 20px;
    }
  }

  &__icon {
    flex-shrink: 0;

    &--angle-right {
      width: 6px;
      height: 20px;
      margin-right: 10px;
      color: var(--icon--angle-right-color);

      @media (min-width: $screen-lg) {
        margin-left: 10px;
      }
    }

    &--prev {
      width: 10px;
      margin-right: 15px;
      transform: rotate(180deg);
    }
  }
}
</style>
