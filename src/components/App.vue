<template>
  <div class="app" :class="classes" @keydown.esc="close">
    <splash-screen v-if="!ready"></splash-screen>
    <layout v-else></layout>
    <modal></modal>
    <notification></notification>
    <context-menu></context-menu>
  </div>
</template>

<script>
import axios from 'axios';
import '../styles';
import '../styles/markdownHighlighting.scss';
import '../styles/app.scss';
import Layout from './Layout';
import Modal from './Modal';
import Notification from './Notification';
import ContextMenu from './ContextMenu';
import SplashScreen from './SplashScreen';
import syncSvc from '../services/syncSvc';
import networkSvc from '../services/networkSvc';
import tempFileSvc from '../services/tempFileSvc';
import store from '../store';
import './common/vueGlobals';
import config from '../config/index';

const themeClasses = {
  light: ['app--light'],
  dark: ['app--dark'],
};

export default {
  components: {
    Layout,
    Modal,
    Notification,
    ContextMenu,
    SplashScreen,
  },
  data: () => ({
    ready: false,
  }),
  computed: {
    classes() {
      const result = themeClasses[store.getters['data/computedSettings'].colorTheme];
      return Array.isArray(result) ? result : themeClasses.light;
    },
  },
  methods: {
    close() {
      tempFileSvc.close();
    },
    async getUserInfo() {
      const userToken = this.$cookies.get('user_token');
      try {
        const userInfo = await axios.get(`${config.baseUrl}${config.api.getUserInfo}/${userToken}`);
        store.commit('userInfo/setUserInfo', userInfo);
      } catch (e) {
        console.error(e);
      }
    },
  },
  async created() {
    try {
      await this.getUserInfo();
      await syncSvc.init();
      await networkSvc.init();
      this.ready = true;
      tempFileSvc.setReady();
    } catch (err) {
      if (err && err.message === 'RELOAD') {
        window.location.reload();
      } else if (err && err.message !== 'RELOAD') {
        console.error(err); // eslint-disable-line no-console
      }
    }
  },
};
</script>
