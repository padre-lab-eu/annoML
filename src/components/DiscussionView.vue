<template>
  <div class="discussion">
    <div v-for="question in questions" :key="question.id">
      <question-editor
        v-if="question === $annomlstore.getters.getCurrentPost"
        :question="question"
        :key="question.id"
        :point-annotations="$annomlstore.getters.currentPointAnnotations"
        :rectangle-annotations="
          $annomlstore.getters.currentRectangleAnnotations
        "
        :discussion="discussion"
        @save-question="saveQuestion"
        @update-question="updateQuestion"
        @delete-question="deleteQuestion"
      />
      <question
        v-else
        :question="question"
        :key="question.id"
        @edit-question="editQuestion"
        @up-vote-question="upVoteQuestion"
        @down-vote-question="downVoteQuestion"
      />
    </div>
  </div>
</template>

<script>
/* eslint-disable no-console,no-param-reassign */

import Question from './discussion/Question.vue';
import QuestionEditor from './discussion/QuestionEditor.vue';
import APIService from '../service/APIService';

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
      questions: [],
      editorOpen: false,
      maxHeight: 0,
    };
  },
  mounted() {},
  created() {
    this.questions = this.discussion.questions;
    this.questions.forEach((question) => {
      if (question.color) {
        this.$annomlstore.commit('addUsedColor', question.color);
      }
      if (question.pointAnnotations.length > 0) {
        this.$annomlstore.commit(
          'addPointAnnotations',
          question.pointAnnotations,
        );
      }
      if (question.rectangleAnnotations.length > 0) {
        this.$annomlstore.commit(
          'addRectangleAnnotations',
          question.rectangleAnnotations,
        );
      }
    });
    this.$annomlstore.watch(
      (state, getters) => getters.currentPointAnnotations,
      (currentPointAnnotations) => {
        if (
          currentPointAnnotations.length > 0
          && !this.$annomlstore.getters.hasCurrentPost
        ) {
          this.createQuestion();
        }
      },
    );
    this.$annomlstore.watch(
      (state, getters) => getters.currentRectangleAnnotations,
      (currentRectangleAnnotations) => {
        if (
          currentRectangleAnnotations.length > 0
          && !this.$annomlstore.getters.hasCurrentPost
        ) {
          this.createQuestion();
        }
      },
    );
  },
  methods: {
    /**
     * Question Events
     */
    createQuestion() {
      const newQuestion = {
        id: Date.now(),
        pointAnnotations: [],
        rectangleAnnotations: [],
        answers: [],
        author: null,
        upVotes: [],
        downVotes: [],
      };
      this.questions.push(newQuestion);
      this.$annomlstore.commit('setCurrentPost', newQuestion);
    },
    saveQuestion(question) {
      this.$annomlstore.commit('mergeCurrentAnnotations');
      this.$annomlstore.commit('clearCurrentAnnotations');
      this.$annomlstore.commit('addUsedColor', question.color);
      this.$annomlstore.commit('removeCurrentPost');
      this.$annomlstore.commit('enableSelectable');
      this.$set(
        this.questions,
        this.questions.findIndex(q => q.id === question.id),
        question,
      );
      APIService(this.$serviceApiAuthenticated)
        .addQuestion(this.discussion.id, question)
        .then((response) => {
          this.$set(
            this.questions,
            this.questions.findIndex(q => q.id === question.id),
            response,
          );
          if (response.pointAnnotations.length > 0) {
            this.$annomlstore.commit(
              'removePointAnnotations',
              question.pointAnnotations,
            );
            this.$annomlstore.commit(
              'addPointAnnotations',
              response.pointAnnotations,
            );
          }
          if (response.rectangleAnnotations.length > 0) {
            this.$annomlstore.commit(
              'removeRectangleAnnotations',
              question.rectangleAnnotations,
            );
            this.$annomlstore.commit(
              'addRectangleAnnotations',
              response.rectangleAnnotations,
            );
          }
          if (response.color !== question.color) {
            this.$annomlstore.commit('addUsedColor', question.color);
          }
        });
    },
    updateQuestion(question) {
      this.$annomlstore.commit('mergeCurrentAnnotations');
      this.$annomlstore.commit('clearCurrentAnnotations');
      this.$annomlstore.commit('removeCurrentPost');
      this.$annomlstore.commit('enableSelectable');
      this.$set(
        this.questions,
        this.questions.findIndex(q => q.id === question.id),
        question,
      );
      APIService(this.$serviceApiAuthenticated)
        .updateQuestion(question)
        .then((response) => {
          this.$set(
            this.questions,
            this.questions.findIndex(q => q.id === question.id),
            response,
          );
          if (response.pointAnnotations.length > 0) {
            this.$annomlstore.commit(
              'removePointAnnotations',
              question.pointAnnotations,
            );
            this.$annomlstore.commit(
              'addPointAnnotations',
              response.pointAnnotations,
            );
          }
          if (response.rectangleAnnotations.length > 0) {
            this.$annomlstore.commit(
              'removeRectangleAnnotations',
              question.rectangleAnnotations,
            );
            this.$annomlstore.commit(
              'addRectangleAnnotations',
              response.rectangleAnnotations,
            );
          }
          if (response.color !== question.color) {
            this.$annomlstore.commit('addUsedColor', question.color);
          }
        });
    },
    deleteQuestion(question) {
      this.$annomlstore.commit('clearCurrentAnnotations');
      this.$annomlstore.commit('removeCurrentPost');
      this.$annomlstore.commit('enableSelectable');
      if (question.color) {
        this.$annomlstore.commit('removeUsedColor', question.color);
      }
      this.questions = this.questions.filter(q => q.id !== question.id);
      if (question.author) {
        APIService(this.$serviceApiAuthenticated)
          .deleteQuestion(this.discussion.id, question)
          .then((response) => {
            console.log(response);
            this.$set(
              this.questions,
              this.questions.findIndex(q => q.id === question.id),
              response,
            );
          });
      }
    },
    editQuestion(question) {
      if (!this.$annomlstore.getters.hasCurrentPost) {
        this.$annomlstore.commit('setCurrentPost', question);
        if (question.pointAnnotations.length > 0) {
          this.$annomlstore.commit(
            'removePointAnnotations',
            question.pointAnnotations,
          );
          this.$annomlstore.commit(
            'setCurrentPointAnnotations',
            question.pointAnnotations,
          );
        }
        if (question.rectangleAnnotations.length > 0) {
          this.$annomlstore.commit(
            'removeRectangleAnnotations',
            question.rectangleAnnotations,
          );
          this.$annomlstore.commit(
            'setCurrentRectangleAnnotations',
            question.rectangleAnnotations,
          );
        }
      }
    },
    /**
     * Vote Handling
     */
    upVoteQuestion(question) {
      APIService(this.$serviceApiAuthenticated)
        .upVoteQuestion(question)
        .then((response) => {
          this.$set(
            this.questions,
            this.questions.findIndex(q => q.id === question.id),
            response,
          );
        });
    },
    downVoteQuestion(question) {
      APIService(this.$serviceApiAuthenticated)
        .downVoteQuestion(question)
        .then((response) => {
          this.$set(
            this.questions,
            this.questions.findIndex(q => q.id === question.id),
            response,
          );
        });
    },
  },
};
</script>

<style lang="scss"></style>
