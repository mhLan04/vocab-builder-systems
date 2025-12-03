<template>
  <form class="ui form" @submit.prevent="onSubmit" novalidate>
    <div v-if="formError" class="ui negative message">
      <div class="header">Please fix the errors below</div>
      <p>{{ formError }}</p>
    </div>

    <!-- English -->
    <div :class="['field', { error: errors.english }]">
      <label>English</label>
      <input
        v-model.trim="localWord.english"
        type="text"
        placeholder="Enter English word"
      />
      <div v-if="errors.english" class="ui pointing red basic label">
        {{ errors.english }}
      </div>
    </div>

    <!-- German -->
    <div :class="['field', { error: errors.german }]">
      <label>German</label>
      <input
        v-model.trim="localWord.german"
        type="text"
        placeholder="Enter German word"
      />
      <div v-if="errors.german" class="ui pointing red basic label">
        {{ errors.german }}
      </div>
    </div>

    <!-- French -->
    <div :class="['field', { error: errors.french }]">
      <label>French</label>
      <input
        v-model.trim="localWord.french"
        type="text"
        placeholder="Enter French word"
      />
      <div v-if="errors.french" class="ui pointing red basic label">
        {{ errors.french }}
      </div>
    </div>

    <button type="submit" class="ui primary button">
      Save
    </button>
  </form>
</template>

<script>
export default {
  name: 'word-form',
  props: {
    word: {
      type: Object,
      default: () => ({
        english: '',
        german: '',
        french: ''
      })
    }
  },
  data() {
    return {
      localWord: { ...this.word }, 
      errors: {},                  
      formError: ''               
    };
  },
  watch: {
    word: {
      handler(newWord) {
        this.localWord = { ...newWord };
      },
      deep: true
    }
  },
  methods: {
    isValidWord(value) {
      if (!value) return false;
      const pattern = /^[\p{L}\s'’-]+$/u;
      return pattern.test(value.trim());
    },

    validate() {
      const errors = {};

      // English
      if (!this.localWord.english || !this.localWord.english.trim()) {
        errors.english = 'English word cannot be blank';
      } else if (!this.isValidWord(this.localWord.english)) {
        errors.english =
          'English word can only contain letters, spaces, hyphens or apostrophes';
      }

      // German
      if (!this.localWord.german || !this.localWord.german.trim()) {
        errors.german = 'German word cannot be blank';
      } else if (!this.isValidWord(this.localWord.german)) {
        errors.german =
          'German word can only contain letters, spaces, hyphens or apostrophes';
      }

      // French
      if (!this.localWord.french || !this.localWord.french.trim()) {
        errors.french = 'French word cannot be blank';
      } else if (!this.isValidWord(this.localWord.french)) {
        errors.french =
          'French word can only contain letters, spaces, hyphens or apostrophes';
      }

      this.errors = errors;

      this.formError =
        Object.keys(errors).length > 0
          ? 'Please correct the highlighted fields before saving.'
          : '';

      return Object.keys(errors).length === 0;
    },


    onSubmit() {
      if (!this.validate()) {
        this.$nextTick(() => {
          if (this.$el && this.$el.scrollIntoView) {
            this.$el.scrollIntoView({ behavior: 'smooth', block: 'start' });
          }
        });
        return;
      }

      this.$emit('createOrUpdate', { ...this.localWord });
    }
  }
};
</script>
