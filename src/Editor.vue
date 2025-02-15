<template>
    <div>
        <!-- Editor Mode -->
        <div class="medium-editor-container" v-if="!readOnly">
            <insert-embed v-if="editor" 
                :uploadUrl="options.uploadUrl"
                :onChange="triggerChange"
                :editorRef="$refs.editor"
                :editor="editor"
                v-on:uploaded="uploadedCallback"></insert-embed>
            <list-handler v-if="editor"
                :editor="editor"
                :onChange="triggerChange"></list-handler>
            <div class="editor" 
                v-bind:class="editorClass"
                v-html="prefill"
                ref="editor">
            </div>
        </div>
        <!-- Read Only Mode -->
        <read-mode v-if="readOnly" :content="prefill"></read-mode>
    </div>
</template>

<script>
import MediumEditor from 'medium-editor';
import InsertEmbed from './libs/InsertEmbed';
import ListHandler from './libs/ListHandler';
import ReadMode from './libs/ReadMode';
import _ from 'underscore';

export default {
  name: "medium-editor",
  data() {
    return {
      editor: null,
      defaultOptions: {
        forcePlainText: false,
        placeholder: {
          text: "Write something great!!"
        },
        toolbar: {
          buttons: ["bold", "italic", "quote", "h1", "h2", "h3", "h4", "h5"]
        }
      },
      hasContent: false
    };
  },
  props: ["options", "onChange", "prefill", "readOnly", "content"],
  computed: {
    editorOptions() {
      return _.extend(this.defaultOptions, this.options);
    },
    editorClass() {
        return {
            'has-content': this.hasContent
        }
    }
  },
  components: {
    InsertEmbed,
    ListHandler,
    ReadMode
  },
  mounted() {
    if (!this.readOnly) {
      this.createElm();
    }
  },
  methods: {
    createElm() {
      this.editor = new MediumEditor(this.$refs.editor, this.editorOptions);

      if (this.prefill) {
        if (/<[a-z][\s\S]*>/i.test(this.prefill)) {
          this.hasContent = true;
        } else {
          this.hasContent = false;
        }
        this.$refs.editor.focus();
      }

      this.editor.subscribe("editableInput", this.triggerChange);
    },
    destroyElm() {
      this.editor.destroy();
    },
    triggerChange() {
      this.content = this.editor.getContent();

      setTimeout(() => {
        if (/<[a-z][\s\S]*>/i.test(this.content)) {
          this.hasContent = true;
        } else {
          this.hasContent = false;
        }
      }, 0);

      this.$emit("input", this.content);

      if (this.onChange) {
        this.onChange(this.content);
      }
    },
    uploadedCallback(url) {
      console.log("callback")
      this.$emit("uploaded", url);
    }
  },
  destroyed() {
    this.destroyElm();
  }
};
</script>
