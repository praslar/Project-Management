<template>
  <div class="main">
    <project-header :memberData="memberData" :idProject="id"></project-header>
    <div class="info-box">
      <span class="info-box-icon bg-yellow">
        <i class="fa fa-files-o"></i>
      </span>
      <div class="info-box-content">
        <p class="info-box-number">{{memberData.name}}</p>
        <p class="info-box-text">{{tasks | count}} Task</p>
      </div>
    </div>
    <gantt-elastic
      :options="options"
      :tasks="tasks"
      @tasks-changed="tasksUpdate"
      @options-changed="optionsUpdate"
      @dynamic-style-changed="styleUpdate"
    >
      <gantt-header slot="header" :options="headerOptions"></gantt-header>
    </gantt-elastic>
    <taskModal></taskModal>
  </div>
</template>
<script>
import taskModal from './Elememts/TaskModal'
import GanttElastic from 'gantt-elastic'
import GanttHeader from 'gantt-elastic-header'
import dayjs from 'dayjs'
import ProjectHeader from '../../layout/Headers/ProjectHeader'
import * as Services from '../../../services'
import { eventBus } from '../../../main'
// just helper to get current dates
function getDate(hours) {
  const currentDate = new Date()
  const currentYear = currentDate.getFullYear()
  const currentMonth = currentDate.getMonth()
  const currentDay = currentDate.getDate()
  const timeStamp = new Date(
    currentYear,
    currentMonth,
    currentDay,
    0,
    0,
    0
  ).getTime()
  return new Date(timeStamp + hours * 60 * 60 * 1000).getTime()
}
export default {
  name: 'Gantt',
  props: ['id'],
  components: {
    GanttElastic,
    GanttHeader,
    ProjectHeader,
    taskModal
  },
  data() {
    return {
      memberData: {
        projectID: '',
        name: '',
        effort: '',
        startDate: '',
        updateDate: '',
        members: []
      },
      tasks: [
        {
          id: 0,
          label: 'Preloading',
          user: 'Preloading',
          start: getDate(0),
          duration: 1 * 24 * 60 * 60 * 1000,
          type: 'project'
        }
      ],
      headerOptions: {
        title: {
          label: 'ID: ' + this.id,
          html: false
        },
        locale: {
          name: 'en',
          Now: 'Now',
          'X-Scale': 'Width',
          'Y-Scale': 'Height',
          'Task list width': 'Task width',
          'Before/After': 'Expand',
          'Display task list': 'Hide Task'
        }
      },
      dynamicStyle: {},
      options: {
        scope: {
          before: 1,
          after: 80
        },
        maxRows: 1000,
        maxHeight: 1000,
        times: {
          timeZoom: 21
        },
        row: {
          height: 20
        },
        calendar: {
          hour: {
            display: false
          }
        },
        chart: {
          progress: {
            bar: false
          },
          text: {
            display: false
          },
          expander: {
            display: false
          }
        },
        taskList: {
          expander: {
            straight: true
          },
          columns: [
            {
              id: 1,
              label: 'ID',
              value: 'id',
              width: 35
            },
            {
              id: 2,
              label: 'Description',
              value: 'label',
              width: 100,
              expander: true,
              html: true,
              events: {
                click({ data }) {
                  // console.log(data)
                  eventBus.$emit('quan', data)
                }
              }
            },
            {
              id: 3,
              label: 'Assignee',
              value: 'user',
              width: 80
            },
            {
              id: 3,
              label: 'Start',
              value: task => dayjs(task.start).format('DD-MM-YYYY'),
              width: 78
            },
            {
              id: 4,
              label: 'End',
              value: task => dayjs(task.start + task.duration).format('DD-MM-YYYY'),
              width: 78
            }
          ]
        }
      }
    } // return
  },
  methods: {
    tasksUpdate(tasks) {
      this.tasks = tasks
    },
    optionsUpdate(options) {
      this.options = options
      // console.log(this.$modal)
    },
    styleUpdate(style) {
      this.dynamicStyle = style
    }
  },
  beforeCreate() {
    Services.getMemberOfProject()
      .then(response => {
        this.memberData = response
      })
      .catch(error => {
        console.log(error)
      })
    Services.getTasksByProjectID()
      .then(response => {
        this.tasks = response.tasks
      })
      .catch(error => {
        console.log(error)
      })
  },
  created () {
    eventBus.$on('quan', (data) => {
        // console.log(data)
        this.$modal.show('taskModal', { data: data })
    })
  }
}
</script>
<style>
.gantt-elastic__chart-days-highlight-rect {
  fill: #fae596 !important;
  z-index: 9999 !important;
}
.gantt-elastic__chart-days-highlight-rects{
   fill: rgba(255, 138, 138, 0.952) !important
}
.gantt-elastic__grid-line-time {
    stroke-width: 4px !important
}
</style>
<style scoped>
.info-box-icon {
  height: 50px;
  font-size: 30px;
  line-height: 40px;
  padding: 5px
}
.info-box-number {
  margin-bottom: 0
}
.info-box {
  min-height: 30px;
  height: 50px
}
.title {
  padding: 10px;
  padding-left: 20px;
  color: black;
  font-size: 24px;
  font-family: "Roboto", sans-serif
}
button {
  margin: 10px;
  font-size: 16px;
  padding: 7px 12px
}
.split-panel {
  min-height: 700px
}
</style>