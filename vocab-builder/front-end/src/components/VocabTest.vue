<template>
  <div class="test-container">
    <div v-if="!isTestStarted && !isTestFinished" class="start-section">
      <p> have {{ totalWords }} Words in the system</p>
      
      <div class="language-selector">
        <label>Choose language to translate from English to:</label>
        <select v-model="selectedLanguage" class="ui dropdown">
          <option value="german">German (Deutsch)</option>
          <option value="french">French (Français)</option>
        </select>
      </div>

      <div class="question-count-selector" style="margin-top: 1rem;">
        <label>Number of questions:</label>
        <select v-model.number="selectedQuestionCount" class="ui dropdown">
          <option v-if="totalWords >= 5" :value="5">5 questions</option>
          <option v-if="totalWords >= 10" :value="10">10 questions</option>
          <option v-if="totalWords >= 20" :value="20">20 questions</option>
          <option :value="totalWords">All ({{ totalWords }}) questions</option>
        </select>
      </div>

      <button type="button" class="ui button primary" @click="startTest">
        Start Examination
      </button>
    </div>
    

    <div v-else-if="!isTestFinished" class="question-section">
      <div class="progress-info">
        Question {{ currentQuestionIndex + 1 }} / {{ totalWords }}
      </div>
      <div class="ui progress">
        <div class="bar" :style="{ width: progressPercentage + '%' }"></div>
      </div>

      <div class="question-card">
        <h2>
          Translate to {{ selectedLanguage === 'german' ? 'German' : 'French' }}:
        </h2>
        <div class="english-word">{{ currentQuestion.english }}</div>

        <div class="choices">
          <button
            v-for="(choice, idx) in choices"
            :key="idx"
            class="ui button fluid choice-button"
            :class="{
              selected: selectedChoiceIndex === idx,
              correct: showFeedback && choice === getCorrectAnswer(),
              wrong:
                showFeedback &&
                selectedChoiceIndex === idx &&
                choice !== getCorrectAnswer()
            }"
            @click="selectChoice(idx)"
            :disabled="showFeedback"
          >
            {{ choice }}
          </button>
        </div>

        <div class="action-row" style="margin-top:12px">
          <button
            type="button"
            class="ui button primary"
            @click="confirmAnswer"
            :disabled="selectedChoiceIndex === null || showFeedback"
          >
            Answer
          </button>
        </div>
      </div>

      <div v-if="showFeedback" :class="['feedback', feedbackClass]">
        <p v-if="isCorrect">
          ✓ Correct! The answer is: {{ getCorrectAnswer() }}
        </p>
        <p v-else>
          ✗ Wrong! The correct answer is: {{ getCorrectAnswer() }}
        </p>
        <button type="button" class="ui button" @click="nextQuestion">
          Next Question
        </button>
      </div>
    </div>

    <!-- RESULT MODAL -->
    <div v-if="showModal" class="modal-overlay">
      <div class="modal">
        <h2>Examination Results</h2>

        <div class="score">
          <div class="score-number">{{ correctCount }} / {{ totalWords }}</div>
          <div class="score-percent">{{ scorePercentage }}%</div>
        </div>

    <div class="results-list">
  <h3>Summary</h3>
  <ul>
    <li v-for="(result, index) in testResults" :key="index">
      <strong>{{ result.english }}</strong>
      - {{ result.userAnswer || '(no answer)' }}

      <span
        class="status-icon"
        :class="result.status"
        style="margin-left: 8px;"
      >
        {{ result.status === 'correct' ? '✓' : '✗' }}
      </span>
    </li>
  </ul>
</div>



        <!-- ✅ Review incorrect answers -->
        <div
          v-if="incorrectAnswers.length > 0"
          class="review-section"
          style="margin-top: 1.5rem;"
        >
          <h3>Review incorrect answers</h3>
          <table class="ui celled table">
            <thead>
              <tr>
                <th>#</th>
                <th>Question (English)</th>
                <th>Correct answer ({{ selectedLanguageLabel }})</th>
                <th>Your answer</th>
              </tr>
            </thead>
            <tbody>
              <tr
                v-for="(item, idx) in incorrectAnswers"
                :key="'inc-' + idx"
              >
                <td>{{ item.questionNumber }}</td>
                <td>{{ item.english }}</td>
                <td>{{ item.correctAnswer }}</td>
                <td>
                  <span v-if="item.userAnswer">
                    {{ item.userAnswer }}
                  </span>
                  <span v-else class="muted">
                    (no answer)
                  </span>
                </td>
              </tr>
            </tbody>
          </table>
        </div>

        <div
          v-else
          class="ui positive message"
          style="margin-top: 1.5rem;"
        >
          <div class="header">Perfect score!</div>
          <p>You did not make any mistakes in this test.</p>
        </div>

        <div class="modal-actions">
          <button type="button" class="ui button" @click="goToWords">
            Close
          </button>
          <button
            type="button"
            class="ui button primary"
            @click="tryAgain"
          >
            Try Again
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { api } from '../helpers/helpers';

function shuffle(array) {
  for (let i = array.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [array[i], array[j]] = [array[j], array[i]];
  }
  return array;
}

export default {
  name: 'vocab-test',
  data() {
    return {
      words: [],
      currentQuestionIndex: 0,
      isTestStarted: false,
      isTestFinished: false,
      choices: [],
      selectedChoiceIndex: null,
      showFeedback: false,
      isCorrect: false,
      correctCount: 0,
      testResults: [],
      selectedLanguage: 'german',
      selectedQuestionCount: 5,
      showModal: false,
      incorrectAnswers: [] // lưu câu sai
    };
  },
  computed: {
    currentQuestion() {
      return this.words[this.currentQuestionIndex] || {};
    },
    totalWords() {
      return this.words.length;
    },
    progressPercentage() {
      if (this.totalWords === 0) return 0;
      return Math.round((this.currentQuestionIndex / this.totalWords) * 100);
    },
    scorePercentage() {
      if (this.totalWords === 0) return 0;
      return Math.round((this.correctCount / this.totalWords) * 100);
    },
    feedbackClass() {
      return this.isCorrect ? 'correct' : 'wrong';
    },
    // dùng trong bảng review
    selectedLanguageLabel() {
      if (this.selectedLanguage === 'german') return 'German';
      if (this.selectedLanguage === 'french') return 'French';
      return this.selectedLanguage;
    }
  },
  async mounted() {
    await this.loadWords();
  },
  methods: {
    async loadWords() {
      try {
        this.words = await api.getWords();
      } catch (err) {
        console.error('Lỗi khi tải từ vựng:', err);
        this.words = [];
      }
    },
    startTest() {
      if (this.words.length === 0) {
        alert('Không có từ vựng để test');
        return;
      }
      const shuffled = shuffle([...this.words]);
      const limit = Math.min(this.selectedQuestionCount, shuffled.length);
      this.words = shuffled.slice(0, limit);

      this.currentQuestionIndex = 0;
      this.correctCount = 0;
      this.testResults = [];
      this.incorrectAnswers = []; // reset câu sai
      this.isTestStarted = true;
      this.isTestFinished = false;
      this.showModal = false;

      this.prepareChoices();
    },
    // Prepare multiple-choice options: 1 correct + up to 3 distractors
    prepareChoices() {
      const current = this.currentQuestion;
      if (!current) {
        this.choices = [];
        return;
      }

      const correctAnswer = this.getCorrectAnswer();

      const pool = this.words
        .map(w =>
          this.selectedLanguage === 'german' ? w.german : w.french
        )
        .filter(g => g && g !== correctAnswer);

      shuffle(pool);
      const distractors = pool.slice(0, Math.min(3, pool.length));

      const opts = shuffle([correctAnswer, ...distractors]);
      this.choices = opts;
      this.selectedChoiceIndex = null;
      this.showFeedback = false;
    },
    selectChoice(idx) {
      if (this.showFeedback) return;
      this.selectedChoiceIndex = idx;
    },
    confirmAnswer() {
      if (this.selectedChoiceIndex === null) return;

      const selected = this.choices[this.selectedChoiceIndex];
      const correctAnswer = this.getCorrectAnswer();

      this.isCorrect = selected === correctAnswer;

      if (this.isCorrect) {
        this.correctCount++;
      } else {
        // 👉 Lưu câu sai để review
        this.incorrectAnswers.push({
          questionNumber: this.currentQuestionIndex + 1,
          english: this.currentQuestion.english,
          correctAnswer,
          userAnswer: selected
        });
      }

      this.testResults.push({
        english: this.currentQuestion.english,
        german: this.currentQuestion.german,
        french: this.currentQuestion.french,
        userAnswer: selected,
        status: this.isCorrect ? 'correct' : 'wrong'
      });

      this.showFeedback = true;
    },
    nextQuestion() {
      this.currentQuestionIndex++;
      if (this.currentQuestionIndex >= this.totalWords) {
        this.isTestFinished = true;
        this.isTestStarted = false;
        this.showModal = true;
        this.saveTestResult();
      } else {
        this.prepareChoices();
      }
    },
    resetTest() {
      this.isTestStarted = false;
      this.isTestFinished = false;
      this.currentQuestionIndex = 0;
      this.correctCount = 0;
      this.testResults = [];
      this.choices = [];
      this.selectedChoiceIndex = null;
      this.showFeedback = false;
      this.showModal = false;
      this.incorrectAnswers = [];
    },
    closeModal() {
      this.showModal = false;
    },
    goToWords() {
      this.showModal = false;
      this.$router.push({ name: 'words' });
    },
    tryAgain() {
      this.showModal = false;
      this.resetTest();
      this.startTest();
    },
    saveTestResult() {
      try {
        const historyKey = 'vocabTestHistory';
        const stored = localStorage.getItem(historyKey);
        const history = stored ? JSON.parse(stored) : [];

        const entry = {
          date: new Date().toISOString(),
          language: this.selectedLanguage,
          correct: this.correctCount,
          total: this.totalWords,
          percent: this.scorePercentage,
          details: this.testResults
        };

        history.unshift(entry);
        if (history.length > 50) history.length = 50;
        localStorage.setItem(historyKey, JSON.stringify(history));
      } catch (e) {
        console.warn('Could not save test history', e);
      }
    },
    getCorrectAnswer() {
      if (!this.currentQuestion) return '';
      return this.selectedLanguage === 'german'
        ? this.currentQuestion.german
        : this.currentQuestion.french;
    }
  }
};
</script>
