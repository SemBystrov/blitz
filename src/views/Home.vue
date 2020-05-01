<template>
  <v-container fluid class="timetable-bg">
    <v-row justify="center" :align="'center'" class="timetable">
      <v-col cols="12" sm="11" md="8" lg="6">
        <v-card v-if="has_actual_task" elevation="6" class="pa-3">
          <v-card-title class="headline">
            {{ get_actual_task.title }}
            <v-spacer></v-spacer>
            <v-card-subtitle class="mt-1">
              <countdown :refresh="refresh_token" ref="countdown" :time="get_left_time" :emit-events="true" @start="activators.end_time = false" @end="activators.end_time = true">
                <template v-if="!activators.end_time" slot-scope="props">{{ props.minutes }}:{{ props.seconds }}</template>

              </countdown>
              <template v-if="activators.end_time">
                BLITZ!
              </template>
            </v-card-subtitle>
          </v-card-title>

          <v-card-text>
            {{ get_actual_task.text }}
          </v-card-text>
          <v-bottom-navigation v-if="activators.end_time" class="elevation-0 mt-8 mb-8">
            <v-btn @click="to_end_task">
              <span>В очередь</span>
              <v-icon>mdi-alarm-snooze</v-icon>
            </v-btn>
            <v-btn @click="update_time_task">
              <span>Продолжить</span>
              <v-icon>mdi-alarm-plus</v-icon>
            </v-btn>
            <v-btn icon @click="delete_task(0)">
              <span class="green--text text--darken-2">Готово</span>
              <v-icon class="green--text text--darken-2">mdi-alarm-check</v-icon>
            </v-btn>
          </v-bottom-navigation>
        </v-card>
        <v-card v-else>
          <v-card-title>
            <v-spacer></v-spacer>Нет задач<v-spacer></v-spacer>
          </v-card-title>
          <v-card-actions>
            <v-spacer></v-spacer>
              <v-btn
                x-large
                color="black"
                text
                elevation="4"
                class="ma-8"
                @click="activators.add_task = true"
              >
                <v-icon class="mr-2">mdi-notebook</v-icon>
                Создать новую!
              </v-btn>
            <v-spacer></v-spacer>

          </v-card-actions>
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
    <v-bottom-sheet v-model="activators.setting" fullscreen class="bg">
      <v-sheet class="bg">
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
        <v-row justify="center">
          <v-col cols="8">
            <v-list v-if="task_list.length > 0">
              <draggable v-model="task_list" @change="update_time_task">
                  <v-list-item v-for="(task, index) in task_list" :key="task.title" class="elevation-1 mt-2">
                    {{ task.title }}
                    <v-spacer></v-spacer>
                    <v-btn :key="'task-item-btn-' + task.title" icon @click="delete_task(index)"><v-icon>mdi-close</v-icon></v-btn>
                  </v-list-item>
              </draggable>


            </v-list>
            <v-alert v-else>Задач нет</v-alert>
            <v-btn class="mt-4" @click="activators.add_task = true"><v-icon>mdi-plus</v-icon>Добавить</v-btn>
          </v-col>
        </v-row>

      </v-sheet>
    </v-bottom-sheet>

    <v-dialog
      v-model="activators.add_task"
      max-width="300"
    >
      <v-card class="pa-4">
        <v-card-title class="mb-3 pl-0">Добавление задачи</v-card-title>

        <v-form>
          <v-text-field solo v-model="new_task.title" label="Название задачи" ></v-text-field>
          <v-textarea solo v-model="new_task.text" label="Текст задачи"></v-textarea>
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

const min40 = 2400000;

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
        text: '',
        time: min40
      },
      task_list: [],
      refresh_token: 1,
    }
  },
  computed: {
    get_actual_task: ({task_list}) => {
      return task_list[0];
    },
    has_actual_task: ({task_list}) => {
      return task_list.length > 0
    },
    get_left_time ({task_list, refresh_token}) {
      return task_list[0].time + refresh_token;
    }
  },
  methods: {
    refresh_countdown (){
      this.refresh_token = -1 * this.refresh_token;
    },
    add_task () {
      let task = {
        ...this.new_task
      };
      this.task_list.push(task);
      this.new_task.title = '';
      this.new_task.text = '';
      this.activators.add_task = false;
    },
    delete_task (index) {
      this.task_list.splice(index, 1);
    },
    to_end_task () {
      let task = {
        ...this.task_list[0]
      };
      this.delete_task(0);
      this.task_list.push(task);
      this.refresh_countdown();
    },
    update_time_task () {
      this.refresh_countdown();
    }
  },
  mounted() {
  },
}
</script>


<style scope lang="scss">
  .timetable {
    height: 100vh;
    background: url('../assets/logo.png');
  }
  .bg {
    height: 100vh;
  }
</style>