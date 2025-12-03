<template>
  <div>
    <h1>New Word</h1>
    <word-form @createOrUpdate="createOrUpdate"></word-form>
  </div>
</template>

<script>
import { api } from '../helpers/helpers';
import WordForm from '../components/WordForm.vue';

export default {
  name: 'new-word',
  components: {
    'word-form': WordForm
  },
  data() {
    return {
      existingWords: [],   
      isLoading: false     
    };
  },
  async mounted() {
    this.existingWords = await api.getWords();
  },
  methods: {
    async createOrUpdate(word) {
      const normalize = value => (value || '').trim().toLowerCase();

      const newEnglish = normalize(word.english);
      const newGerman  = normalize(word.german);
      const newFrench  = normalize(word.french);

      const duplicate = this.existingWords.find(w =>
        normalize(w.english) === newEnglish &&
        normalize(w.german)  === newGerman  &&
        normalize(w.french)  === newFrench
      );

      if (duplicate) {
        this.flash(
          'This vocabulary entry already exists in all three languages.',
          'error'
        );

        const goToEdit = window.confirm(
          'An identical word (English, German and French) already exists. Do you want to edit that word instead?'
        );

        if (goToEdit) {
          this.$router.push(`/words/${duplicate._id}/edit`);
        }
        return;
      }

      this.isLoading = true;
      try {
        const res = await api.createWord(word);
        this.flash('Word created', 'success');
        this.$router.push(`/words/${res._id}`);
      } finally {
        this.isLoading = false;
      }
    }
  }
};
</script>
