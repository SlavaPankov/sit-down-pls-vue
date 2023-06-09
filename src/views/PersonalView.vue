<template>
  <section class="personal" v-if="pageLoaded">
    <div class="container personal__container">
      <h1 class="heading-reset personal__heading">
        Личный кабинет
      </h1>
      <div class="personal__head">
        <h2 class="heading-reset personal__subheading">
          Личные данные
        </h2>
        <button class="btn-reset personal__edit-button"
                @click.prevent="editHandle"
        >
          {{ editData ? 'Сохранить' : 'Редактировать данные' }}
        </button>
      </div>
      <form class="personal__form">
          <base-form-text-input title="Фамилия:"
                                placeholder="Фамилия"
                                type="text"
                                name="surName"
                                :require="true"
                                :error="formError.surName"
                                :disabled="!editData"
                                v-model:value="formData.surName"/>
          <base-form-text-input title="Имя:"
                                placeholder="Имя"
                                type="text"
                                name="name"
                                :require="true"
                                :error="formError.name"
                                :disabled="!editData"
                                v-model:value="formData.name"/>
          <base-form-text-input title="Отчество:"
                                placeholder="Отчество"
                                type="text"
                                name="middleName"
                                :error="formError.middleName"
                                :disabled="!editData"
                                v-model:value="formData.middleName"/>
          <base-form-text-input class="order-form__label--phone" title="Телефон:"
                                placeholder="+7 (999)-999-99-99"
                                type="tel"
                                name="phone"
                                :require="true"
                                :error="formError.phone"
                                :disabled="!editData"
                                v-model:value="formData.phone"/>
          <base-form-text-input class="order-form__label--email" title="Email:"
                                placeholder="example@email.com"
                                type="email"
                                name="email"
                                :require="true"
                                :error="formError.email"
                                :disabled="!editData"
                                v-model:value="formData.email"/>
        </form>
      <h2 class="heading-reset personal__subheading personal__subheading--history">
        История заказов
      </h2>
      <orders-list :orders="paginatedOrders" />
      <base-pagination v-if="user.orders.length > countPerPage"
                       :total-count="user.orders.length"
                       :count-per-page="countPerPage"
                       v-model:page="page"
      />
    </div>
  </section>
  <base-modal :open="updatingUser">
    <base-spinner class="spinner" />
  </base-modal>
  <base-spinner v-if="pageLoading" />
</template>

<script>
// eslint-disable-next-line import/no-extraneous-dependencies
import axios from 'axios';
import { mapGetters, mapActions, mapMutations } from 'vuex';
import { BASE_URL } from '@/api/config';
import BaseFormTextInput from '@/components/BaseFormTextInput';
import BaseSpinner from '@/components/BaseSpinner';
import BaseModal from '@/components/BaseModal';
import OrdersList from '@/components/OrdersList';
import BasePagination from '@/components/BasePagination';

export default {
  name: 'PersonalView',
  components: {
    BaseFormTextInput,
    BaseSpinner,
    BaseModal,
    OrdersList,
    BasePagination,
  },

  data() {
    return {
      user: {},
      formData: {},
      formError: {},
      editData: false,
      updatingUser: false,
      updatedUser: true,
      pageLoading: true,
      pageLoaded: false,
      page: 1,
      countPerPage: 8,
    };
  },

  computed: {
    ...mapGetters([
      'getRememberToken',
    ]),

    paginatedOrders() {
      return this.user.orders.slice(
        this.countPerPage * (this.page - 1),
        this.countPerPage * this.page,
      );
    },
  },

  methods: {
    ...mapActions(['loadUserInfo']),
    ...mapMutations({
      updateUserStore: 'updateUser',
    }),

    editHandle() {
      if (this.editData) {
        this.formError = {};

        this.formError.surName = this.checkLength(this.formData.surName.length)
          || this.isValidTextInput(this.formData.surName);
        this.formError.name = this.checkLength(this.formData.name.length)
          || this.isValidTextInput(this.formData.name);
        this.formError.phone = this.checkPhoneLength(this.formData.phone.length);

        Object.keys(this.formError).forEach((key) => {
          if (this.formError[key] === undefined) {
            delete this.formError[key];
          }
        });

        if (Object.keys(this.formError).length === 0) {
          this.updateUser();
        }
      }

      this.editData = true;
    },

    updateUser() {
      this.updatingUser = true;
      this.updatedUser = false;

      return axios.put(`${BASE_URL}/api/users/${this.getRememberToken}`, {
        ...this.formData,
      })
        .then((response) => {
          this.user = response.data.payload;
          document.querySelector('.modal').classList.add('modal-close');
          this.updatedUser = true;
          this.editData = false;
          this.updateUserStore(response.data.payload);

          setTimeout(() => {
            this.updatingUser = false;
          }, 300);
        });
    },

    // eslint-disable-next-line consistent-return
    checkLength(value) {
      if (value < 3) {
        return 'Минимальная длина 3 символа';
      }

      if (value > 30) {
        return 'Максимальная длина 30 символов';
      }
    },

    // eslint-disable-next-line consistent-return
    checkPhoneLength(value) {
      if (value < 18) {
        return 'Некорректный формат';
      }
    },

    // eslint-disable-next-line consistent-return
    isValidTextInput(value) {
      if (!/^[а-я-]+$/i.test(value)) {
        return 'Неверный формат';
      }
    },
  },

  created() {
    if (this.getRememberToken === '') {
      this.$router.push({ name: 'auth' });
    }

    this.loadUserInfo().then((response) => {
      if (response.data.error === null) {
        this.pageLoaded = true;
        this.pageLoading = false;
        this.user = response.data.payload;
        this.formData.surName = this.user.sur_name;
        this.formData.name = this.user.name;
        this.formData.middleName = this.user.middle_name;
        this.formData.phone = this.user.phone;
        this.formData.email = this.user.email;
      } else {
        this.$router.push({ name: 'auth' });
      }
    });
  },
};
</script>

<style lang="scss" scoped>
@import "@/assets/scss/_mixins.scss";

.personal {
  &__container {
    padding-top: 35px;
    padding-bottom: 70px;
  }

  &__heading {
    margin-bottom: 26px;
    font-size: 32px;
    line-height: 32px;
    font-weight: 400;
    color: var(--black);
  }

  &__head {
    display: flex;
    align-items: center;
    justify-content: space-between;
    margin-bottom: 26px;
  }

  &__subheading {
    font-size: 24px;
    line-height: 24px;
    font-weight: 400;
    color: var(--black);

    &--history {
      margin-bottom: 26px;
    }
  }

  &__edit-button {
    @include btn-primary;
    padding: 20px 25px;
  }

  &__form {
    margin-bottom: 36px;
    display: flex;
    flex-wrap: wrap;
    gap: 15px;
  }
}

.spinner div {
  border-color: var(--white) transparent transparent transparent;
}

.modal-close {
  animation: close .3s ease-in-out;
}

@keyframes close {
  from {
    opacity: 1;
  }

  to {
    opacity: 0;
  }
}
</style>
