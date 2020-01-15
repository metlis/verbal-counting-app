<template>
  <v-container fluid>
    <v-row>
      <v-col
        cols="8"
        offset="2"
      >
        <v-card>
          <v-card-title>
            Verbal counting trainer
          </v-card-title>
          <v-card-text>
            <!-- Start button -->
            <div class="d-flex">
              <v-btn
                v-if="!started && !resultMessage"
                @click="startTask"
                class="my-5 mx-auto"
                color="success"
                x-large
                text
                dark
              >
                Start
              </v-btn>
            </div>
            <!-- Task content -->
            <div
              v-if="started"
              class="task"
            >
              <v-row>
                <v-progress-linear
                  v-if="taskOptions.timer.isSet"
                  :value="timerPercent"
                  :color="timerColor"
                />
              </v-row>
              <v-row>
                <!-- Left argument -->
                <v-col
                  cols="2"
                  offset="2"
                >
                  <div class="py-5">
                    {{ taskNumLeft }}
                  </div>
                </v-col>
                <!-- Task sign -->
                <v-col
                  cols="1"
                  class="py-8"
                >
                  <div>{{ taskSign }}</div>
                </v-col>
                <!-- Right argument -->
                <v-col cols="2">
                  <div class="py-5">
                    {{ taskNumRight }}
                  </div>
                </v-col>
                <!-- Equals sign -->
                <v-col
                  cols="1"
                  class="py-8"
                >
                  <div>=</div>
                </v-col>
                <!-- Result -->
                <v-col cols="2">
                  <v-text-field
                    v-model="taskAnswer"
                    @keydown.enter="submitAnswer"
                    :disabled="paused"
                  />
                  <span
                    v-if="taskOptions.showTasksCount"
                    class="counter"
                  >
                    <span class="counter-incorrect">{{getIncorrectAnswersNum()}}</span>
                    /
                    <span class="counter-correct">{{getCorrectAnswersNum()}}</span>
                    <span v-if="taskOptions.tasksLimit.isSet">
                    / {{getTasksLeftNum()}}
                    </span>
                  </span>
                </v-col>
              </v-row>
              <!-- Submit button -->
              <v-row>
                <v-col cols="10" offset="1">
                  <v-btn
                    @click="submitAnswer"
                    text
                  >
                    Submit
                  </v-btn>
                </v-col>
              </v-row>
              <!-- Task controls -->
              <div class="d-flex justify-end">
                <v-btn
                  v-if="taskOptions.timer.isSet"
                  @click="processBreakButtonClick"
                  class="ma-1"
                  text
                  small
                >
                  {{ breakButtonText }}
                </v-btn>
                <v-btn
                  @click="stopTask"
                  class="ma-1"
                  color="red lighten-2"
                  text
                  small
                >
                  Stop
                </v-btn>
              </div>
            </div>
            <!-- Result content -->
            <div
              v-if="resultMessage"
              class="task"
            >
              <v-row>
                <v-col cols="12">
                  {{resultMessage}}
                </v-col>
              </v-row>
              <v-row>
                <v-col cols="12">
                  <v-btn
                    @click="startTask"
                    small
                    text
                  >
                    Restart
                  </v-btn>
                </v-col>
              </v-row>
            </div>
          </v-card-text>
          <v-card-actions>
            <v-btn-toggle
              v-model="activeSection"
              mandatory
              dense
              borderless
            >
              <v-btn
                v-for="section in sections"
                :key="section"
                small
              >
                {{ section }}
              </v-btn>
            </v-btn-toggle>
            <v-spacer></v-spacer>
            <v-btn
              icon
              @click="showTaskOptions = !showTaskOptions"
            >
              <v-icon>{{ chevron }}</v-icon>
            </v-btn>
          </v-card-actions>
          <!-- Task options -->
          <v-expand-transition>
            <div v-show="showTaskOptions">
              <v-divider></v-divider>
              <v-card-text>
                <h4 class="options">
                  Options
                </h4>
                <br>
                <v-select
                  v-model="taskOptions.levels.selected"
                  :disabled="started"
                  :items="levelsNames"
                  label="Level"
                />
                <p class="sub-header">Flow</p>
                <v-row class="ma-0 pa-0">
                  <v-switch
                    v-model="taskOptions.timer.isSet"
                    :disabled="started"
                    class="ma-1"
                    label="Set the timer"
                  />
                  <v-text-field
                    v-if="taskOptions.timer.isSet"
                    v-model="taskOptions.timer.initValue"
                    :rules="taskOptions.timer.rules"
                    :disabled="started"
                    class="ma-1"
                    label="Time in seconds"
                    hide-details="auto"
                  />
                </v-row>
                <v-row class="ma-0 pa-0">
                  <v-switch
                    v-model="taskOptions.tasksLimit.isSet"
                    :disabled="started"
                    class="ma-1"
                    label="Limit the number of tasks"
                  />
                  <v-text-field
                    v-if="taskOptions.tasksLimit.isSet"
                    v-model="taskOptions.tasksLimit.value"
                    :rules="taskOptions.tasksLimit.rules"
                    :disabled="started"
                    class="ma-1"
                    label="Max number of tasks"
                    hide-details="auto"
                  />
                </v-row>
                <v-switch
                  v-model="taskOptions.showTasksCount"
                  :disabled="started"
                  class="ma-1"
                  label="Show the counter for tasks"
                />
                <v-switch
                  v-model="taskOptions.showOptions"
                  :disabled="started"
                  class="ma-1"
                  label="Show options"
                />
                <p class="sub-header">Results</p>
                <v-switch
                  v-model="taskOptions.showAnswers"
                  :disabled="started"
                  class="ma-1"
                  label="Show correct answers"
                />
                <v-switch
                  v-model="taskOptions.showHistory"
                  :disabled="started"
                  class="ma-1"
                  label="Show history"
                />
              </v-card-text>
            </div>
          </v-expand-transition>
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
export default {
  name: 'TrainerContent',

  data() {
    return {
      showTaskOptions: true,
      sections: ['Multiplication', 'Division', 'Addition', 'Subtraction', 'Combined'],
      activeSection: 0,
      taskOptions: {
        tasksLimit: {
          isSet: false,
          value: 10,
          rules: [
            value => Number(value) >= 1 || 'Min number 1',
          ],
        },
        timer: {
          isSet: false,
          initValue: 10,
          currentValue: 0,
          timerInstance: '',
          rules: [
            value => Number(value) >= 10 || 'Min time is 10 seconds',
          ],
          colors: {
            blue: 'light-blue',
            red: 'red lighten-3',
            green: 'light-green',
          },
        },
        levels: {
          selected: 'Easy',
          items: {
            table: {
              name: 'Table',
              maxResult: 100,
              minResult: 1,
              minArg: 1,
              maxArg: 10,
            },
            easy: {
              name: 'Easy',
              multiplication: {
                maxResult: 10000,
                minResult: 101,
                minArg: 2,
                maxArg: 100,
              },
            },
            intermediate: {
              name: 'Intermediate',
              multiplication: {
                maxResult: 1000000,
                minResult: 10001,
                minArg: 11,
                maxArg: 90909,
              },
            },
            hard: {
              name: 'Hard',
              multiplication: {
                maxResult: 1000000000,
                minResult: 1000001,
                minArg: 101,
                maxArg: 9900990,
              },
            },
            extraHard: {
              name: 'Extra-hard',
              multiplication: {
                maxResult: 1000000000000,
                minResult: 1000000001,
                minArg: 1001,
                maxArg: 999000999,
              },
            },
          },
        },
        showAnswers: true,
        showTasksCount: true,
        showHistory: true,
        showOptions: true,
      },
      started: false,
      paused: false,
      taskAnswer: '',
      chevronIcons: {
        up: 'mdi-chevron-up',
        down: 'mdi-chevron-down',
      },
      breakButtonTexts: {
        pause: 'Pause',
        continue: 'Continue',
      },
      tableLevelName: 'Table',
      subTasks: [],
      activeSubTask: '',
      sessions: {
        id: '',
        results: {},
      },
      resultMessage: '',
      noSolvedTasksMessage: 'You have not solved any tasks',
    };
  },

  computed: {
    chevron() {
      return this.showTaskOptions ? this.chevronIcons.up : this.chevronIcons.down;
    },
    breakButtonText() {
      return this.paused ? this.breakButtonTexts.continue : this.breakButtonTexts.pause;
    },
    taskNumLeft() {
      return this.addCommas(this.activeSubTask[0]);
    },
    taskNumRight() {
      return this.addCommas(this.activeSubTask[1]);
    },
    taskNumResult() {
      const left = this.taskNumLeft;
      const right = this.taskNumRight;
      switch (this.activeSection) {
        case 0:
          return left * right;
        case 1:
          return left / right;
        case 2:
          return left + right;
        case 3:
          return left - right;
        default:
          return '';
      }
    },
    taskSign() {
      switch (this.activeSection) {
        case 0:
          return '*';
        case 1:
          return '/';
        case 2:
          return '+';
        case 3:
          return '-';
        default:
          return '';
      }
    },
    timerPercent() {
      const { currentValue, initValue } = this.taskOptions.timer;
      return currentValue / initValue * 100;
    },
    timerColor() {
      const { timer } = this.taskOptions;
      const timePassed = timer.currentValue / timer.initValue;
      if (timePassed === 1) return timer.colors.green;
      if (timePassed >= 0.8) return timer.colors.red;
      return timer.colors.blue;
    },
    levelsNames() {
      const names = [];
      Object.values(this.taskOptions.levels.items).forEach((item) => {
        // generate table level name for multiplication and division sections
        if (
          item.name === this.tableLevelName
          && (this.activeSection <= 1)
        ) {
          const sectionName = this.sections[this.activeSection];
          names.push(`${sectionName} ${item.name}`);
        }
        // generate other levels
        if (item.name !== this.tableLevelName) names.push(item.name);
      });
      return names;
    },
  },

  methods: {
    startTask() {
      this.initSession();
      this.generateSubTasks();
      this.goToNextSubTask();
      this.hideTaskOptions();
      this.startTimer();
      this.clearResultMessage();
      this.started = true;
    },
    stopTask() {
      this.clearTasks();
      this.clearTaskAnswer();
      this.createResultMessage();
      this.stopTimer();
      this.started = false;
      this.paused = false;
    },
    getCorrectAnswersNum() {
      const results = this.sessions.results[this.sessions.id];
      return results.correct.length;
    },
    getIncorrectAnswersNum() {
      const results = this.sessions.results[this.sessions.id];
      return results.incorrect.length;
    },
    getTasksLeftNum() {
      return this.taskOptions.tasksLimit.value
        - this.getIncorrectAnswersNum() - this.getCorrectAnswersNum();
    },
    taskFinished() {
      return this.taskOptions.tasksLimit.isSet && this.getTasksLeftNum() === 0;
    },
    createResultMessage() {
      const correctAnswers = this.getCorrectAnswersNum();
      const subTasksNum = correctAnswers + this.getIncorrectAnswersNum();
      if (subTasksNum === 0) {
        this.resultMessage = this.noSolvedTasksMessage;
      } else {
        const answersCorrectPercent = Math.round((correctAnswers / subTasksNum) * 100);
        this.resultMessage = `Total tasks solved: ${subTasksNum}.
        Correct answers: ${correctAnswers} (${answersCorrectPercent}%)`;
      }
    },
    hideTaskOptions() {
      this.showTaskOptions = false;
    },
    goToNextSubTask() {
      const nextTask = this.subTasks.splice(0, 1)[0];
      this.activeSubTask = nextTask;
    },
    processBreakButtonClick() {
      this.paused = !this.paused;
      if (this.paused) {
        clearInterval(this.taskOptions.timer.timerInstance);
      } else this.startTimer();
    },
    clearTaskAnswer() {
      this.taskAnswer = '';
    },
    clearTasks() {
      this.activeSubTask = '';
      this.subTasks = [];
    },
    clearResultMessage() {
      this.resultMessage = '';
    },
    initSession() {
      if (!this.sessions.id) {
        this.sessions.id = 1;
      } else this.sessions.id += 1;
      this.sessions.results[this.sessions.id] = {
        correct: [],
        incorrect: [],
      };
    },
    submitAnswer() {
      if (!this.answerValueCorrect(this.taskAnswer)) return;
      // store user's answers
      const results = this.sessions.results[this.sessions.id];
      if (+this.taskAnswer === this.taskNumResult) {
        results.correct.push(this.activeSubTask);
      } else {
        this.activeSubTask.push(+this.taskAnswer);
        results.incorrect.push(this.activeSubTask);
      }
      // go to the next task
      if (!this.taskFinished()) {
        this.clearTaskAnswer();
        if (this.subTasks.length === 0) this.generateSubTasks();
        this.goToNextSubTask();
      } else {
        this.stopTask();
      }
    },
    answerValueCorrect(val) {
      const num = Number(val);
      return !(!num || num !== parseInt(val, 10));
    },
    startTimer() {
      if (this.taskOptions.timer.isSet) {
        const interval = setInterval(() => {
          const { timer } = this.taskOptions;
          if (timer.currentValue !== timer.initValue) {
            timer.currentValue += 1;
          } else clearInterval(interval);
        }, 1000);
        this.taskOptions.timer.timerInstance = interval;
      }
    },
    stopTimer() {
      if (this.taskOptions.timer.isSet) {
        this.taskOptions.timer.currentValue = 0;
        clearInterval(this.taskOptions.timer.timerInstance);
      }
    },
    generateSubTasks() {
      // generate sub-tasks for multiplication/division table
      if (
        (this.activeSection === 0 || this.activeSection === 1)
        && this.taskOptions.levels.selected.indexOf(this.tableLevelName) > -1
      ) {
        this.subTasks = this.generateTableSubTasks();
      // generate sub-tasks for multiplication section
      } else if (this.activeSection === 0) {
        this.subTasks.push(this.generateMultiplicationSubTask());
      }
    },
    // generates all possible sub-tasks for table level
    generateTableSubTasks() {
      const {
        minResult,
        maxResult,
        minArg,
        maxArg,
      } = this.taskOptions.levels.items.table;
      const tasks = [];
      let leftArg = minArg;
      let rightArg = minArg;
      while (leftArg <= maxResult && leftArg * rightArg <= maxResult) {
        if (leftArg * rightArg >= minResult && leftArg <= rightArg && rightArg <= maxArg) {
          tasks.push([leftArg, rightArg]);
        }
        rightArg += 1;
        if (rightArg > maxResult || leftArg * rightArg > maxResult) {
          rightArg = minArg;
          leftArg += 1;
        }
      }
      return this.shuffleArray(tasks);
    },
    generateMultiplicationSubTask() {
      const level = Object.values(this.taskOptions.levels.items)
        .find(item => item.name === this.taskOptions.levels.selected);
      const {
        maxArg,
        minArg,
        maxResult,
        minResult,
      } = level.multiplication;
      let leftArg = this.generateRandomNumber(minArg, maxArg);
      let rightArg = this.generateRandomNumber(minArg, maxArg);
      while (
        leftArg * rightArg < minResult
        || leftArg * rightArg > maxResult
      ) {
        leftArg = this.generateRandomNumber(minArg, maxArg);
        rightArg = this.generateRandomNumber(minArg, maxArg);
      }
      return [leftArg, rightArg];
    },
    generateRandomNumber(min, max) {
      return Math.floor(Math.random() * (max - min) + min);
    },
    // https://stackoverflow.com/a/12646864
    shuffleArray(arr) {
      const array = arr;
      for (let i = array.length - 1; i > 0; i -= 1) {
        const j = Math.floor(Math.random() * (i + 1));
        [array[i], array[j]] = [array[j], array[i]];
      }
      return array;
    },
    // https://stackoverflow.com/questions/2901102/how-to-print-a-number-with-commas-as-thousands-separators-in-javascript
    addCommas(num) {
      try {
        return num.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ',');
      } catch (e) {
        return '';
      }
    },
  },

  watch: {
    timerPercent(val) {
      if (val === 100) this.stopTask();
    },
  },
};
</script>

<style lang="stylus" scoped>
  .options
    text-align center
  .task
    font-size 25px
    text-align center
  .counter
    font-size 12px
  .counter-correct
    color #8BC34A
  .counter-incorrect
    color #EF9A9A
  .sub-header
    font-size 12px
    opacity 0.7
    margin-bottom 10px
</style>
