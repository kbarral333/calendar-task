<template>
  <v-app>
    <v-card>
      <v-card-text
       class="text-center"
      >
        <v-date-picker
        v-model="picker"
        full-width
        elevation="5"
        class="p-12"
        />
      </v-card-text>
    </v-card>
    <v-card>
      <v-card-title>Tasks</v-card-title>
      <v-card-text>
        <v-card
         v-for="t in tasksOfTheDay.tasks"
         :key="t.name"
        >
          <v-card-title>{{t.name}}</v-card-title>
          <v-card-text
            class="ml-5"
          >
            <h3>Urgency: {{t.urgency}}
            </h3>
            <br>
            <v-sheet v-if="t.urgency == 'High'">
              <v-row>
                <v-col>
                <h4>Internal Deadline: {{t.internalDeadline}}</h4>   
                </v-col>
              </v-row>
              <v-row>
                <v-col>
                  <h4> Deadline: {{t.clientDeadline}}</h4>
                </v-col>
              </v-row>
            </v-sheet>
          </v-card-text>
          <v-card-actions>
            <v-btn
              @click="editTask(t.name, t.urgency, t.internalDeadline, t.clientDeadline)"
            >
              Edit
            </v-btn>
          </v-card-actions>
        </v-card>
      </v-card-text>
        <v-card-actions>
          <v-btn @click="addTask">Add task</v-btn>
        </v-card-actions>
    </v-card>
    <v-overlay
      :value="isOverlayOn"
      opacity="0.9"
    >
      <v-form ref="form">
        <v-text-field
          type="text"
          label="Task name"
          v-model="task.name"
        />
        <v-select
          label="Priority"
          :items="urgency"
          v-model="task.urgency"
        >
        </v-select>
        <div v-if="task.urgency == 'High'">
          <v-text-field
            type="date"
            label="Internal Deadline"
            v-model="task.internalDeadline"
          />
          <v-text-field
            type="date"
            label="Client Deadline"
            v-model="task.clientDeadline"
          />
        </div>
        <v-btn color="success" @click="addNewTask">Add</v-btn>
      </v-form>
    </v-overlay>
    <v-overlay
      :value="isEditing"
      opacity="0.9"
    >
      <v-form ref="form">
        <v-text-field
          type="text"
          label="Task name"
          v-model="task.name"
        />
        <v-select
          label="Priority"
          :items="urgency"
          v-model="task.urgency"
        >
        </v-select>
        <div v-if="task.urgency == 'High'">
          <v-text-field
            type="date"
            label="Internal Deadline"
            v-model="task.internalDeadline"
          />
          <v-text-field
            type="date"
            label="Client Deadline"
            v-model="task.clientDeadline"
          />
        </div>
        <v-btn color="success" @click="overwriteTask">Confirm</v-btn>
      </v-form>
    </v-overlay>
  </v-app>
</template>

<script>

export default {
  name: 'App',

  components: {
    //
  },
  mounted (){
    var i
    if(localStorage.allTasks){
      var temp, t
      temp = localStorage.getItem('allTasks')
      this.allTasks = JSON.parse(temp)
      for (i in this.allTasks){
        this.existingDates.push(this.allTasks[i].day)
        if(this.allTasks[i].day == this.picker){
          this.tasksOfTheDay.tasks = this.allTasks[i].tasks
          for (t in this.tasksOfTheDay.tasks){
            this.table.push(this.tasksOfTheDay.tasks[t].name)
          }
        }
      }
    }
  },
  watch: {
    picker: function (){
      var i, t
      this.tasksOfTheDay = {}
      this.table = []
      for (i in this.allTasks){
        if(this.allTasks[i].day == this.picker){
          this.tasksOfTheDay.tasks = this.allTasks[i].tasks
          for (t in this.tasksOfTheDay.tasks){
            this.table.push(this.tasksOfTheDay.tasks[t].name)
          }
        } else {
          this.tasksOfTheDay.day = this.picker
        }
      }
    }
  },
  data: () => ({
    picker: (new Date(Date.now() - (new Date()).getTimezoneOffset() * 60000)).toISOString().substr(0, 10),
    allTasks: new Array,
    tasksOfTheDay: {
      day: '',
      tasks: new Array
    },
    task: {
      name: '',
      urgency: '',
      internalDeadline: '',
      clientDeadline: '',
    },
    isOverlayOn: false,
    isEditing: false,
    tempIndex: '',
    table: new Array,
    existingDates: new Array,
    urgency: ['Low', 'Moderate', 'High']
  }),
  methods: {
    addTask(){
      if(!this.tasksOfTheDay.day){
        this.tasksOfTheDay.day = this.picker
      }
      this.isOverlayOn = true
      this.task = {}
    },
    addNewTask(){
      console.log(this.tasksOfTheDay,this.picker)
      try {
        this.tasksOfTheDay.tasks.push(this.task)
      } catch {
        this.tasksOfTheDay.tasks = []
        this.tasksOfTheDay.tasks.push(this.task)
      }
      this.table.push(this.task.name)
      this.task = {}
      this.addToAllTasks()
      this.isOverlayOn = false
    },
    editTask(name, urgency, internalDeadline, clientDeadline){
      var t
      for (t in this.table){
        if (name == this.table[t])
        this.tempIndex = t
      }
      this.task = {}
      this.task.name = name
      this.task.urgency = urgency
      this.task.internalDeadline = internalDeadline
      this.task.clientDeadline = clientDeadline
      this.isEditing = true
    },
    overwriteTask(){
      var num = this.tempIndex
      this.isEditing = false
      this.tasksOfTheDay.tasks[num] = this.task
      this.addToAllTasks()
    },
    addToAllTasks(){
      var d
      var indx = -1
      for(d in this.existingDates){
        if(this.picker == this.existingDates[d]){
          indx = d
        }
      }
      if(indx == -1){
        this.allTasks.push(this.tasksOfTheDay)
        this.existingDates.push(this.picker)
      } else{
        this.allTasks[indx].tasks = this.tasksOfTheDay.tasks
      }
      localStorage.setItem('allTasks',JSON.stringify(this.allTasks))
    }
  }
};
</script>