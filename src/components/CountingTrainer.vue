<template>
  <v-container fluid>
    <v-row>
      <v-col
        cols="12"
        md="8"
        offset-md="2"
      >
        <v-card>
          <v-card-title>
            {{appTitle}}
          </v-card-title>
          <v-card-text>
            <!-- Start button -->
            <div class="d-flex">
              <v-btn
                v-if="startButtonIsVisible"
                @click="startTask"
                class="my-5 mx-auto"
                color="success"
                x-large
                text
                dark
              >
                {{controlButtons.start}}
              </v-btn>
            </div>
            <!-- Task content -->
            <div
              v-if="started"
              class="task"
            >
              <!-- Timer -->
              <v-row>
                <v-progress-linear
                  v-if="taskOptions.timer.isSet"
                  :value="timerPercent"
                  :color="timerColor"
                  class="mb-5 mb-md-0"
                />
              </v-row>
              <!-- Sub-task -->
              <v-row v-if="!visibleCorrectAnswer">
                <!-- Left argument -->
                <v-col
                  cols="12"
                  md="2"
                  offset-md="2"
                  class="px-0"
                >
                  <div class="py-md-5">
                    {{ subTaskFirstArgument }}
                  </div>
                </v-col>
                <!-- Task sign -->
                <v-col
                  cols="12"
                  md="2"
                  class="py-md-8 px-md-0"
                >
                  <div>
                    {{ subTaskSign }}
                  </div>
                </v-col>
                <!-- Right argument -->
                <v-col
                  cols="12"
                  md="2"
                  class="px-md-0"
                >
                  <div class="py-md-5">
                    {{ subTaskSecondArgument }}
                  </div>
                </v-col>
                <!-- Equals sign -->
                <v-col
                  cols="12"
                  md="1"
                  class="py-md-8 px-md-0"
                >
                  <div>
                    =
                  </div>
                </v-col>
                <!-- Result -->
                <v-col
                  cols="12"
                  md="2"
                  class="px-md-0"
                >
                  <!-- Answer input -->
                  <v-text-field
                    v-model="subTaskAnswer"
                    @keydown.enter="submitAnswer"
                    :disabled="paused"
                  />
                  <!-- Task counter -->
                  <span
                    v-if="taskOptions.showTasksCount.value"
                    class="counter"
                  >
                    <span class="counter-incorrect">
                      {{getIncorrectAnswersQuantity()}}
                    </span>
                    /
                    <span class="counter-correct">
                      {{getCorrectAnswersQuantity()}}
                    </span>
                    <span v-if="taskOptions.tasksLimit.isSet">
                    / {{getTasksLeftQuantity()}}
                    </span>
                  </span>
                </v-col>
              </v-row>
              <!-- Correct answer -->
              <v-row v-if="visibleCorrectAnswer">
                <v-col
                  cols="10"
                  offset="1"
                  class="py-md-8"
                >
                  <span class="correct-answer">
                    {{correctAnswerPrettified}}
                  </span>
                </v-col>
                <v-col
                  cols="10"
                  offset="1"
                >
                  <v-btn
                    @click="visibleCorrectAnswer = ''"
                    text
                  >
                    {{controlButtons.continue}}
                  </v-btn>
                </v-col>
              </v-row>
              <!-- Hints -->
              <v-row v-if="hintsAreVisible">
                <v-col
                  cols="10"
                  offset="1"
                  class="py-md-8"
                >
                  <v-chip
                    v-for="hint in answerHints"
                    @click="submitAnswerFromHints(hint)"
                    :key="hint"
                    :disabled="paused"
                    class="ma-1 option"
                  >
                    {{hint}}
                  </v-chip>
                </v-col>
              </v-row>
              <!-- Submit button -->
              <v-row v-if="!visibleCorrectAnswer">
                <v-col
                  cols="10"
                  offset="1"
                >
                  <v-btn
                    @click="submitAnswer"
                    :disabled="paused"
                    text
                  >
                    {{controlButtons.submit}}
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
                  {{controlButtons.stop}}
                </v-btn>
              </div>
            </div>
            <!-- Result content -->
            <div
              v-if="taskResultMessage"
              class="task"
            >
              <v-row>
                <v-col cols="12">
                  {{taskResultMessage}}
                </v-col>
              </v-row>
              <v-row>
                <v-col cols="12">
                  <v-btn
                    @click="startTask"
                    small
                    text
                  >
                    {{controlButtons.restart}}
                  </v-btn>
                </v-col>
              </v-row>
            </div>
          </v-card-text>
          <v-card-actions>
            <!-- Sections -->
            <v-btn-toggle
              v-model="activeSection"
              mandatory
              dense
              borderless
            >
              <v-btn
                v-for="(sign, section) in sections"
                :key="section"
                :disabled="started"
                color="transparent"
                small
              >
                <span class="d-none d-sm-none d-md-inline">
                  {{ section }}
                </span>
                <span class="mobile-section-button d-md-none">
                  {{sign}}
                </span>
              </v-btn>
            </v-btn-toggle>
            <v-spacer />
            <v-btn
              icon
              @click="showTaskOptions = !showTaskOptions"
            >
              <v-icon>
                {{ chevron }}
              </v-icon>
            </v-btn>
          </v-card-actions>
          <!-- Task options -->
          <v-expand-transition>
            <div v-show="showTaskOptions">
              <v-divider />
              <v-card-text>
                <h4 class="options">
                  {{taskOptionsHeader}}
                </h4>
                <br>
                <v-select
                  v-model="taskOptions.levels.selected"
                  :label="taskOptions.levels.label"
                  :disabled="started"
                  :items="levelsNames"
                />
                <v-row class="ma-0 pa-0">
                  <v-switch
                    v-model="taskOptions.timer.isSet"
                    :label="taskOptions.timer.switchLabel"
                    :disabled="started"
                    class="ma-1"
                  />
                  <v-text-field
                    v-if="taskOptions.timer.isSet"
                    v-model="taskOptions.timer.initValue"
                    :rules="taskOptions.timer.rules"
                    :label="taskOptions.timer.fieldLabel"
                    :disabled="started"
                    class="ma-1"
                    hide-details="auto"
                  />
                </v-row>
                <v-row class="ma-0 pa-0">
                  <v-switch
                    v-model="taskOptions.tasksLimit.isSet"
                    :label="taskOptions.tasksLimit.switchLabel"
                    :disabled="started"
                    class="ma-1"
                  />
                  <v-text-field
                    v-if="taskOptions.tasksLimit.isSet"
                    v-model="taskOptions.tasksLimit.value"
                    :rules="taskOptions.tasksLimit.rules"
                    :label="taskOptions.tasksLimit.fieldLabel"
                    :disabled="started"
                    class="ma-1"
                    hide-details="auto"
                  />
                </v-row>
                <v-switch
                  v-model="taskOptions.showTasksCount.value"
                  :label="taskOptions.showTasksCount.label"
                  :disabled="started"
                  class="ma-1"
                />
                <v-switch
                  v-model="taskOptions.showHints.value"
                  :label="taskOptions.showHints.label"
                  :disabled="started"
                  class="ma-1"
                />
                <v-switch
                  v-model="taskOptions.showCorrectAnswers.value"
                  :label="taskOptions.showCorrectAnswers.label"
                  :disabled="started"
                  class="ma-1"
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
  name: 'CountingTrainer',

  data() {
    return {
      started: false,
      paused: false,
      showTaskOptions: true,
      taskOptions: {
        tasksLimit: {
          switchLabel: 'Установить максимальное количество заданий',
          fieldLabel: 'Максимальное количество заданий',
          isSet: false,
          value: 10,
          rules: [
            value => Number(value) >= 1 || 'Минимальное количество 1',
          ],
        },
        timer: {
          switchLabel: 'Установить ограничение по времени',
          fieldLabel: 'Время в секундах',
          isSet: false,
          initValue: 10,
          currentValue: 0,
          timerInstance: '',
          rules: [
            value => Number(value) >= 10 || 'Минимальное время 10 секунд',
          ],
          colors: {
            blue: 'light-blue',
            red: 'red lighten-3',
            green: 'light-green',
          },
        },
        levels: {
          label: 'Уровень',
          selected: 'Легкий',
          default: 'Легкий',
          items: {
            table: {
              name: 'Таблица',
              multiplicationOrDivision: {
                maxResult: 100,
                minResult: 1,
                minArg: 1,
                maxArg: 10,
              },
              additionOrSubtraction: {},
            },
            easy: {
              name: 'Легкий',
              multiplicationOrDivision: {
                maxResult: 1000,
                minResult: 100,
                minArg: 10,
                maxArg: 100,
              },
              additionOrSubtraction: {
                maxResult: 200,
                minResult: 0,
                minArg: 10,
                maxArg: 100,
              },
            },
            intermediate: {
              name: 'Средний',
              multiplicationOrDivision: {
                maxResult: 100000,
                minResult: 1000,
                minArg: 100,
                maxArg: 1000,
              },
              additionOrSubtraction: {
                maxResult: 2000,
                minResult: 200,
                minArg: 100,
                maxArg: 1000,
              },
            },
            hard: {
              name: 'Сложный',
              multiplicationOrDivision: {
                maxResult: 100000000,
                minResult: 1000000,
                minArg: 1000,
                maxArg: 10000,
              },
              additionOrSubtraction: {
                maxResult: 200000,
                minResult: 2000,
                minArg: 1000,
                maxArg: 10000,
              },
            },
            extraHard: {
              name: 'Очень сложный',
              multiplicationOrDivision: {
                maxResult: 1000000000000,
                minResult: 100000000,
                minArg: 10000,
                maxArg: 10000000,
              },
              additionOrSubtraction: {
                maxResult: 200000000,
                minResult: 200000,
                minArg: 10000,
                maxArg: 100000000,
              },
            },
          },
        },
        showCorrectAnswers: {
          label: 'Показывать правильный ответ при ошибке',
          value: false,
        },
        showTasksCount: {
          label: 'Показывать счетчик заданий',
          value: true,
        },
        showHints: {
          label: 'Показывать подсказки',
          value: false,
        },
      },
      sections: {
        Умножение: '*',
        Деление: '/',
        Сложение: '+',
        Вычитание: '-',
        Комбинированный: 'C',
      },
      subTasks: [],
      activeSubTaskContent: '',
      activeSection: 0,
      combinedSubSection: '',
      subTaskAnswer: '',
      visibleCorrectAnswer: '',
      sessions: {
        id: '',
        results: {},
      },
      taskResultMessage: '',
      noSolvedTasksMessage: 'Вы не решили ни одного заданий',
      solvedTasksMessages: ['Количество решенных заданий', 'Правильных ответов'],
      appTitle: 'Тренажер устного счета',
      taskOptionsHeader: 'Настройки',
      tableLevelName: 'Таблица',
      chevronIcons: {
        up: 'mdi-chevron-up',
        down: 'mdi-chevron-down',
      },
      breakButtonTexts: {
        pause: 'Пауза',
        continue: 'Продолжить',
      },
      controlButtons: {
        start: 'Старт',
        stop: 'Стоп',
        pause: 'Пауза',
        continue: 'Продолжить',
        submit: 'Ответить',
        restart: 'Начать заново',
      },
    };
  },

  computed: {
    subTaskResult() {
      // if combined section is active, use its current subsection index
      const section = this.combinedSubSection !== '' ? this.combinedSubSection : this.activeSection;
      const firstArg = this.activeSubTaskContent[0];
      const secondArg = this.activeSubTaskContent[1];
      switch (section) {
        case 0:
          return firstArg * secondArg;
        case 1:
          return firstArg / secondArg;
        case 2:
          return firstArg + secondArg;
        case 3:
          return firstArg - secondArg;
        default:
          return '';
      }
    },
    subTaskSign() {
      const section = this.combinedSubSection !== '' ? this.combinedSubSection : this.activeSection;
      switch (section) {
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
        const levelName = item.name;
        // generate table level name for multiplication/division
        if (
          levelName === this.tableLevelName
          && this.activeSection <= 1
        ) {
          const sectionName = Object.keys(this.sections)[this.activeSection];
          names.push(`${sectionName} ${levelName}`);
        }
        // generate other levels
        if (levelName !== this.tableLevelName) names.push(levelName);
      });
      return names;
    },
    correctAnswerPrettified() {
      return this.addCommas(this.visibleCorrectAnswer);
    },
    answerHints() {
      let answer = this.subTaskResult;
      let firstHint;
      let secondHint;
      if (answer <= 10) {
        firstHint = answer + this.generateRandomNumber(1, 2);
        secondHint = answer + this.generateRandomNumber(3, 4);
      } else if (answer <= 100) {
        firstHint = answer + this.generateRandomNumber(1, 5);
        secondHint = answer + this.generateRandomNumber(6, 10);
      } else if (answer <= 1000) {
        firstHint = answer + this.generateRandomNumber(10, 50);
        secondHint = answer + this.generateRandomNumber(51, 100);
      } else if (answer <= 10000) {
        firstHint = answer + this.generateRandomNumber(100, 200);
        secondHint = answer + this.generateRandomNumber(201, 300);
      } else {
        firstHint = answer + this.generateRandomNumber(300, 650);
        secondHint = answer + this.generateRandomNumber(651, 1000);
      }
      // add commas
      answer = this.addCommas(answer);
      firstHint = this.addCommas(firstHint);
      secondHint = this.addCommas(secondHint);
      return this.shuffleArray([answer, firstHint, secondHint]);
    },
    startButtonIsVisible() {
      return !this.started && !this.taskResultMessage;
    },
    hintsAreVisible() {
      return this.taskOptions.showHints.value && !this.visibleCorrectAnswer;
    },
    chevron() {
      return this.showTaskOptions ? this.chevronIcons.up : this.chevronIcons.down;
    },
    breakButtonText() {
      return this.paused ? this.breakButtonTexts.continue : this.breakButtonTexts.pause;
    },
    subTaskFirstArgument() {
      return this.addCommas(this.activeSubTaskContent[0]);
    },
    subTaskSecondArgument() {
      return this.addCommas(this.activeSubTaskContent[1]);
    },
  },

  methods: {
    startTask() {
      this.initializeSession();
      this.generateSubTasks();
      this.showSubTask();
      this.hideTaskOptions();
      this.startTimer();
      this.clearTaskResultMessage();
      this.started = true;
    },
    stopTask() {
      this.clearSubTasks();
      this.clearSubTaskAnswer();
      this.clearCombinedSubSection();
      this.createResultMessage();
      this.clearVisibleCorrectAnswer();
      this.stopTimer();
      this.started = false;
      this.paused = false;
    },
    submitAnswer() {
      if (!this.answerValueCorrect(this.subTaskAnswer)) return;
      // store user's answers
      const results = this.sessions.results[this.sessions.id];
      if (+this.subTaskAnswer === this.subTaskResult) {
        results.correct.push(this.activeSubTaskContent);
      } else {
        this.showCorrectAnswer();
        // save incorrect sub-task answer as the last item in sub-task content array
        // not used now
        this.activeSubTaskContent.push(+this.subTaskAnswer);
        results.incorrect.push(this.activeSubTaskContent);
      }
      this.goToNextSubTask();
    },
    submitAnswerFromHints(hint) {
      const answer = Number(hint.replace(/,/g, ''));
      this.subTaskAnswer = answer;
      this.submitAnswer();
    },
    createResultMessage() {
      const correctAnswersQuantity = this.getCorrectAnswersQuantity();
      const subTasksQuantity = correctAnswersQuantity + this.getIncorrectAnswersQuantity();
      if (subTasksQuantity === 0) {
        this.taskResultMessage = this.noSolvedTasksMessage;
      } else {
        const answersCorrectPercent = Math.round((correctAnswersQuantity / subTasksQuantity) * 100);
        this.taskResultMessage = `${this.solvedTasksMessages[0]}: ${subTasksQuantity}.
        ${this.solvedTasksMessages[1]}: ${correctAnswersQuantity} (${answersCorrectPercent}%)`;
      }
    },
    getCorrectAnswersQuantity() {
      const results = this.sessions.results[this.sessions.id];
      return results.correct.length;
    },
    getIncorrectAnswersQuantity() {
      const results = this.sessions.results[this.sessions.id];
      return results.incorrect.length;
    },
    getTasksLeftQuantity() {
      return this.taskOptions.tasksLimit.value
        - this.getIncorrectAnswersQuantity() - this.getCorrectAnswersQuantity();
    },
    taskFinished() {
      return this.taskOptions.tasksLimit.isSet && this.getTasksLeftQuantity() === 0;
    },
    hideTaskOptions() {
      this.showTaskOptions = false;
    },
    showSubTask() {
      const nextTask = this.subTasks.splice(0, 1)[0];
      this.activeSubTaskContent = nextTask;
    },
    processBreakButtonClick() {
      this.paused = !this.paused;
      if (this.paused) {
        clearInterval(this.taskOptions.timer.timerInstance);
      } else this.startTimer();
    },
    clearSubTaskAnswer() {
      this.subTaskAnswer = '';
    },
    clearCombinedSubSection() {
      this.combinedSubSection = '';
    },
    clearSubTasks() {
      this.activeSubTaskContent = '';
      this.subTasks = [];
    },
    clearTaskResultMessage() {
      this.taskResultMessage = '';
    },
    clearVisibleCorrectAnswer() {
      this.visibleCorrectAnswer = '';
    },
    initializeSession() {
      if (!this.sessions.id) {
        this.sessions.id = 1;
      } else this.sessions.id += 1;
      this.sessions.results[this.sessions.id] = {
        correct: [],
        incorrect: [],
      };
    },
    showCorrectAnswer() {
      if (this.taskOptions.showCorrectAnswers.value) {
        this.visibleCorrectAnswer = this.subTaskResult;
      }
    },
    goToNextSubTask() {
      if (!this.taskFinished()) {
        this.clearSubTaskAnswer();
        if (this.subTasks.length === 0) this.generateSubTasks();
        this.showSubTask();
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
      const isTable = this.taskOptions.levels.selected.indexOf(this.tableLevelName) > -1;
      // generate sub-tasks for tables
      if (isTable) {
        // multiplication table
        if (this.activeSection === 0) {
          this.subTasks = this.generateTableSubTasks(true);
        // division table
        } else {
          this.subTasks = this.generateTableSubTasks();
        }
      // generate sub-task for multiplication section
      } else if (this.activeSection === 0) {
        this.subTasks.push(this.generateMultiplicationOrDivisionSubTask(true));
        // generate sub-task for division section
      } else if (this.activeSection === 1) {
        this.subTasks.push(this.generateMultiplicationOrDivisionSubTask());
        // generate sub-task for addition section
      } else if (this.activeSection === 2) {
        this.subTasks.push(this.generateAdditionOrSubtractionSubTask(true));
        // generate sub-task for subtraction section
      } else if (this.activeSection === 3) {
        this.subTasks.push(this.generateAdditionOrSubtractionSubTask());
        // generate sub-task for combined section
      } else if (this.activeSection === 4) {
        const randomSection = this.generateRandomNumber(0, 3);
        this.combinedSubSection = randomSection;
        switch (randomSection) {
          case 0:
            this.subTasks.push(this.generateMultiplicationOrDivisionSubTask(true));
            break;
          case 1:
            this.subTasks.push(this.generateMultiplicationOrDivisionSubTask());
            break;
          case 2:
            this.subTasks.push(this.generateAdditionOrSubtractionSubTask(true));
            break;
          case 3:
            this.subTasks.push(this.generateAdditionOrSubtractionSubTask());
            break;
          default:
            break;
        }
      }
    },
    // generates all possible sub-tasks for the table level
    generateTableSubTasks(isMultiplication) {
      const level = this.taskOptions.levels.items.table;
      const {
        maxArg,
        minArg,
        maxResult,
        minResult,
      } = level.multiplicationOrDivision;
      const tasks = [];
      let leftArg = minArg;
      let rightArg = minArg;
      while (leftArg <= maxResult && leftArg * rightArg <= maxResult) {
        if (leftArg * rightArg >= minResult && leftArg <= rightArg && rightArg <= maxArg) {
          // multiplication table arguments
          if (isMultiplication) {
            tasks.push([leftArg, rightArg]);
          // division table arguments
          } else {
            tasks.push([leftArg * rightArg, leftArg]);
          }
        }
        rightArg += 1;
        if (rightArg > maxResult || leftArg * rightArg > maxResult) {
          rightArg = minArg;
          leftArg += 1;
        }
      }
      // change the order of items in the resulting array
      return this.shuffleArray(tasks);
    },
    generateMultiplicationOrDivisionSubTask(isMultiplication) {
      const level = this.getLevelOptions();
      const {
        maxArg,
        minArg,
        maxResult,
        minResult,
      } = level.multiplicationOrDivision;
      let leftArg = this.generateRandomNumber(minArg, maxArg);
      let rightArg = this.generateRandomNumber(minArg, maxArg);
      while (
        leftArg * rightArg < minResult
        || leftArg * rightArg > maxResult
      ) {
        leftArg = this.generateRandomNumber(minArg, maxArg);
        rightArg = this.generateRandomNumber(minArg, maxArg);
      }
      // multiplication arguments
      if (isMultiplication) return [leftArg, rightArg];
      // division arguments
      return [leftArg * rightArg, leftArg];
    },
    generateAdditionOrSubtractionSubTask(isAddition) {
      const level = this.getLevelOptions();
      const {
        maxArg,
        minArg,
        maxResult,
        minResult,
      } = level.additionOrSubtraction;
      let leftArg = this.generateRandomNumber(minArg, maxArg);
      let rightArg = this.generateRandomNumber(minArg, maxArg);
      while (
        (leftArg + rightArg < minResult || leftArg + rightArg > maxResult)
        || (this.activeSection === 3 && rightArg < minResult)
      ) {
        leftArg = this.generateRandomNumber(minArg, maxArg);
        rightArg = this.generateRandomNumber(minArg, maxArg);
      }
      // addition arguments
      if (isAddition) return [leftArg, rightArg];
      // subtraction arguments
      return [leftArg + rightArg, leftArg];
    },
    getLevelOptions() {
      return Object.values(this.taskOptions.levels.items)
        .find(item => item.name === this.taskOptions.levels.selected);
    },
    // https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Math/random
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
    // https://stackoverflow.com/a/2901298
    addCommas(num) {
      try {
        return num.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ',');
      } catch (e) {
        return '';
      }
    },
  },

  watch: {
    // stop task if the time has ended
    timerPercent(val) {
      if (val === 100) this.stopTask();
    },
    // if table level was selected for multiplication/division,
    // change it to default level on section change
    activeSection() {
      if (this.taskOptions.levels.selected.indexOf(this.tableLevelName) > -1) {
        this.taskOptions.levels.selected = this.taskOptions.levels.default;
      }
    },
  },
};
</script>

<style lang="stylus" scoped>
  .options
    text-align center
  .task
    font-size 18px
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
  .correct-answer
    color #EF9A9A
  .option
    cursor pointer
  .mobile-section-button
    font-size 16px
</style>
