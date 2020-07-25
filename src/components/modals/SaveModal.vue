<template>
  <modal-inner aria-label="Insert image">
    <div class="modal__content">
      <table>
        <tr>
          <th>文章标签</th>
          <td>
            <form-entry label="">
              <input slot="field" class="textfield" type="text"/>
            </form-entry>
          </td>
        </tr>
        <tr>
          <th>系统分类</th>
          <td>
            <form-entry label="">
              <select slot="field" class="textfield" type="text">
                <option>web前端</option>
              </select>
            </form-entry>
          </td>
        </tr>
        <tr>
          <th>个人分类</th>
          <td>
            <form-entry label="">
              <select slot="field" class="textfield" type="text">
                <option>web前端</option>
              </select>
            </form-entry>
          </td>
        </tr>
        <tr>
          <th>文章分类</th>
          <td>
            <div>
              <input  name="atype" id="atype_1" type="radio" /><label for="atype_1">原创</label>
              <input  name="atype" id="atype_2" type="radio" /><label for="atype_2">转载</label>
              <input  name="atype" id="atype_3" type="radio" /><label for="atype_3">翻译</label>
            </div>
          </td>
        </tr>
        <tr>
          <th>阅读权限</th>
          <td>
            <div>
              <input  name="rtype" id="rtype_1" type="radio" /><label for="rtype_1">公开</label>
              <input  name="rtype" id="rtype_2" type="radio" /><label for="rtype_2">粉丝可见</label>
              <input  name="rtype" id="rtype_3" type="radio" /><label for="rtype_3">付费可见</label>
              <input  name="rtype" id="rtype_4" type="radio" /><label for="rtype_4">私密</label>
            </div>
          </td>
        </tr>

      </table>
    </div>
    <div class="modal__button-bar">
      <button class="button" @click="reject()">保存草稿</button>
      <button class="button button--resolve" @click="resolve">发送</button>
    </div>
  </modal-inner>
</template>

<script>
  import modalTemplate from './common/modalTemplate';
  import MenuEntry from '../menus/common/MenuEntry';
  import googleHelper from '../../services/providers/helpers/googleHelper';
  import store from '../../store';

  export default modalTemplate({
    components: {
      MenuEntry,
    },
    data: () => ({
      url: '',
    }),
    computed: {
      googlePhotosTokens() {
        const googleTokensBySub = store.getters['data/googleTokensBySub'];
        return Object.values(googleTokensBySub)
          .filter(token => token.isPhotos)
          .sort((token1, token2) => token1.name.localeCompare(token2.name));
      },
    },
    methods: {
      resolve(evt) {
        evt.preventDefault(); // Fixes https://github.com/benweet/stackedit/issues/1503
        if (!this.url) {
          this.setError('url');
        } else {
          const { callback } = this.config;
          this.config.resolve();
          callback(this.url);
        }
      },
      reject() {
        const { callback } = this.config;
        this.config.reject();
        callback(null);
      },
      async addGooglePhotosAccount() {
        try {
          await googleHelper.addPhotosAccount();
        } catch (e) { /* cancel */ }
      },
      async openGooglePhotos(token) {
        const { callback } = this.config;
        this.config.reject();
        const res = await googleHelper.openPicker(token, 'img');
        if (res[0]) {
          store.dispatch('modal/open', {
            type: 'googlePhoto',
            url: res[0].url,
            callback,
          });
        }
      },
    },
  });
</script>
