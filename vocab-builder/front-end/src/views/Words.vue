<template>
  <div>
    <h1>Words</h1>

    <div class="search-section">
      <input
        v-model="searchQuery"
        type="text"
        placeholder="Search by English, German, or French..."
        class="ui input search-input"
      />
    </div>

    <div class="controls-section">
      <!-- removed: Search-in control (searches all columns by default) -->

      <div class="control-group">
        <label>Display column:</label>
        <select v-model="displayLang" class="ui dropdown">
          <option value="all">Show all</option>
          <option value="english">English only</option>
          <option value="german">German only</option>
          <option value="french">French only</option>
          <option value="english-german">English - German</option>
          <option value="english-french">English - French</option>
          <option value="german-french">German - French</option>
        </select>
      </div>

      <div class="control-group">
        <label>Sort by:</label>
        <select v-model="sortBy" class="ui dropdown">
          <option value="none">No Sort</option>
          <option v-for="opt in availableSortOptions" :key="opt.value" :value="opt.value">{{ opt.label }}</option>
        </select>
      </div>
    </div>

    <table id="words" class="ui celled compact table">
      <thead>
        <tr>
          <th v-if="showColumn('english')">English</th>
          <th v-if="showColumn('german')">German</th>
          <th v-if="showColumn('french')">French</th>
          <th colspan="3"></th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(word, i) in filteredWords" :key="i">
          <td v-if="showColumn('english')">{{ word.english }}</td>
          <td v-if="showColumn('german')">{{ word.german }}</td>
          <td v-if="showColumn('french')">{{ word.french }}</td>

          <td width="75" class="center aligned">
            <router-link :to="{ name: 'show', params: { id: word._id } }">
              Show
            </router-link>
          </td>

          <td width="75" class="center aligned">
            <router-link :to="{ name: 'edit', params: { id: word._id } }">
              Edit
            </router-link>
          </td>

          <td width="75" class="center aligned">
            <a href="#" @click.prevent="onDestroy(word._id)">Delete</a>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
import { api } from '../helpers/helpers';

export default {
  name: 'words',
  data() {
    return {
      words: [],
      searchQuery: '',
      // display control
      displayLang: 'all', // which column to display: all|english|german|french (also supports pairs)
      sortBy: 'none'
    };
  },
  computed: {
    filteredWords() {
      const q = this.searchQuery.trim().toLowerCase();
      
      // Step 1: Apply search filter (search all columns)
      let result = this.words;
      if (q) {
        result = result.filter(word =>
          (word.english || '').toLowerCase().includes(q) ||
          (word.german || '').toLowerCase().includes(q) ||
          (word.french || '').toLowerCase().includes(q)
        );
      }

      // Step 2: If displayLang is set to a single column, ensure we only include rows that have that column
      if (this.displayLang === 'english') {
        result = result.filter(word => word.english);
      } else if (this.displayLang === 'german') {
        result = result.filter(word => word.german);
      } else if (this.displayLang === 'french') {
        result = result.filter(word => word.french);
      }

      // Step 3: Apply sort
      if (this.sortBy !== 'none') {
        result = [...result].sort((a, b) => {
          let aVal, bVal;
          
          if (this.sortBy === 'english-asc') {
            aVal = (a.english || '').toLowerCase();
            bVal = (b.english || '').toLowerCase();
            return aVal.localeCompare(bVal);
          } else if (this.sortBy === 'english-desc') {
            aVal = (a.english || '').toLowerCase();
            bVal = (b.english || '').toLowerCase();
            return bVal.localeCompare(aVal);
          } else if (this.sortBy === 'german-asc') {
            aVal = (a.german || '').toLowerCase();
            bVal = (b.german || '').toLowerCase();
            return aVal.localeCompare(bVal);
          } else if (this.sortBy === 'german-desc') {
            aVal = (a.german || '').toLowerCase();
            bVal = (b.german || '').toLowerCase();
            return bVal.localeCompare(aVal);
          } else if (this.sortBy === 'french-asc') {
            aVal = (a.french || '').toLowerCase();
            bVal = (b.french || '').toLowerCase();
            return aVal.localeCompare(bVal);
          } else if (this.sortBy === 'french-desc') {
            aVal = (a.french || '').toLowerCase();
            bVal = (b.french || '').toLowerCase();
            return bVal.localeCompare(aVal);
          }
        });
      }

      return result;
    }
    ,
    availableSortOptions() {
      // show sort options only for columns that are visible according to displayLang
      const opts = [];
      const showEnglish = this.displayLang === 'all' || this.displayLang.includes('english');
      const showGerman = this.displayLang === 'all' || this.displayLang.includes('german');
      const showFrench = this.displayLang === 'all' || this.displayLang.includes('french');

      if (showEnglish) {
        opts.push({ value: 'english-asc', label: 'English A-Z' });
        opts.push({ value: 'english-desc', label: 'English Z-A' });
      }
      if (showGerman) {
        opts.push({ value: 'german-asc', label: 'German A-Z' });
        opts.push({ value: 'german-desc', label: 'German Z-A' });
      }
      if (showFrench) {
        opts.push({ value: 'french-asc', label: 'French A-Z' });
        opts.push({ value: 'french-desc', label: 'French Z-A' });
      }
      return opts;
    }
  },
  async mounted() {
    this.words = await api.getWords();
  },
  watch: {
    displayLang(newVal) {
      // if current sortBy is not available for the new display selection, reset to 'none'
      const allowed = this.availableSortOptions.map(o => o.value);
      if (this.sortBy !== 'none' && !allowed.includes(this.sortBy)) {
        this.sortBy = 'none';
      }
    }
  },
  methods: {
    async onDestroy(id) {
      const sure = window.confirm('Are you sure?');
      if (!sure) return;

      await api.deleteWord(id);
      this.flash('Word deleted successfully!', 'success');
      this.words = this.words.filter(word => word._id !== id);
    }
    ,
    showColumn(col) {
      // col is one of: 'english' | 'german' | 'french'
      if (this.displayLang === 'all') return true;
      if (this.displayLang === col) return true;
      if (this.displayLang === 'english-german' && (col === 'english' || col === 'german')) return true;
      if (this.displayLang === 'english-french' && (col === 'english' || col === 'french')) return true;
      if (this.displayLang === 'german-french' && (col === 'german' || col === 'french')) return true;
      return false;
    }
  }
};
</script>

<style scoped>
.search-section { margin-bottom: 20px; text-align: center; }
.search-input { max-width: 500px; padding: 10px 12px; }

.controls-section { display: flex; gap: 30px; margin-bottom: 20px; justify-content: center; flex-wrap: wrap; }
.control-group { display: flex; align-items: center; gap: 10px; }
.control-group label { font-weight: bold; white-space: nowrap; }
.control-group select { padding: 8px 12px; border: 1px solid #ddd; border-radius: 4px; }
</style>

