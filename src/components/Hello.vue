<template>
  <div class="hello">
    <div class="student-input">
      <!-- this is a comment -->
      <h4>Sign in to get started</h4>
      <p>Enter your first name</p>
      <input type="text" placeholder="First Name" v-model="studentName" />
      <p>Enter your last name</p>
      <input type="text" placeholder="Last Name" v-model="studentLastName" />
      <p>Enter your perm #</p>
      <input type="text" placeholder="perm #" v-model="studentPerm" />
      <p>Select Subject</p>
      <select v-model="selectedSubject">
        <option v-for="hilee in subjects">{{hilee}}</option>
      </select>

      <div v-if="selectedSubject">
        <p>Select Course</p>
        <select v-model="selectedCourse">
          <option v-for="course in availableCourses">{{course}}</option>
        </select>
      </div>

      <div v-if="selectedCourse">
        <p>Select Reason</p>
        <select v-model="selectedReason">
          <option v-for="reason in reasons">{{reason}}</option>
        </select>
      </div>

      <div>
        <button @click="enterQueue" class="queueButton">Enter Queue</button>
        <p v-if="errors" class="thisisawarning">
          {{errorMessage}}
        </p>
      </div>
    </div>

    <h2>Students in queue:</h2>
    <table>
      <thead>
        <tr>
          <th>Time Arrived</th>
          <th>Student Name</th>
          <!--  <th>Subject</th>  -->
          <th>Course</th>

          <th>Status</th>
          <th style="width: 20%">Tutor</th>
          <th>Action</th>
          <th>Reason</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="student in studentsInQueue">
          <td>{{student.timestamp.format('h:mm:ssA')}}</td>
          <td>{{student.name}}</td>
          <!--  <td>{{student.subject}}</td>  -->
          <td>{{student.course}}</td>

          <td>{{student.status}}</td>
          <td>
            <template v-if="student.editingTutor">
              <select v-model="student.tutor">
                <option v-for="tutor in activeTutors">{{tutor.tutorName}}</option>
              </select>
              <a href="#" @click.prevent="closeTutorSelection(student)">Close</a>
            </template>
            <template v-else>
              {{student.tutor}}
            </template>
          </td> <!-- don't forget to always close your html tags: <td></td> -->
          <td class="student-actions">
            <a href="#" @click.prevent="statusPriority(student)">Priority Course</a>
            <a href="#" @click.prevent="statusWaiting(student)">Waiting</a>
            <a href="#" @click.prevent="statusTutor(student)">Working with</a>
            <a href="#" @click.prevent="statusStudying(student)">Studying</a>
            <a href="#" @click.prevent="statusDone(student)">Done</a>
          </td>
          <td>{{student.reason}}</td>
        </tr>
      </tbody>
    </table>

    <h2>Students completed:</h2>
    <table>
      <thead>
        <tr>
          <th>Time Arrived</th>
          <th>Time Ended</th>
          <th>Duration</th>
          <th>Student Name</th>
          <th>Last</th>
          <th>Perm</th>
          <th>Subject</th>
          <th>Course</th>
          <th>Reason</th>
          <th>Tutor</th>
          <th>Status</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="student in studentsCompleted">
          <td>{{student.timestamp.format('MMM D,  h:mm A')}}</td>
          <td>{{student.endingTimestamp.format('h:mm:ss A')}}</td>
          <td>{{student.totalTime}} minutes</td>
          <td>{{student.name}}</td>
          <td>{{student.lastname}}</td>
          <td>{{student.perm}}</td>
          <td>{{student.subject}}</td>
          <td>{{student.course}}</td>
          <td>{{student.reason}}</td>
          <td>{{student.tutor}}</td>
          <td>{{student.status}}</td>
        </tr>
      </tbody>
    </table>

  </div>
</template>

<script>
import moment from 'moment'
import _ from 'lodash'
export default {
  name: 'hello',
  data () {
    return {
      errors: false,
      errorMessage: '',
      studentName: '',
      studentLastName: '',
      studentPerm: '',
      selectedSubject: '',
      selectedCourse: '',
      selectedReason: '',
      studentsInQueue: [],
      studentsCompleted: [],
      subjects: ['Math', 'Physics', 'PStat', "Other"],
      courses: {
        'Math': ['Math 3A', 'Math 3B', 'Math 4A', 'Math 4B', 'Math 6A', 'Math 6B', 'Math 34A', 'Math 34B'],
        'Physics': ['Physics 1', 'Physics 2', 'Physics 3', 'Physics 4', 'Physics 5', 'Physics 6A', 'Physics 6B', 'Physics 6C'],
        'PStat': ['PStat 5A', 'PStat 109'],
        'Other': ['ECE 10A', 'ECE 10B', 'ECE 10C', 'Engr 3', 'Astro 1'],
      },
      reasons:['1 - Need help getting started.','2 - Started but got stuck.', '3 - Can do others but not this one', '4 - I did problem already and just need someone to check my answer', '5 - I want to learn concepts better, no specific question.', '6 - Other'],
      activeTutors: [
        {
          tutorName: '',
          subjects: []
        },
        {
          tutorName: 'Alex',
          subjects: ['Math 3A', 'Math 34']
        },
        {
          tutorName: 'Binh',
          subjects: ['Math 3A', 'Math 34']
        },
        {
          tutorName: 'Damien',
          subjects: ['Math 3A']
        },
        {
          tutorName: 'Lee',
          subjects: ['Math 3A']
        },
        {
          tutorName: 'Loren',
          subjects: ['Math 3A']
        },
        {
          tutorName: 'Vince',
          subjects: ['Math 3A']
        }

      ]
    }
  },
  computed: {
    availableCourses() {
      return this.courses[this.selectedSubject]
    },
    sortedStudentsInQueue() {
      // return the students in queue, except sorted by students that arent studying
    }
  },
  methods: {
    enterQueue() {
      var isValid = this.validateInput()
      console.log(isValid)
      if(isValid !== true) {
        this.errors = true
        this.errorMessage = isValid
        return false
      }
      this.errors = false

      // initialize the object that we will place into the queue
      var student = {
        name: this.studentName,
        lastname: this.studentLastName,
        perm: this.studentPerm,
        subject: this.selectedSubject,
        course: this.selectedCourse,
        reason: this.selectedReason,
        // we initialize tutor as an empty string, to be set later
        tutor: '',
        // we also want a boolean to indicate whether we are editing the tutor
        // so we can update the UI accordingly (show dropdown)
        editingTutor: false,
        timestamp: new moment(),
        status: 'Waiting'
      }
      // push new object into studentsInQueue array
      this.studentsInQueue.push(student)
      // clear inputs
      this.studentName = ''
      this.studentLastName = ''
      this.studentPerm = ''
      this.selectedCourse = ''
      this.selectedSubject = ''
      this.selectedReason =''

      this.saveDataToStorage()
    },
    validateInput() {
      this.errorMessage = ''
      if(this.studentName.length <= 1) {
        return 'No first name entered'
      }
      if(this.studentLastName.length <= 1) {
        return 'No last name entered'
      }
      if(this.studentPerm.length !== 7) {
        window.alert('Please entera valid perm!');
        return 'Please enter a valid perm #'

      }
      if(this.selectedSubject.length <= 1) {
        return 'No subject selected'
      }
      if(this.selectedCourse <= 1) {
        return 'No course selected'
      }
      return true
    },
    statusPriority(student) {
      student.status = 'Priority Course'
      // set to true to toggle dropdown visibility
      student.editingTutor = true
    },
    statusStudying(student) {
      student.status = 'studying'
    },
    statusTutor(student) {
      student.status = 'Working with'
      // set to true to toggle dropdown visibility
      student.editingTutor = true
    },
    closeTutorSelection(student) {
      student.editingTutor = false
    },
    statusWaiting(student) {
      student.status = 'Waiting'
    },
    statusDone(student) {
      student.status = 'done'
      // loop through students and find the matching object
      var completedStudent = _.find(this.studentsInQueue, {perm: student.perm})
      // remove them from studentsInQueue array
      _.remove(this.studentsInQueue, {perm: completedStudent.perm})
      // add additional ending timestamp and duration
      completedStudent.endingTimestamp = new moment()
      completedStudent.totalTime = completedStudent.endingTimestamp.diff(completedStudent.timestamp, 'minutes')
      // add them to studentsCompleted array
      this.studentsCompleted.push(completedStudent)
      this.saveDataToStorage()
    },
    loadDataFromStorage() {
      // load queued students
      var inqueue = window.localStorage.getItem('studentsInQueue')
      if(inqueue !== null) {
        this.studentsInQueue = JSON.parse(window.localStorage.getItem('studentsInQueue'))
        _.forEach(this.studentsInQueue, (student) => {
          student.timestamp = new moment(student.timestamp)
        })
      } else {
        this.studentsInQueue = []
      }

      // load completed students
      var completed = window.localStorage.getItem('studentsCompleted')
      if(completed !== null) {
        this.studentsCompleted = JSON.parse(completed)
        _.forEach(this.studentsCompleted, (student) => {
          student.timestamp = new moment(student.timestamp)
          student.endingTimestamp = new moment(student.endingTimestamp)
        })
      } else {
        this.studentsCompleted = []
      }
    },
    saveDataToStorage() {
      window.localStorage.setItem('studentsInQueue', JSON.stringify(this.studentsInQueue))
      window.localStorage.setItem('studentsCompleted', JSON.stringify(this.studentsCompleted))
    }
  },
  mounted() {
    this.loadDataFromStorage()
  }
}
</script>

<style>
  /* example css comment */
  * {
    box-sizing: border-box;
  }
  .queueButton {
    margin-top: 20px;
    width: 200px;
  }
  .thisisawarning {
    color: red;
    font-size: 14px;
  }
  .student-input {
    background-color: #ccc;
    padding: 5px 20px 20px;
    text-align: center;
  }
  .student-input p {
    margin: 0;
  }
  .student-actions a {
    margin-right: 4px;
  }
  table {
    width: 100%;
  }
  table th {
    text-align: left;
  }
  input {
    width: 200px;
    padding: 4px;
    margin-bottom: 15px;
  }
  input::-webkit-input-placeholder {
    color: #ccc;
  }
  select {
    margin-bottom: 15px;
    width: 200px;
    padding: 4px;
  }
</style>
