<template>
  <div class="onboarding-container">

    <div v-if="step === 0" class="d-flex flex-column align-items-center justify-content-center p-2 min-vh100">
      <div class="w-75 d-flex flex-column align-items-center justify-content-center">
        <logo class="mb-3 logo" />
        <h1 class="text-center text-white text-lowercase mb-2">Добро пожаловать в Ноосферу</h1>
        <p class="text-center text-white">Ваш личный сервер готов к установке.</p>
        <transition name="slide-up" appear>
          <b-button
            size="lg"
            class="btn-start mt-4 mx-auto d-block px-4"
            @click="next"
          >Начать</b-button>
        </transition>
      </div>
    </div>
    
    <div v-else class="d-flex flex-column align-items-center justify-content-center min-vh100">
      <logo class="ml-3 mt-3 logo logo-small d-none d-sm-block" />
      <logo class="logo logo-xs d-sm-none" />
        <div v-if="step === 1" :key="1" class="card-glass px-3 px-sm-4 pt-5 pb-5 mt-5 mt-sm-0 mx-2">
          <h3 class="text-center text-white mb-2">Создать аккаунт</h3>
          <p class="text-center text-white mb-2">Информация о вашей учетной записи хранится только на вашем сервере. Обязательно сделайте резервную копию своего пароля, так как его невозможно сбросить.</p>
          <div class="form-container mt-4 d-flex flex-column form-container w-100 align-items-center">
            <div class="position-relative w-100">
              <b-form-input
                v-model="name"
                ref="name"
                placeholder="Имя"
                class="glass-input w-100"
                @input="resetError('nameError')"
                @blur="validateName"
                autofocus
              ></b-form-input>
              <small v-if="nameError" class="text-left text-white w-100">{{ nameError }}</small>
            </div>
            <div class="d-block py-2"></div>
            <div class="position-relative w-100">
              <input-password
                v-model="password"
                ref="password"
                placeholder="Пароль"
                inputGroupClass="glass-input-group"
                inputClass="glass-input w-100"
                @input="resetError('passwordError')"
                @blur="validatePassword"
              />
              <small v-if="passwordError" class="text-left text-white w-100">{{ passwordError }}</small>
            </div>
            <div class="d-block py-2"></div>
            <div class="position-relative w-100">
              <input-password
                v-model="confirmPassword"
                ref="confirmPassword"
                placeholder="Подтверждение пароля"
                inputGroupClass="glass-input-group"
                inputClass="glass-input w-100"
                @input="resetError('confirmPasswordError')"
                @blur="validateConfirmPassword"
              />
              <small v-if="confirmPasswordError" class="text-left text-white w-100">{{ confirmPasswordError }}</small>
            </div>
            <div class="d-block py-2"></div>
            <b-button
              class="btn-next mt-2 d-block px-4"
              @click="createAccount"
              :disabled="isRegistering"
            >{{ isRegistering ? 'Creating' : 'Создать' }}</b-button>
          </div>
        </div>
        <div v-if="step === 2" :key="2" class="card-glass px-3 px-sm-4 pt-5 pb-5 mt-5 mt-sm-0 mx-2">
          <h3 class="text-center text-white mb-2">Поздравляем 🎉</h3>
          <p class="text-center text-white mb-2">Все готово.</p>
          <div class="form-container mt-4 d-flex flex-column form-container w-100 align-items-center">
            <p class="opacity-80 text-center text-white">Нажимая «Далее», вы соглашаетесь с тем, что Ноосфера находится в стадии бета-тестирования и не может считаться полностью безопасной..</p>
          </div>
          <div class="d-flex flex-column align-items-center">
          <b-button
            class="btn-next mt-2 d-block px-4"
            @click="next"
          >Далее</b-button>
          </div>
        </div>
    </div>
  </div>
</template>

<script>
import Vue from "vue";
import VueConfetti from "vue-confetti";
import { mapState } from "vuex";

import InputPassword from "@/components/Utility/InputPassword";
import Logo from '@/components/Logo.vue';

Vue.use(VueConfetti);

export default {
  data() {
    return {
      name: "",
      password: "",
      confirmPassword: "",
      step: 0,
      isRegistering: false,
      nameError: "",
      passwordError: "",
      confirmPasswordError: ""
    };
  },
  computed: {
    ...mapState({
      registered: state => state.user.registered,
    })
  },
  methods: {
    next() {
      return this.step === 2 ? this.$router.push({name: 'home'}) : this.step++;
    },
    async createAccount() {
      this.validateName();
      this.validatePassword();
      this.validateConfirmPassword();
      if (this.nameError || this.passwordError || this.confirmPasswordError) {
        return;
      }
      this.isRegistering = true;
      try {
        await this.$store.dispatch("user/register", {
          name: this.name,
          password: this.password,
        });
      } catch (error) {
        this.isRegistering = false;
        if (error.response && error.response.data) {
          this.$bvToast.toast(`${error.response.data}`, {
            title: "Error",
            autoHideDelay: 3000,
            variant: "danger",
            solid: true,
            toaster: "b-toaster-bottom-right"
          });
        }
        return;
      }
      this.isRegistering = false;

      this.$confetti.start({
        particles: [
          {
            type: "rect"
          }
        ]
      });
      window.setTimeout(() => {
        this.$confetti.stop();
      }, 3000);
      this.step = 2;
    },
    resetError(error) {
      this[error] = '';
    },
    validateName() {
      if (!this.name.length) {
        this.nameError = 'Пожалуйста введите Имя для продолжения';
        return false;
      }
    },
    validatePassword() {
      if (this.password.length < 12) {
        this.passwordError = 'Пароль должен состоять минимум из 12 символов';
        return false;
      }
    },
    validateConfirmPassword() {
      if (!this.confirmPassword) {
        this.confirmPasswordError = 'Пожалуйста подтвердите свой пароль';
        return false;
      }
      if (this.password !== this.confirmPassword) {
        this.confirmPasswordError = 'Пароль не совпадает';
        return false;
      }
    }
  },
  created() {
    if (this.registered) {
      return this.$router.push({name: "home"});
    }
  },
  components: {
    InputPassword,
    Logo,
  }
};
</script>

<style lang="scss" scoped>

.onboarding-container {
  background-size: cover;
  position: relative;
}

.card-glass {
  max-width: 500px;
}

.btn-start {
  border-radius: 2.25rem;
  background: #fff;
  color: rgba(47, 47, 47, 0.8);
  font-weight: 700;
  letter-spacing: 0.2px;
}

.btn-next {
  border-radius: 2.25rem;
  font-weight: 500;
  letter-spacing: 0.2px;
  background-image: linear-gradient(93.78deg, rgba(255, 255, 255, 0.36) 0%, rgba(236, 235, 235, 0.35) 100%);
  background-color: transparent;
  text-align: center;
  text-transform: uppercase;
  color: #FFFFFF;
  border: solid 2px rgba(255, 255, 255, 0.2) !important;
  padding: 0.5rem 2.25rem;
}

.btn-start:not(:disabled):not(.disabled):active {
  background: rgba(255, 255, 255, 0.9);
  color: rgba(47, 47, 47, 0.8);
}

.logo {
  height: 20vh;
  max-height: 200px;
  width: auto;
    &.logo-small {
      position: absolute;
      top: 0;
      left: 0;
      height: 10vh;
      max-height: 100px;
    }
    &.logo-xs {
      position: absolute;
      top: 40px;
      height: 10vh;
      max-height: 100px;
  }
}

.form-container {
  max-width: 100%;
}

.dot-indicator {
  display: flex;
  flex-wrap: wrap;
  gap: 0.4rem;
  .dot {
    display: block;
    height: 0.4rem;
    width: 0.4rem;
    border-radius: 0.4rem;
    background: rgba(255, 255, 255, 0.3);
    &.filled {
      background: #fff;
    }
  }
}

.onboarding-progress {
  position: absolute;
  width: 100%;
  top: 0;
  left: 0;
  border-radius: 0;
  background: transparent;
}


.slide-up-enter-active {
  transition: transform 0.8s, opacity 0.8s ease;
  transition-delay: 0.4s;
}
.slide-up-leave-active {
  transition-duration: 0s;
}
.slide-up-enter {
  transform: translate3d(0, 10px, 0);
  opacity: 0;
}
.slide-up-enter-to {
  transform: translate3d(0, 0, 0);
  opacity: 1;
}
</style>

