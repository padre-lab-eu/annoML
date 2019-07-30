<template>
  <div>
    <b-card class="mb-2 ml-4">
      <annotation-select
        class="annotation-select"
        v-if="
          $store.getters.visualizationSelectable ||
            pointAnnotations.length > 0 ||
            rectangleAnnotations.length > 0
        "
        :point-annotations="pointAnnotations"
        :rectangle-annotations="rectangleAnnotations"
        :annotation-color="comment.color"
        :edit="true"
        @select-annotation="selectAnnotation"
        @delete-annotation="deleteAnnotation"
        @update-annotation="updateAnnotation"
        @update-color="updateAnnotationColor"
      />
      <b-button
        :disabled="$store.getters.visualizationSelectable"
        class="float-right"
        size="sm"
        @click="addNewAnnotation"
        variant="outline-primary"
        >Add Annotation
      </b-button>
      <div class="editor">
        <div class="editor-toolbar">
          <b-button-toolbar>
            <editor-menu-bar
              class="menubar"
              :editor="editor"
              v-slot="{ commands, isActive }"
            >
              <b-button-group size="sm">
                <b-button
                  class="menubar__b-button"
                  :class="{ active: isActive.bold() }"
                  @click="commands.bold"
                >
                  <i class="fa fa-bold"
                /></b-button>

                <b-button
                  class="menubar__b-button"
                  :class="{ active: isActive.italic() }"
                  @click="commands.italic"
                >
                  <i class="fa fa-italic"
                /></b-button>

                <b-button
                  class="menubar__b-button"
                  :class="{ active: isActive.strike() }"
                  @click="commands.strike"
                >
                  <i class="fa fa-strikethrough"
                /></b-button>

                <b-button
                  class="menubar__b-button"
                  :class="{ active: isActive.underline() }"
                  @click="commands.underline"
                >
                  <i class="fa fa-underline"
                /></b-button>

                <b-button
                  class="menubar__button"
                  :class="{ active: isActive.bullet_list() }"
                  @click="commands.bullet_list"
                >
                  <i class="fa fa-list-ul" />
                </b-button>

                <b-button
                  class="menubar__button"
                  :class="{ active: isActive.ordered_list() }"
                  @click="commands.ordered_list"
                >
                  <i class="fa fa-list-ol" />
                </b-button>

                <b-button
                  class="menubar__button"
                  :class="{ active: isActive.blockquote() }"
                  @click="commands.blockquote"
                >
                  <i class="fa fa-quote-left" />
                </b-button>

                <b-button
                  class="menubar__button"
                  :class="{ active: isActive.code_block() }"
                  @click="commands.code_block"
                >
                  <i class="fa fa-code" />
                </b-button>
                <b-button class="menubar__button" @click="commands.undo">
                  <i class="fa fa-undo" />
                </b-button>

                <b-button class="menubar__button" @click="commands.redo">
                  <i class="fa fa-undo icon-flipped" />
                </b-button>
              </b-button-group>
            </editor-menu-bar>
          </b-button-toolbar>
        </div>
        <editor-content class="editor__content" id="editor" :editor="editor" />
      </div>

      <b-button v-if="comment.id" @click="updateComment" variant="success"
        >Save
      </b-button>
      <b-button v-else @click="submitComment" variant="success"
        >Submit</b-button
      >
      <b-button @click="deleteComment" variant="danger">Delete </b-button>
    </b-card>
  </div>
</template>

<script>
/* eslint-disable vue/require-default-prop,no-console,no-alert */
import { Editor, EditorContent, EditorMenuBar } from 'tiptap';
import {
  Blockquote,
  CodeBlock,
  HardBreak,
  HorizontalRule,
  OrderedList,
  BulletList,
  ListItem,
  TodoItem,
  TodoList,
  Bold,
  Code,
  Italic,
  Link,
  Strike,
  Underline,
  History,
} from 'tiptap-extensions';
import AnnotationSelect from '@/components/discussion/annotation/AnnotationSelect.vue';

export default {
  name: 'CommentEditor',
  components: {
    EditorContent,
    EditorMenuBar,
    AnnotationSelect,
  },
  props: {
    comment: {
      type: Object,
      default() {
        return {};
      },
    },
    pointAnnotations: {
      type: Array,
      default() {
        return [];
      },
    },
    rectangleAnnotations: {
      type: Array,
      default() {
        return [];
      },
    },
  },
  data() {
    return {
      editor: new Editor({
        extensions: [
          new Blockquote(),
          new BulletList(),
          new CodeBlock(),
          new HardBreak(),
          new HorizontalRule(),
          new ListItem(),
          new OrderedList(),
          new TodoItem(),
          new TodoList(),
          new Bold(),
          new Code(),
          new Italic(),
          new Link(),
          new Strike(),
          new Underline(),
          new History(),
        ],
        content: this.comment.body,
      }),
    };
  },
  created() {},
  watch: {},
  methods: {
    /**
     * Answer Operations
     */
    submitComment() {
      if (this.editorHasContent()) {
        this.clearAnnotation();
        const post = {
          body: this.editor.getJSON(),
          author: {
            username: '',
          },
          pointAnnotations: this.pointAnnotations,
          rectangleAnnotations: this.rectangleAnnotations,
        };
        if (
          this.pointAnnotations.length > 0
          || this.rectangleAnnotations.length > 0
        ) {
          post.color = this.comment.color;
        }
        this.$emit('save-comment', post);
      } else {
        alert('Enter a content');
      }
    },
    updateComment() {
      if (this.editorHasContent()) {
        this.pointAnnotations.forEach((a) => {
          const pointAnnotation = a;
          pointAnnotation.color = this.comment.color;
          console.log(pointAnnotation);
          return pointAnnotation;
        });
        this.rectangleAnnotations.forEach((a) => {
          const rectangleAnnotation = a;
          rectangleAnnotation.color = this.comment.color;
          return rectangleAnnotation;
        });
        const post = {
          id: this.comment.id,
          color: this.comment.color,
          pointAnnotations: this.pointAnnotations,
          rectangleAnnotations: this.rectangleAnnotations,
        };
        this.$emit('update-comment', post);
      } else {
        alert('Enter title and content');
      }
    },
    deleteComment() {
      this.$emit('delete-comment', this.comment);
    },
    /**
     * Annotation Handling
     */
    selectAnnotation(annotation) {
      if (annotation.color === 'gray') {
        this.clearAnnotation();
      } else {
        this.pointAnnotations.forEach((a) => {
          const pointAnnotation = a;
          if (pointAnnotation.id === annotation.id) {
            pointAnnotation.color = 'gray';
          } else {
            pointAnnotation.color = this.post.color;
          }
        });
        this.rectangleAnnotations.forEach((a) => {
          const rectangleAnnotation = a;
          if (rectangleAnnotation.id === annotation.id) {
            rectangleAnnotation.color = 'gray';
          } else {
            rectangleAnnotation.color = this.post.color;
          }
        });
      }
    },
    addNewAnnotation() {
      this.$store.commit('enableSelectable');
    },
    clearAnnotation() {
      this.pointAnnotations.forEach((a) => {
        const pointAnnotation = a;
        pointAnnotation.color = this.comment.color;
      });
      this.rectangleAnnotations.forEach((a) => {
        const rectangleAnnotation = a;
        rectangleAnnotation.color = this.comment.color;
      });
    },
    deleteAnnotation(annotation) {
      if (annotation.type === 'POINT') {
        this.pointAnnotations.filter(a => a.id !== annotation.id);
      } else if (annotation.type === 'RECTANGLE') {
        this.rectangleAnnotations.filter(a => a.id !== annotation.id);
      }
    },
    updateAnnotation(annotation) {
      if (annotation.annotationType === 'POINT') {
        this.$set(
          this.pointAnnotations,
          this.pointAnnotations.findIndex(a => a.id === annotation.id),
          annotation,
        );
      } else if (annotation.annotationType === 'RECTANGLE') {
        this.$set(
          this.rectangleAnnotations,
          this.rectangleAnnotations.findIndex(a => a.id === annotation.id),
          annotation,
        );
      }
    },
    updateAnnotationColor(value) {
      this.comment.color = value;
      this.pointAnnotations.forEach((a) => {
        const pointAnnotation = a;
        if (pointAnnotation.color !== 'gray') {
          pointAnnotation.color = value;
        }
      });
      this.rectangleAnnotations.forEach((a) => {
        const rectangleAnnotation = a;
        if (rectangleAnnotation.color !== 'gray') {
          rectangleAnnotation.color = value;
        }
      });
    },
    /**
     * Helpers
     */
    editorHasContent() {
      const body = this.editor.getHTML();
      return body !== '<p></p>';
    },
  },
  beforeDestroy() {
    this.editor.destroy();
  },
};
</script>

<style lang="scss">
$color-black: #000000;
$color-white: #ffffff;
$color-grey: #dddddd;

#question-title {
  border: none;
  font-size: 2rem;
  width: 100%;
}

#question-title:focus {
  outline: none;
}

.annotation-select {
  margin-top: 0.5rem;
  margin-bottom: 0.5rem;
}
.editor-toolbar {
  margin-bottom: 0.5rem;
}
.icon-flipped {
  transform: scaleX(-1);
  -moz-transform: scaleX(-1);
  -webkit-transform: scaleX(-1);
  -ms-transform: scaleX(-1);
}

.editor {
  margin-top: 0.5rem;
  margin-bottom: 0.5rem;

  .ProseMirror {
    resize: vertical;
    position: relative;
    overflow-y: scroll;
    height: calc(1.5em + 5rem + 2px);
    padding: 0.375rem 0.75rem;
    font-size: 1rem;
    font-weight: 400;
    line-height: 1.5;
    color: #495057;
    background-color: #fff;
    background-clip: padding-box;
    border: 1px solid #ced4da;
    border-radius: 0.25rem;
  }

  .ProseMirror p {
  }

  .ProseMirror-focused {
    outline: none;
  }

  &__content {
    word-wrap: break-word;

    * {
      caret-color: currentColor;
    }

    p {
      margin-top: 0.2rem;
      margin-bottom: 0.2rem;
    }

    pre {
      padding: 0.7rem 1rem;
      border-radius: 5px;
      background: $color-black;
      color: $color-white;
      font-size: 0.8rem;
      overflow-x: auto;

      code {
        display: block;
      }
    }

    p code {
      display: inline-block;
      padding: 0 0.4rem;
      border-radius: 5px;
      font-size: 0.8rem;
      font-weight: bold;
      background: rgba($color-black, 0.1);
      color: rgba($color-black, 0.8);
    }

    ul,
    ol {
      padding-left: 1rem;
    }

    li > p,
    li > ol,
    li > ul {
      margin: 0;
    }

    a {
      color: inherit;
    }

    blockquote {
      border-left: 3px solid rgba($color-black, 0.1);
      color: rgba($color-black, 0.8);
      padding-left: 0.8rem;
      font-style: italic;

      p {
        margin: 0;
      }
    }

    img {
      max-width: 100%;
      border-radius: 3px;
    }

    table {
      border-collapse: collapse;
      table-layout: fixed;
      width: 100%;
      margin: 0;
      overflow: hidden;

      td,
      th {
        min-width: 1em;
        border: 2px solid $color-grey;
        padding: 3px 5px;
        vertical-align: top;
        box-sizing: border-box;
        position: relative;
        > * {
          margin-bottom: 0;
        }
      }

      th {
        font-weight: bold;
        text-align: left;
      }

      .selectedCell:after {
        z-index: 2;
        position: absolute;
        content: "";
        left: 0;
        right: 0;
        top: 0;
        bottom: 0;
        background: rgba(200, 200, 255, 0.4);
        pointer-events: none;
      }

      .column-resize-handle {
        position: absolute;
        right: -2px;
        top: 0;
        bottom: 0;
        width: 4px;
        z-index: 20;
        background-color: #adf;
        pointer-events: none;
      }
    }

    .tableWrapper {
      margin: 1em 0;
      overflow-x: auto;
    }

    .resize-cursor {
      cursor: ew-resize;
      cursor: col-resize;
    }
  }
}
</style>