<template>
  <div id="app">
    <div id="form"></div>
    <div id="gantt-header" class="h-12 p-2">
      <button
        @click="addTask"
        class="bg-blue-500 hover:bg-blue-600 text-white py-2 px-4 text-xs ml-4"
      >
        <span class="font-bold text-xs"> タスクの追加 </span>
      </button>
      <div>
        <div
          class="fixed top-0 left-0 right-0 flex justify-center mt-24 z-50"
          v-show="show"
        >
          <div class="overlay" v-show="show" @click="show=false"></div>
          <div class="content" v-show="show">
            <h2 class="font-bold" v-if="update_mode">タスクの更新</h2>
            <h2 class="font-bold" v-else>タスクの追加</h2>
            <div class="my-4">
              <label class="text-xs">タスクのタイトル</label>
              <input
                class="text-xs border rounded-lg px-4 py-2"
                v-model="form.name"
              />
            </div>
            <div class="my-4">
              <label class="text-xs">タスクオーナー</label>
              <input
                class="text-xs border rounded-lg px-4 py-2"
                v-model="form.incharge_user"
              />
            </div>
            <div class="my-4">
              <label class="text-xs">開始日</label>
              <input
                class="text-xs border rounded-lg px-4 py-2"
                v-model="form.start_date"
              />
            </div>
            <div class="my-4">
              <label class="text-xs">完了期限日</label>
              <input
                class="text-xs border rounded-lg px-4 py-2"
                v-model="form.end_date"
              />
            </div>
            <div class="my-4">
              <label class="text-xs">タスク完了率</label>
              <input
                class="text-xs border rounded-lg px-4 py-2"
                v-model="form.percentage"
              />
            </div>
            <div v-if="update_mode" class="flex items-center justify-between">
              <button
                @click="updateTask(form.id)"
                class="bg-blue-500 hover:bg-blue-600 text-white font-bold py-2 px-4 rounded-lg flex items-center"
              >
                <span class="font-bold text-xs">タスクを更新</span>
              </button>
              <button
                @click="deleteTask(form.id)"
                class="bg-red-500 hover:bg-red-600 text-white py-2 px-4 rounded-lg flex items-center ml-2"
              >
                <span class="text-xs font-bold text-white">タスクを削除</span>
              </button>
            </div>
            <div v-else>
              <button
                @click="saveTask"
                class="bg-blue-500 hover:bg-blue-600 text-white font-bold py-2 px-4 rounded-lg flex items-center"
              >
                <span class="font-bold text-xs">タスクを追加</span>
              </button>
            </div>
          </div>
        </div>
      </div>
    </div>
    <div id="gantt-content" class="flex">
      <div id="gantt-task">
        <div id="gantt-task">
          <div
            id="gantt-task-title"
            class="flex items-center bg-gray-300 h-20"
            ref="task"
          >
            <div
              class="border-t border-r border-b flex items-center justify-center font-bold text-xs w-48 h-full"
            >
              タスクのタイトル
            </div>
            <div
              class="border-t border-r border-b flex items-center justify-center font-bold text-xs w-32 h-full"
            >
              タスクオーナー
            </div>
            <div
              class="border-t border-r border-b flex items-center justify-center font-bold text-xs w-24 h-full"
            >
              開始日
            </div>
            <div
              class="border-t border-r border-b flex items-center justify-center font-bold text-xs w-24 h-full"
            >
              完了期限日
            </div>

            <div
              class="border-t border-r border-b flex items-center justify-center font-bold text-xs w-32 h-full"
            >
              タスク完了率
            </div>
          </div>

          <div id="gantt-task-list">
            <div
              v-for="(list,index) in lists"
              :key="index"
              class="flex h-10 border-b"
              @dragstart="dragTask(list)"
              @dragover.prevent="dragTaskOver(list)"
              draggable="true"
            >
                <div
                  class="border-r flex items-center font-bold w-48 text-sm pl-4"
                >
                  {{list.name}}
                </div>

                <div
                  class="border-r flex items-center justify-center w-32 text-sm"
                >
                  {{list.incharge_user}}
                </div>
                <div
                  class="border-r flex items-center justify-center w-24 text-sm"
                >
                  {{list.start_date}}
                </div>
                <div
                  class="border-r flex items-center justify-center w-24 text-sm"
                >
                  {{list.end_date}}
                </div>

                <div class="flex items-center justify-center w-12 text-sm">
                  {{list.percentage}}%
                </div>
            </div>
          </div>
        </div>
      </div>
      <div
        id="gantt-calendar"
        class="overflow-x-scroll"
        :style="`width:${calendarViewWidth}px`"
        ref="calendar"
      >
        <div id="gantt-date" class="h-20">
          <div id="gantt-year-month" class="relative h-8">
            <div v-for="(calendar,index) in calendars" :key="index">
              <div
                class="bg-blue-700 text-white border-b border-r border-t h-8 absolute font-bold text-sm flex items-center justify-center"
                :style="`width:${calendar.calendar*block_size}px;left:${calendar.start_block_number*block_size}px`"
              >
                {{calendar.date}}
              </div>
            </div>
          </div>
          <div id="gantt-day" class="relative h-12">
            <div v-for="(calendar,index) in calendars" :key="index">
              <div v-for="(day,index) in calendar.days" :key="index">
                <div
                  class="border-r border-b h-12 absolute flex items-center justify-center flex-col font-bold text-xs"
                  :style="`width:${block_size}px;left:${day.block_number*block_size}px`"
                >
                  <span
                    class="bg-blue-400 h-6 pt-1 border-b block w-full text-center"
                    >{{ day.day }}</span
                  >
                  <span class="bg-blue-300 h-6 pt-1 block w-full text-center"
                    >{{ day.dayOfWeek }}</span
                  >
                </div>
              </div>
            </div>
          </div>
          <div id="gantt-height" class="relative">
            <div v-for="(calendar,index) in calendars" :key="index">
              <div v-for="(day,index) in calendar.days" :key="index">
                <div
                  class="border-r border-b absolute"
                  :style="`width:${block_size}px;left:${day.block_number*block_size}px;height:${calendarViewHeight}px`"
                ></div>
              </div>
            </div>
          </div>
        </div>
        <div
          id="gantt-bar-area"
          class="relative"
          :style="`width:${calendarViewWidth}px;height:${calendarViewHeight}px`"
        >
          <div v-for="(bar,index) in taskBars" :key="index">
            <div
              :style="bar.style"
              class="absolute h-5 bg-gray-200"
              v-if="bar.list.cat === 'task'"
              @mousedown="mouseDownMove(bar.list)"
            >
              <div class="w-full h-full" style="pointer-events: none">
                <div
                  class="h-full bg-blue-500"
                  style="pointer-events: none"
                  :style="`width:${bar.list.percentage}%`"
                ></div>
              </div>
              <div
                class="absolute w-2 h-full" style="top:0px;left:0px;cursor:col-resize"
                @mousedown.stop="mouseDownResize(bar.list,'left')"
              ></div>
              <div
                class="absolute w-2 h-full" style="top:0;right:0px;cursor:col-resize"
                @mousedown.stop="mouseDownResize(bar.list,'right')"
              ></div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>

import moment from "moment";

export default {
  name: 'App',
   data() {
          return {
            start_month: moment().subtract(1, 'months').format('YYYY-MM'),
            end_month: moment().add(1, 'months').format('YYYY-MM'),
            block_size: 30,
            block_number: 0,
            calendars: [],
            inner_width: '',
            inner_height: '',
            task_width: '',
            task_height: '',
            today: moment(),
            dragging: false,
            pageX: '',
            elememt: '',
            left: '',
            task_id: '',
            width: '',
            leftResizing: false,
            rightResizing: false,
            task: '',
            show: false,
            update_mode: false,
            form: {
              id: '',
              name: '',
              start_date: '',
              end_date: '',
              incharge_user: '',
              percentage: 0,
            },

            tasks: [],
          };
        },
        methods: {
          getDays(year, month, block_number) {
            const dayOfWeek = ['日', '月', '火', '水', '木', '金', '土'];
            let days = [];
            let date = moment(`${year}-${month}-01`); 
            let num = date.daysInMonth();
            for (let i = 0; i < num; i++) {
              days.push({
                day: date.date(),
                dayOfWeek: dayOfWeek[date.day()],
                block_number,
              });
              date.add(1, 'day');
              block_number++;
            }
            return days;
          },

          getCalendar() {
            let block_number = 0;
            let days;
            let start_month = moment(this.start_month);
            let end_month = moment(this.end_month);
            let between_month = end_month.diff(start_month, 'months');
            for (let i = 0; i <= between_month; i++) {
              days = this.getDays(
                start_month.year(),
                start_month.format('MM'),
                block_number
              );
              this.calendars.push({
                date: start_month.format('YYYY年MM月'),
                year: start_month.year(),
                month: start_month.month(),
                start_block_number: block_number,
                calendar: days.length,
                days: days,
              });
              start_month.add(1, 'months');
              block_number = days[days.length - 1].block_number;
              block_number++;
            }
            return block_number;
          },

          getWindowSize() {
            this.inner_width = window.innerWidth;
            this.inner_height = window.innerHeight;
            this.task_width = this.$refs.task.offsetWidth;
            this.task_height = this.$refs.task.offsetHeight;
          },
          todayPosition() {
            this.$refs.calendar.scrollLeft = this.scrollDistance;
          },
          mouseDownMove(list) {
            this.dragging = true;
            this.pageX = event.pageX;
            this.element = event.target;
            this.left = event.target.style.left;
            this.task_id = list.id;
          },

          mouseMove() {
            if (this.dragging) {
              let diff = this.pageX - event.pageX;
              this.element.style.left = `${
                parseInt(this.left.replace('px', '')) - diff
              }px`;
            }
          },
          stopDrag() {
            if (this.dragging) {
              let diff = this.pageX - event.pageX;
              let days = Math.ceil(diff / this.block_size);
              if (days !== 0) {
                let task = this.tasks.find((task) => task.id === this.task_id);
                let start_date = moment(task.start_date).add(-days, 'days');
                let end_date = moment(task.end_date).add(-days, 'days');
                task['start_date'] = start_date.format('YYYY-MM-DD');
                task['end_date'] = end_date.format('YYYY-MM-DD');
              } else {
                this.element.style.left = `${this.left.replace('px', '')}px`;
              }
            }
            if (this.leftResizing) {
              let diff = this.pageX - event.pageX;
              let days = Math.ceil(diff / this.block_size);
              if (days !== 0) {
                let task = this.tasks.find((task) => task.id === this.task_id);
                let start_date = moment(task.start_date).add(-days, 'days');
                let end_date = moment(task.end_date);
                if (end_date.diff(start_date, 'days') <= 0) {
                  task['start_date'] = end_date.format('YYYY-MM-DD');
                } else {
                  task['start_date'] = start_date.format('YYYY-MM-DD');
                }
              } else {
                this.element.style.width = this.width;
                this.element.style.left = `${this.left.replace('px', '')}px`;
              }
            }

            if (this.rightResizing) {
              let diff = this.pageX - event.pageX;
              let days = Math.ceil(diff / this.block_size);
              if (days === 1) {
                this.element.style.width = `${parseInt(
                  this.width.replace('px', '')
                )}px`;
              } else if (days <= 2) {
                days--;
                let task = this.tasks.find((task) => task.id === this.task_id);
                let end_date = moment(task.end_date).add(-days, 'days');
                task['end_date'] = end_date.format('YYYY-MM-DD');
              } else {
                let task = this.tasks.find((task) => task.id === this.task_id);
                let start_date = moment(task.start_date);
                let end_date = moment(task.end_date).add(-days, 'days');
                if (end_date.diff(start_date, 'days') < 0) {
                  task['end_date'] = start_date.format('YYYY-MM-DD');
                } else {
                  task['end_date'] = end_date.format('YYYY-MM-DD');
                }
              }
            }
            this.dragging = false;
            this.leftResizing = false;
            this.rightResizing = false;
          },

          mouseDownResize(task, direction) {
            direction === 'left'
              ? (this.leftResizing = true)
              : (this.rightResizing = true);
            this.pageX = event.pageX;
            this.width = event.target.parentElement.style.width;
            this.left = event.target.parentElement.style.left;
            this.element = event.target.parentElement;
            this.task_id = task.id;
          },

          mouseResize() {
            if (this.leftResizing) {
              let diff = this.pageX - event.pageX;
              if (
                parseInt(this.width.replace('px', '')) + diff >
                this.block_size
              ) {
                this.element.style.width = `${
                  parseInt(this.width.replace('px', '')) + diff
                }px`;
                this.element.style.left = `${
                  this.left.replace('px', '') - diff
                }px`;
              }
            }
            if (this.rightResizing) {
              let diff = this.pageX - event.pageX;
              if (
                parseInt(this.width.replace('px', '')) - diff >
                this.block_size
              ) {
                this.element.style.width = `${
                  parseInt(this.width.replace('px', '')) - diff
                }px`;
              }
            }
          },

          dragTask(dragTask) {
            this.task = dragTask;
          },

          dragTaskOver(overTask) {
            let deleteIndex;
            let addIndex;
            if (overTask.id !== this.task.id) {
              this.tasks.map((task, index) => {
                if (task.id === this.task.id) deleteIndex = index;
              });
              this.tasks.map((task, index) => {
                if (task.id === overTask.id) addIndex = index;
              });
              this.tasks.splice(deleteIndex, 1);
              this.tasks.splice(addIndex, 0, this.task);
            }
          },

          addTask() {
            this.update_mode = false;
            this.form = {};
            this.show = true;
          },

          saveTask() {
            this.form.id = Math.random();
            this.tasks.push(this.form);
            this.form = {};
            this.show = false;
            console.log(this.tasks);
          },

        },
        mounted() {
          this.getCalendar();
          this.getWindowSize();
          this.$nextTick(() => {
            this.todayPosition();
          });
          window.addEventListener('mousemove', this.mouseMove);
          window.addEventListener('mouseup', this.stopDrag);
          window.addEventListener('mousemove', this.mouseResize);
        },
        computed: {
          calendarViewWidth() {
            return this.inner_width - this.task_width;
          },

          calendarViewHeight() {
            return this.inner_height - this.task_height - 48 - 20;
          },

          scrollDistance() {
            let start_date = moment(this.start_month);
            let between_days = this.today.diff(start_date, 'days');
            return (
              (between_days + 1) * this.block_size - this.calendarViewWidth / 2
            );
          },

          lists() {
            let lists = [];
            this.tasks.map((task) => {
              lists.push({ cat: 'task', ...task });
            });
            return lists;
          },

          taskBars() {
            let start_date = moment(this.start_month);
            let top = 10;
            let left;
            let between;
            let start;
            let style;
            return this.lists.map((list) => {
              style = {};
              let date_from = moment(list.start_date);
              let date_to = moment(list.end_date);
              between = date_to.diff(date_from, 'days');
              between++;
              start = date_from.diff(start_date, 'days');
              left = start * this.block_size;
              style = {
                top: `${top}px`,
                left: `${left}px`,
                width: `${this.block_size * between}px`,
              };

              top = top + 40;
              return {
                style,
                list,
              };
            });
          },
        },
      };
</script>

<style>
.overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: gray;
  opacity: 0.5;
}

.content {
  background-color: white;
  position: relative;
  border-radius: 10px;
  padding: 40px;
}
</style>

<style src="./assets/tailwind.css" />
