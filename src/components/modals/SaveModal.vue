<template>
  <modal-inner class="modal__inner-1--settings" aria-label="Settings">
    <div class="modal__content">
      <tabble>
        <tr>
          <th>文章标签</th>
          <td>
            <form-entry label="">
              <input slot="field" class="textfield" type="text">
            </form-entry>
          </td>
        </tr>
        <tr>
          <th>系统分类</th>
          <td>
            <form-entry label="">
              <input slot="field" class="textfield" type="text">
            </form-entry>
          </td>
        </tr>
        <tr>
          <th>个人分类</th>
          <td>
            <form-entry label="">
              <input slot="field" class="textfield" type="text">
            </form-entry>
          </td>
        </tr>
        <tr>
          <th>文章类型</th>
          <td>
            <form-entry label="">
              <input slot="field" class="textfield" type="text">
            </form-entry>
          </td>
        </tr>
        <tr>
          <th>阅读权限</th>
          <td>
            <form-entry label="">
              <input slot="field" class="textfield" type="text">
            </form-entry>
          </td>
        </tr>
      </tabble>
    </div>
    <div class="modal__button-bar">
      <button class="button" @click="config.reject()">Cancel</button>
      <button class="button button--resolve" @click="resolve">Ok</button>
    </div>
  </modal-inner>
</template>

<script>
  import yaml from 'js-yaml';
  import { mapGetters } from 'vuex';
  import ModalInner from './common/ModalInner';
  import Tab from './common/Tab';
  import CodeEditor from '../CodeEditor';
  import defaultSettings from '../../data/defaults/defaultSettings.yml';
  import store from '../../store';
  import badgeSvc from '../../services/badgeSvc';

  const emptySettings = `# Add your custom settings here to override the
# default settings.
`;

  export default {
    components: {
      ModalInner,
      Tab,
      CodeEditor,
    },
    data: () => ({
      defaultSettings,
      customSettings: null,
    }),
    computed: {
      ...mapGetters('modal', [
        'config',
      ]),
      strippedCustomSettings() {
        return this.customSettings === emptySettings ? '\n' : this.customSettings.replace(/\t/g, '  ');
      },
    },
    created() {
      const settings = store.getters['data/settings'];
      this.setCustomSettings(settings === '\n' ? emptySettings : settings);
    },
    methods: {
      setCustomSettings(value) {
        this.customSettings = value;
        try {
          yaml.safeLoad(this.strippedCustomSettings);
          this.error = null;
        } catch (e) {
          this.error = e.message;
        }
      },
      async resolve() {
        if (!this.error) {
          const settings = this.strippedCustomSettings;
          await store.dispatch('data/setSettings', settings);
          const customSettings = yaml.safeLoad(settings);
          if (customSettings.shortcuts) {
            badgeSvc.addBadge('changeShortcuts');
          }
          const computedSettings = store.getters['data/computedSettings'];
          const customSettingsCount = Object
            .keys(customSettings)
            .filter(key => key !== 'shortcuts' && computedSettings[key])
            .length;
          if (customSettingsCount) {
            badgeSvc.addBadge('changeSettings');
          }
          this.config.resolve(settings);
        }
      },
    },
  };
</script>

<style lang="scss">
  @import '../../styles/variables.scss';

  .modal__inner-1.modal__inner-1--settings {
    max-width: 560px;
  }

  .modal__error--settings {
    white-space: pre-wrap;
    font-family: $font-family-monospace;
    font-size: $font-size-monospace;
  }
</style>
