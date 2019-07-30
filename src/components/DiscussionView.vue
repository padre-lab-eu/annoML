<template>
  <div class="discussion">
    <question
      v-for="question in questions.filter(q => q !== currentEdit)"
      :key="question.id"
      :question="question"
      @edit-question="editQuestion"
    />
    <question-editor
      v-if="currentEdit"
      :question="$store.getters.getCurrentPost"
      :point-annotations="$store.getters.currentPointAnnotations"
      :rectangle-annotations="$store.getters.currentRectangleAnnotations"
      @save-question="saveQuestion"
      @update-question="updateQuestion"
      @delete-question="deleteQuestion"
    />
  </div>
</template>

<script>
/* eslint-disable no-console,no-param-reassign */

import Question from '@/components/discussion/Question.vue';
import QuestionEditor from '@/components/discussion/QuestionEditor.vue';
import APIService from '@/services/APIService';

export default {
  name: 'DiscussionView',
  components: {
    Question,
    QuestionEditor,
  },
  props: {
    discussion: {
      type: Object,
      default() {
        return null;
      },
    },
  },
  data() {
    return {
      currentEdit: null,
      questions: [],
      editorOpen: false,
      maxHeight: 0,
    };
  },
  mounted() {
    this.$store.watch(
      (state, getters) => getters.currentPointAnnotations,
      (currentPointAnnotations) => {
        if (
          currentPointAnnotations.length > 0
          && !this.$store.getters.hasCurrentPost
        ) {
          this.createQuestion();
        }
      },
    );
    this.$store.watch(
      (state, getters) => getters.currentRectangleAnnotations,
      (currentRectangleAnnotations) => {
        if (
          currentRectangleAnnotations.length > 0
          && !this.$store.getters.hasCurrentPost
        ) {
          this.createQuestion();
        }
      },
    );
  },
  created() {
    this.questions = this.discussion.questions;
    this.questions.forEach((question) => {
      if (question.color) {
        this.$store.commit('addUsedColor', question.color);
      }
      if (question.pointAnnotations.length > 0) {
        this.$store.commit('addPointAnnotations', question.pointAnnotations);
      }
      if (question.rectangleAnnotations.length > 0) {
        this.$store.commit(
          'addRectangleAnnotations',
          question.rectangleAnnotations,
        );
      }
    });
  },
  methods: {
    /**
     * Question Events
     */
    createQuestion() {
      const newQuestion = {
        answers: [],
      };
      this.currentEdit = newQuestion;
      this.$store.commit('setCurrentPost', newQuestion);
    },
    saveQuestion(question) {
      this.currentEdit = null;
      this.$store.commit('removeCurrentPost');
      this.$store.commit('enableSelectable');
      this.$store.commit('mergeCurrentAnnotations');
      this.$store.commit('clearCurrentAnnotations');
      this.$store.commit('addUsedColor', question.color);
      this.questions.push(question);
      console.log(question);
      APIService.addQuestion(this.discussion.id, question).then((response) => {
        console.log(response);
        this.$set(
          this.questions,
          this.questions.findIndex(q => q.id === question.id),
          response,
        );
        if (response.pointAnnotations.length > 0) {
          this.$store.commit(
            'removePointAnnotations',
            question.pointAnnotations,
          );
          this.$store.commit('addPointAnnotations', response.pointAnnotations);
        }
        if (response.rectangleAnnotations.length > 0) {
          this.$store.commit(
            'removeRectangleAnnotations',
            question.rectangleAnnotations,
          );
          this.$store.commit(
            'addRectangleAnnotations',
            response.rectangleAnnotations,
          );
        }
        if (response.color !== question.color) {
          this.$store.commit('addUsedColor', question.color);
        }
      });
    },
    updateQuestion(question) {
      this.currentEdit = null;
      this.$store.commit('removeCurrentPost');
      this.$store.commit('enableSelectable');
      this.$store.commit('mergeCurrentAnnotations');
      this.$store.commit('clearCurrentAnnotations');
      APIService.updateQuestion(question).then((response) => {
        this.$set(
          this.questions,
          this.questions.findIndex(q => q.id === question.id),
          response,
        );
        if (response.pointAnnotations.length > 0) {
          this.$store.commit(
            'removePointAnnotations',
            question.pointAnnotations,
          );
          this.$store.commit('addPointAnnotations', response.pointAnnotations);
        }
        if (response.rectangleAnnotations.length > 0) {
          this.$store.commit(
            'removeRectangleAnnotations',
            question.rectangleAnnotations,
          );
          this.$store.commit(
            'addRectangleAnnotations',
            response.rectangleAnnotations,
          );
        }
        if (response.color !== question.color) {
          this.$store.commit('addUsedColor', question.color);
        }
      });
    },
    deleteQuestion(question) {
      this.currentEdit = null;
      this.$store.commit('removeCurrentPost');
      this.$store.commit('enableSelectable');
      this.$store.commit('clearCurrentAnnotations');
      if (question.color) {
        this.$store.commit('removeUsedColor', question.color);
      }
      this.questions.filter(q => q.id !== question.id);
      APIService.deleteQuestion(this.discussion.id, question).then((response) => {
        console.log(response);
        this.$set(
          this.questions,
          this.questions.findIndex(q => q.id === question.id),
          response,
        );
      });
    },
    editQuestion(question) {
      if (!this.$store.getters.hasCurrentPost) {
        this.currentEdit = question;
        this.$store.commit('setCurrentPost', question);
        if (question.pointAnnotations.length > 0) {
          this.$store.commit(
            'removePointAnnotations',
            question.pointAnnotations,
          );
          this.$store.commit(
            'setCurrentPointAnnotations',
            question.pointAnnotations,
          );
        }
        if (question.rectangleAnnotations.length > 0) {
          this.$store.commit(
            'removeRectangleAnnotations',
            question.rectangleAnnotations,
          );
          this.$store.commit(
            'setCurrentRectangleAnnotations',
            question.rectangleAnnotations,
          );
        }
      }
    },
  },
};
</script>

<style></style>