<template>
  <div>
    <h1>Edit Word</h1>
    <word-form :word="word" @createOrUpdate="updateWord"></word-form>
  </div>
</template>

<script>
import { api } from '../helpers/helpers';
import WordForm from '../components/WordForm.vue';

export default {
  name: 'edit',
  components: {
    'word-form': WordForm
  },
  data() {
    return {
      word: {}
    };
  },
  async mounted() {
    this.word = await api.getWord(this.$route.params.id);
  },
  methods: {
    async updateWord(word) {
      await api.updateWord(this.$route.params.id,word);
      this.flash('Word updated successfully!', 'success');
      this.$router.push(`/words/${this.$route.params.id}`);
    }
  }
};
</script>
