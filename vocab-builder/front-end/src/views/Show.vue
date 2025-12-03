<template>
  <div>
    <h1 class="ui header">Show Word</h1>

    <div class="ui form">

      <!-- German -->
      <div class="field">
        <label class="ui label" style="width: 100px;">
          <i class="de flag"></i>
          German
        </label>
        <input
          type="text"
          :value="word.german"
          readonly
          class="ui input"
          style="width: 300px;"
        />
      </div>

      <!-- English -->
      <div class="field">
        <label class="ui label" style="width: 100px;">
          <i class="gb flag"></i>
          English
        </label>
        <input
          type="text"
          :value="word.english"
          readonly
          class="ui input"
          style="width: 300px;"
        />
      </div>
<!-- French -->
      <div class="field">
        <label class="ui label" style="width: 100px;">
        <i class="fr flag"></i>
         French
         </label>

        <input
          type="text"
          :value="word.french"
          readonly
          class="ui input"
          style="width: 300px;"
        />
      </div>

    </div>

    <router-link
      class="ui primary button"
      :to="{ name: 'edit', params: { id: word._id } }"
    >
      Edit word
    </router-link>
  </div>
</template>

<script>
import { api } from '../helpers/helpers';

export default {
  name: 'show',
  data() {
    return {
      word: {}
    };
  },
  async mounted() {
    this.word = await api.getWord(this.$route.params.id);
  }
};
</script>
 
