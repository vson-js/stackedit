<template>
  <div class="editor">
    <div class="editor-numbers" :style="{lineHeight: styles.numberLineHeight}"></div>
    <pre class="editor__inner markdown-highlighting" :style="{padding: styles.editorPadding}" :class="{monospaced: computedSettings.editor.monospacedFontOnly}"></pre>
    <div class="gutter" :style="{left: styles.editorGutterLeft + 'px'}">
      <comment-list v-if="styles.editorGutterWidth"></comment-list>
      <editor-new-discussion-button v-if="!isCurrentTemp"></editor-new-discussion-button>
    </div>
  </div>
</template>

<script>
import { mapGetters } from 'vuex';
import CommentList from './gutters/CommentList';
import EditorNewDiscussionButton from './gutters/EditorNewDiscussionButton';
import editorSvc from '../services/editorSvc';
import store from '../store';

export default {
  components: {
    CommentList,
    EditorNewDiscussionButton,
  },
  computed: {
    ...mapGetters('file', [
      'isCurrentTemp',
    ]),
    ...mapGetters('layout', [
      'styles',
    ]),
    ...mapGetters('data', [
      'computedSettings',
    ]),
  },
  data(){
    return {
      line:0
    }
  },
  methods:{
    computeHtml(){
      setTimeout(() => {
        const text = editorSvc.clEditor.getContent();
        this.line=(text.match(new RegExp('\n', 'gm')) || []).length
      }, 10);
    },
    createLineNumber(newValue){
              const numbers = document.querySelector('.editor-numbers');
        numbers.innerHTML="";
        for(let i=1;i<=newValue;i++){
            numbers.appendChild(document.createTextNode(String(i)));
            numbers.appendChild(document.createElement('BR'));
          }
    }
  },
  created(){
    editorSvc.$on('previewCtx', () => this.computeHtml());
    editorSvc.$on('previewSelectionRange', () => this.computeHtml());
  },
  watch:{
    line:function(newValue){
      this.createLineNumber(newValue)
    }
  },
  mounted() {
    const editorElt = this.$el.querySelector('.editor__inner');
    const onDiscussionEvt = cb => (evt) => {
      let elt = evt.target;
      while (elt && elt !== editorElt) {
        if (elt.discussionId) {
          cb(elt.discussionId);
          return;
        }
        elt = elt.parentNode;
      }
    };

    const classToggler = toggle => (discussionId) => {
      editorElt.getElementsByClassName(`discussion-editor-highlighting--${discussionId}`)
        .cl_each(elt => elt.classList.toggle('discussion-editor-highlighting--hover', toggle));
      document.getElementsByClassName(`comment--discussion-${discussionId}`)
        .cl_each(elt => elt.classList.toggle('comment--hover', toggle));
    };

    editorElt.addEventListener('mouseover', onDiscussionEvt(classToggler(true)));
    editorElt.addEventListener('mouseout', onDiscussionEvt(classToggler(false)));
    editorElt.addEventListener('click', onDiscussionEvt((discussionId) => {
      store.commit('discussion/setCurrentDiscussionId', discussionId);
    }));

    this.$watch(
      () => store.state.discussion.currentDiscussionId,
      (discussionId, oldDiscussionId) => {
        if (oldDiscussionId) {
          editorElt.querySelectorAll(`.discussion-editor-highlighting--${oldDiscussionId}`)
            .cl_each(elt => elt.classList.remove('discussion-editor-highlighting--selected'));
        }
        if (discussionId) {
          editorElt.querySelectorAll(`.discussion-editor-highlighting--${discussionId}`)
            .cl_each(elt => elt.classList.add('discussion-editor-highlighting--selected'));
        }
      },
    );
  },
};
</script>

<style lang="scss">
@import '../styles/variables.scss';

.editor {
  position: absolute;
  width: 100%;
  height: 100%;
  overflow: auto;
}

.editor__inner {
  margin: 0;
  font-family: $font-family-main;
  font-variant-ligatures: no-common-ligatures;
  white-space: pre-wrap;
  word-break: break-word;
  word-wrap: break-word;

  * {
    line-height: $line-height-base;
  }

  .cledit-section {
    font-family: inherit;
  }

  .hide {
    display: none;
  }

  &.monospaced {
    font-family: $font-family-monospace !important;
    font-size: $font-size-monospace !important;

    * {
      font-size: inherit !important;
    }
  }
}

.editor-numbers {
  float: left;
  width: 2em;
  padding: 10px 2px;
  margin-right: 0.5em;
  text-align: right;
  font-family: monospace;
  color: #ccc;
  font-size: 14px;
  line-height: 2.2;
}
</style>
