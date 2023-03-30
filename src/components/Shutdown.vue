<template>
  <div class="d-flex flex-column align-items-center justify-content-center min-vh100 p-2">
    <logo
      class="mb-5 logo"
      :class="hasShutdown ? 'logo-gray' : ''"
     />
    <span class="text-white w-75 text-center">
      <small
        v-if="hasShutdown"
      >Теперь вы можете безопасно отключить питание вашей системы и интернет.</small>
      <small v-else-if="shuttingDown">
        <b-spinner small class="mr-2"></b-spinner>Выключение...
      </small>
      <small v-else-if="rebooting">
        <b-spinner small class="mr-2"></b-spinner>Перезагрузка...
      </small>
    </span>
    <slot></slot>
  </div>
</template>

<script>
import Logo from '@/components/Logo.vue';

export default {
  data() {
    return {};
  },
  created() {},
  methods: {},
  components: {
    Logo,
  },
  props: {
    hasShutdown: Boolean,
    shuttingDown: Boolean,
    rebooting: Boolean
  }
};
</script>

<style lang="scss" scoped>
.logo {
  height: 20vh;
  max-height: 200px;
  width: auto;
  transition: filter 0.3s ease;
  &.logo-gray {
    filter: grayscale(100%);
  }
}
</style>
