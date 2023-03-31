<template>
  <div class="px-2 px-sm-4 pb-2 pb-sm-3">
    <!-- <span>Введите пароль, чтобы увидеть свою seed-фразу, состоящую из 24 слов</span> -->
    <div class="px-0 px-lg-4" v-if="!showSeed">
      <label class="mb-2">Введите пароль</label>
      <input-password
        v-model="password"
        ref="password"
        placeholder="Пароль"
        inputGroupClass="neu-input-group"
        :inputClass="[
                    isIncorrectPassword ? 'incorrect-password' : '',
                    'form-control form-control-lg neu-input w-100'
                  ]"
        :disabled="isLoadingSeed"
      />

      <label v-if="otpEnabled" class="mt-3 mb-0">Введите ваш код для двухфакторной аутентификации</label>
      <div class="seed-input-otp-container">
        <input-otp-token
          v-if="otpEnabled"
          :success="isCorrectOtp"
          :error="isIncorrectOtp"
          :disabled="isLoadingSeed"
          @otpToken="setOtpToken"
        />
      </div>

      <small
        class="mt-2 text-danger error float-right"
        v-show="isIncorrectPassword"
      >Неверный пароль</small>

      <small
        class="mt-2 text-danger error float-right"
        v-show="isIncorrectOtp"
      >Неверный код</small>

      <b-button
        variant="success"
        class="mt-4 mb-2"
        :disabled="isLoadingSeed"
        @click="fetchSeed"
      >
        {{
        isLoadingSeed ? "Расшифровка мнемонической фразы..." : "Показать мнемоническую фразу"
        }}
      </b-button>
    </div>

    <div class="d-flex justify-content-center" v-else>
      <!-- Seed phrase -->
      <seed :words="seed" v-if="seed.length"></seed>
      <b-spinner v-else></b-spinner>
    </div>
  </div>
</template>

<script>
import delay from "@/helpers/delay";

import InputPassword from "@/components/Utility/InputPassword";
import InputOtpToken from "@/components/Utility/InputOtpToken";
import Seed from "@/components/Utility/Seed";
import { mapState } from "vuex";

export default {
  data() {
    return {
      showSeed: false,
      password: "",
      isIncorrectPassword: false,
      isLoadingSeed: false,
      otpToken: "",
      isCorrectOtp: false,
      isIncorrectOtp: false
    };
  },
  computed: {
    ...mapState({
      seed: state => state.user.seed,
      otpEnabled: state => state.user.otpEnabled
    })
  },
  props: { progress: Number },
  created() {},
  methods: {
    setOtpToken(otpToken) {
      this.otpToken = otpToken;
    },
    async fetchSeed() {
      this.isLoadingSeed = true;
      try {
        await this.$store.dispatch("user/getSeed", { password: this.password, otpToken: this.otpToken });
        if (this.otpToken) {
          this.isCorrectOtp = true;
          // delay for ripple animation to complete
          await delay(1000);
        }
        this.showSeed = true;
      } catch (error) {
        if ( error.response && error.response.data ) {

          if (error.response.data === "Incorrect password") {
            this.isIncorrectPassword = true;
          }
          if (error.response.data === "Invalid OTP Token") {
            this.isIncorrectOtp = true;
          }
        }
      }
      this.isLoadingSeed = false;
      // delay for ripple animation to complete
      // and to hide any incorrect password error
      await delay(1000);
      this.isIncorrectPassword = false;
      this.isCorrectOtp = false;
      this.isIncorrectOtp = false;
    }
  },
  components: {
    InputPassword,
    InputOtpToken,
    Seed
  }
};
</script>

<style lang="scss">
</style>
