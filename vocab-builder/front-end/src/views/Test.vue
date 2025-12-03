<template>
  <div>
    <h1 style="text-align: center">Check Vocabulary</h1>
    <vocab-test></vocab-test>
  </div>
</template>

<script>
import VocabTest from '../components/VocabTest.vue';

export default {
  name: 'test',
  components: {
    'vocab-test': VocabTest
  }
};
</script>
