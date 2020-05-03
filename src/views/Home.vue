<template>
  <v-container fluid class="timetable">
    <v-row justify="center" :align="'center'" class="timetable">
      <v-col cols="12" sm="11" md="8" lg="6">
        <v-card elevation="6" class="pa-3">
          <v-card-title class="headline">

            <template v-if="has_actual_task">
              {{ get_actual_task.title }}
            </template>

            <template v-else>
              <v-spacer></v-spacer>
              Нет задач
            </template>


            <v-spacer></v-spacer>
            <v-card-subtitle v-show="has_actual_task" class="mt-1">
              <countdown ref="countdown" :time="get_left_time" :emit-events="true" @start="activators.end_time = false" @progress="actual_task.passed += 1" @end="activators.end_time = true">
                <template v-if="!activators.end_time" slot-scope="props">{{ props.minutes }}:{{ props.seconds }}</template>

              </countdown>
              <template v-if="activators.end_time">
                BLITZ!
              </template>
            </v-card-subtitle>

          </v-card-title>


          <template v-if="has_actual_task">

            <v-card-text>
              {{ get_actual_task.text }}
            </v-card-text>

            <template v-if="activators.end_time">
              <v-bottom-navigation class="elevation-0 mt-8 mb-8">
                <v-btn @click="to_end_task" text v-if="this.task_list.length > 1">
                  <span>В очередь</span>
                  <v-icon>mdi-alarm-snooze</v-icon>
                </v-btn>
                <v-btn @click="refresh_countdown">
                  <span>Продолжить</span>
                  <v-icon>mdi-alarm-plus</v-icon>
                </v-btn>
                <v-btn icon @click="task_done">
                  <span class="green--text text--darken-2">Готово</span>
                  <v-icon class="green--text text--darken-2">mdi-alarm-check</v-icon>
                </v-btn>
              </v-bottom-navigation>

              <v-card flat class="pa-3 pa-sm-12 pt-4">
                <v-card-title>
                  Сделано за сеанс:
                </v-card-title>
                <v-list>
                  <v-list-item-group>
                    <v-list-item v-for="t in done_task_list" :key="'done-' + t.title">
                      {{ t.title }}
                    </v-list-item>
                  </v-list-item-group>
                </v-list>
              </v-card>

            </template>
            <v-slider v-else v-model="actual_task.passed" min="0" max="2400" append-icon="mdi-alarm" class="ml-3 mr-3" color="black" track-color="grey" @change="change_progress">

            </v-slider>

          </template>

          <template v-else>
            <v-card-actions>
              <v-spacer></v-spacer>
                <v-btn
                  x-large
                  color="black"
                  text
                  elevation="4"
                  class="ma-8"
                  @click.stop="activators.add_task = true"
                >
                  <v-icon class="mr-2">mdi-notebook</v-icon>
                  Создать новую!
                </v-btn>
              <v-spacer></v-spacer>

            </v-card-actions>
          </template>



        </v-card>
      </v-col>
    </v-row>


    <v-fab-transition>
      <v-btn
        color="black"
        class="ma-2 ma-sm-6 ma-md-12"
        bottom
        right
        large
        dark
        fixed
        elevation="12"
        @click="activators.setting = true"
        fab
      >
        <v-icon>mdi-plus</v-icon>
      </v-btn>
    </v-fab-transition>


    <v-bottom-sheet v-model="activators.setting" fullscreen class="bg" scrollable>
      <v-card white class="bg">
        <v-btn
          color="black"
          class="ma-1 ma-sm-8 ma-md-12"
          top
          large
          right
          dark
          fixed
          fab
          elevation="12"
          @click="activators.setting = false"
        >
          <v-icon>mdi-close</v-icon>
        </v-btn>
        <v-card-text>
          <v-container>
            <v-row justify="center">
              <v-col cols="8">



                <v-list v-if="task_list.length > 0">

                  <draggable v-model="task_list" @change="update_task">
                      <v-list-item v-for="(task, index) in task_list" :key="task.title" class="elevation-1 mt-2">
                        {{ task.title }}
                        <v-spacer></v-spacer>
                        <v-btn :key="'task-item-btn-' + task.title" icon @click="delete_task(index)"><v-icon>mdi-close</v-icon></v-btn>
                      </v-list-item>
                  </draggable>


                </v-list>
                <v-alert v-else>Задач нет</v-alert>
                </v-col>
            </v-row>
          </v-container>
        </v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn x-large class="ma-12" @click.stop="activators.add_task = true"><v-icon>mdi-plus</v-icon>Добавить</v-btn>
          <v-spacer></v-spacer>
        </v-card-actions>
      </v-card>
    </v-bottom-sheet>


    <v-dialog
      v-model="activators.add_task"
      max-width="300"
    >
      <v-card class="pa-4">
        <v-card-title class="mb-3 pl-0">Добавление задачи</v-card-title>

        <v-form v-if="activators.add_task">
          <v-text-field solo v-model="new_task.title" placeholder="Название задачи"></v-text-field>
          <v-textarea solo v-model="new_task.text" placeholder="Текст задачи"></v-textarea>
        </v-form>

        <v-card-actions>
          <v-spacer></v-spacer>

          <v-btn
            color="black"
            @click="add_task"
            class="white--text"
          >
            Добавить
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </v-container>
</template>

<script>
import draggable from 'vuedraggable'
import {get, set} from 'idb-keyval'

const min40 = 2400002;

export default {
  name: 'Home',
  components: {
    draggable
  },
  data () {
    return {
      activators: {
        setting: false, // Открывает меню настроек
        add_task: false, // Открывает диалоговое окно добавления задачи

        end_time: false, // Показывает панель действий над завершившейся активной задачей
      },

      new_task: {
        title: '',
        text: ''
      },
      actual_task: {
        title: '',
        text: '',
        passed: 0,
        time: min40,
      },
      task_list: [],
      done_task_list: [],
      refresh: 1,
      refresh_passed: 0,
    }
  },
  async created () {
    let val = await get('task_list');
    if (typeof val !== 'undefined' && val.length !== 0){
      this.task_list = val;
      this.update_task();
    }
  },
  computed: {
    get_actual_task: ({actual_task}) => {
      return actual_task;
    },
    has_actual_task: ({task_list}) => {
      return task_list.length > 0
    },
    get_left_time ({actual_task, refresh, refresh_passed}) {
      return actual_task.time + refresh - refresh_passed;
    }
  },
  methods: {

    refresh_countdown (){
      this.actual_task.passed = 0;
      this.refresh_passed = 0;
      this.refresh = (-1) * this.refresh;
    },

    change_progress () {
      this.refresh_passed = this.actual_task.passed * 1000
      this.refresh = (-1) * this.refresh;
    },

    add_task () {
      let task = {
        ...this.new_task
      };

      this.task_list.push(task);
      this.new_task.title = '';
      this.new_task.text = '';
      this.activators.add_task = false;
      this.update_task();
    },

    delete_task (index) {
      this.task_list.splice(index, 1);
      this.update_task();
    },

    task_done () {
      let done_task = {
        ...this.task_list[0]
      }
      this.done_task_list.unshift(done_task);
      this.delete_task(0);
      console.log(this.done_task_list);
    },

    to_end_task () {
      let task = {
        ...this.task_list[0]
      };
      this.task_list.splice(0, 1);
      this.task_list.push(task);
      this.update_task();
    },

    update_task () {
      if ((this.has_actual_task) && !(this.actual_task.title === this.task_list[0].title && this.actual_task.text === this.task_list[0].text)) {
        let task = {
          passed: 0,
          time: min40,
          ...this.task_list[0]
        };
        this.actual_task = task;
        this.refresh_countdown();
      }
      set('task_list', this.task_list);
    }
  },

}
</script>


<style scope lang="scss">
  .timetable {
    height: 100%;
    background: url('../assets/logo.png');
  }
  .bg {
    height: 100vh;
  }
</style>