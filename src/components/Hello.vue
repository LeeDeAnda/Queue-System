<template>

  <div class="hello">
    <div class="student-input">
      <!-- this is a comment -->
      <h1>CLAS Math Science Drop-In</h1>
      <h4>Sign in to get started</h4>
      <p>Enter your first name</p>
      <input type="text" placeholder="First Name" v-model="studentName" />
      <p>Enter your last name</p>
      <input type="text" placeholder="Last Name" v-model="studentLastName" />
      <!-- Use only if collecting Perm -->
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
    <h1 class="studentFinished">Students: please log out when you leave by selecting "Sign out" under Student Action.</h1>

    <h2>Students in queue:</h2>
    <table>
      <thead>
        <tr>
          <th>Time Arrived</th>
          <th>Exit Time</th>
          <th>Student Name</th>
          <!--  <th>Subject</th> -->
          <th>Student Action</th>
          <th>Course</th>
          <th>Status</th>
          <th style="width: 20%">Tutor</th>

          <th>Tutor Actions: Tutor use only!</th>
          <!--  <th>Priority Course</th>  -->
          <!--  <th>Reason</th>  -->
        </tr>
      </thead>
      <tbody class="zebra">
        <tr v-for="student in studentsInQueue" class="zebra">
          <td>{{student.timestamp.format('h:mm A')}}</td>
        <!--  <td>{{student.timestamp2.format('h:mm A')}}</td>  -->
        <!--  if else for the exit time, both lines are needed -->
          <td v-if="student.timeTotal>=1">{{student.timestamp2.format('h:mm A')}}</td>
          <td v-else></td>
          <!--  <td>{{student.timeTotal}}</td>  -->
          <td>{{student.name}}</td>
          <td><button @click="statusEnd(student)" class="ButtonYellow">Sign out</button>
        <!--  <td class="student-actions"><a href="#" @click.prevent="statusEnd(student)">Sign out</a></td>
-->
          <td v-bind:class="{mathColor: student.subject =='Math', physicsColor: student.subject == 'Physics', pstatColor: student.subject == 'PStat', eceColor: student.subject == 'ECE', astroColor: student.subject =='Astro', otherColor: student.subject == 'Other'}">{{student.course}}</td>
          <!--<td v-bind:class="{student.subject +'Color'}">{{student.course}}</td> -->
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
            <!--  <a href="#" @click.prevent="statusDone(student)">Remove Student</a>  -->
            <button @click="statusDone(student)" class="Button">Tutor Use Only!</button>
          <!--<a href="#" @click.prevent="statusStudying(student)">Studying</a>  -->
          <!--  <a href="#" @click.prevent="statusDone(student)">Finished</a>  -->
          </td>
          <!--  <td><template v-if="student.priority">
            <a>Priority Course</a>
          </template></td>  -->
          <!--
          <td>{{student.reason}}</td>
          -->
        </tr>
      </tbody>
    </table>

    <h2 v-if="studentName =='demo'" v-bind:class="{tableColor: selectedSubject !=='Other'}">Students completed:</h2>
    <table v-bind:class="{tableColor: selectedSubject !=='Other'}">
      <thead>
        <tr v-if="studentLastName =='demo'">
          <th>Time Arrived</th>
          <th>Time Ended</th>
          <th>Duration (Min)</th>
          <th>Contacts</th>
          <th>Student Name</th>
          <th>Last</th>
          <th>Perm</th>
        <!--  <th>Subject</th>  -->
          <th>Course</th>

          <th>Reason</th>

          <th>Tutor</th>
          <th>Priority Course</th>
          <!--  <th>Status</th>  -->
        </tr>
      </thead>
      <tbody>
        <!-- Only use if Perm is collected -->
        <tr v-if="studentPerm =='Do not write code here'" v-for="student in studentsCompleted">
        <!--<tr v-if="studentLastName == 'demo'" v-for="student in studentsCompleted">  -->
          <td>{{student.timestamp.format('MMM D,  h:mm')}}</td>
          <td>{{student.endingTimestamp.format('h:mm A')}}</td>
          <td>{{student.totalTime}}</td>
          <td>{{student.contacts}}</td>
          <td>{{student.name}}</td>
          <td>{{student.lastname}}</td>
          <td :class="{permColor: selectedSubject =='Other'}">{{student.perm}}</td>
          <!--  <td>{{student.subject}}</td>  -->
          <td>{{student.course}}</td>

          <td>{{student.reason}}</td>

          <td>{{student.tutor}}</td>
          <td>
            <template v-if="student.priority">
              <a>Priority Course</a>
            </template>
          </td>
          <!-- keep if string <td>{{student.priority}}</td>  -->
          <!--  <td>{{student.status}}</td>  -->
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
      subjects: ['Math', 'Physics', 'PStat', "ECE", "Astro", "Other"],
      courses: {
        'Math': ['Math 3A', 'Math 3B', 'Math 4A', 'Math 4B', 'Math 6A', 'Math 6B', 'Math 34A', 'Math 34B'],
        'Physics': ['Physics 1', 'Physics 2', 'Physics 3', 'Physics 4', 'Physics 5', 'Physics 6A', 'Physics 6B', 'Physics 6C'],
        'PStat': ['PStat 5A', 'PStat 109'],
        'ECE': ['ECE 10A', 'ECE 10B', 'ECE 10C'],
        'Astro': ['Astro 1'],
        'Other':['Engr 3'],
      },
      reasons:['1 - Need help getting started.','2 - Started but got stuck.', '3 - Can do others but not this one.', '4 - I did problem already and just need someone to check my answer.', '5 - I want to learn concepts better, no specific question.', '6 - I want to review for a test.', '7 - I want to work with others in my class.', '8 - Other'],
      activeTutors: [
        {
          tutorName: '',
          subjects: []
        },
        {
          tutorName: 'Alex P',
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
        },
        {
          tutorName: 'Abigail',
          subjects: ['Math 3A']
        },
        {
          tutorName: 'Alex B',
          subjects: ['Math 3A']
        },
        {
          tutorName: 'Avan',
          subjects: ['Math 3A']
        },
        {
          tutorName: 'Blake',
          subjects: ['Math 3A']
        },
        {
          tutorName: 'Bobby',
          subjects: ['Math 3A']
        },
        {
          tutorName: 'Christian',
          subjects: ['Math 3A']
        },
        {
          tutorName: 'Clara',
          subjects: ['Math 3A']
        },
        {
          tutorName: 'Claude',
          subjects: ['Math 3A']
        },
        {
          tutorName: 'Daniel',
          subjects: ['Math 3A']
        },
        {
          tutorName: 'Danielle',
          subjects: ['Math 3A']
        },
        {
          tutorName: 'EriKa',
          subjects: ['Math 3A']
        },
        {
          tutorName: 'George',
          subjects: ['Math 3A']
        },
        {
          tutorName: 'Himangshu',
          subjects: ['Math 3A']
        },
        {
          tutorName: 'Ian',
          subjects: ['Math 3A']
        },
        {
          tutorName: 'Isaac',
          subjects: ['Math 3A']
        },
        {
          tutorName: 'Jared P',
          subjects: ['Math 3A']
        },
        {
          tutorName: 'Jaret D',
          subjects: ['Math 3A']
        },
        {
          tutorName: 'Jason',
          subjects: ['Math 3A']
        },
        {
          tutorName: 'JJ',
          subjects: ['Math 3A']
        },
        {
          tutorName: 'Jordan',
          subjects: ['Math 3A']
        },
        {
          tutorName: 'Jose',
          subjects: ['Math 3A']
        },
        {
          tutorName: 'Joyce',
          subjects: ['Math 3A']
        },
        {
          tutorName: 'Ken W',
          subjects: ['Math 3A']
        },
        {
          tutorName: 'Kenny L',
          subjects: ['Math 3A']
        },
        {
          tutorName: 'Michael',
          subjects: ['Math 3A']
        },
        {
          tutorName: 'Monica',
          subjects: ['Math 3A']
        },
        {
          tutorName: 'Noah',
          subjects: ['Math 3A']
        },
        {
          tutorName: 'Russell',
          subjects: ['Math 3A']
        },
        {
          tutorName: 'Sam A',
          subjects: ['Math 3A']
        },
        {
          tutorName: 'Sam O',
          subjects: ['Math 3A']
        },
        {
          tutorName: 'Shaun',
          subjects: ['Math 3A']
        },
        {
          tutorName: 'Talah',
          subjects: ['Math 3A']
        },
        {
          tutorName: 'Will',
          subjects: ['Math 3A']
        },
        {
          tutorName: 'Zheng',
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
        //.substring(start location, end location)
        reason: this.selectedReason.substring(0,1),

        // we initialize tutor as an empty string, to be set later
        tutor: '',
        // we also want a boolean to indicate whether we are editing the tutor
        // so we can update the UI accordingly (show dropdown)
        editingTutor: false,
        timestamp: new moment(),
        timestamp2: new moment(),

        status: 'Waiting',
        priority: ''
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
      //Use if Perm is being collected
      if(this.studentPerm.length !== 7) {
      //  window.alert('Please entera valid perm!');
      return 'Please enter a valid perm #'
      }
      if(this.selectedSubject <= 1) {
        return 'No subject selected'
      }
      if(this.selectedCourse <= 1) {
        return 'No course selected'
      }
      if(this.selectedReason <= 1){
        return 'Please select a reason for this visit.'
      }
      return true
    },
    statusPriority(student) {
      student.status = 'Priority Course'
      // set to true to toggle dropdown visibility
      student.editingTutor = true
    // keep if string // student.priority = 'Priority Course'
      student.priority = true
    },
    statusStudying(student) {
      student.status = 'Studying'
      student.editingTutor = true
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
    statusEnd(student){
        student.status = 'Gone'
        student.timestamp2 = new moment()
        student.timeTotal = student.timestamp2.diff(student.timestamp, 'minutes')
        student.contacts = student.timeTotal
        student.contacts /= 30
        //var date = new moment().format('h:mm A')
        //window.alert(date)
      },

    statusDone(student) {
      student.status = 'done'
      // loop through students and find the matching object
      //Use if Perm is being collected
      var completedStudent = _.find(this.studentsInQueue, {perm: student.perm})
      //var completedStudent = _.find(this.studentsInQueue, {lastname: student.lastname})
      // remove them from studentsInQueue array
      //Use if Perm is being collected
      _.remove(this.studentsInQueue, {perm: completedStudent.perm})
      //_.remove(this.studentsInQueue, {lastname: completedStudent.lastname})
      // add additional ending timestamp and duration
      completedStudent.endingTimestamp = new moment()
      completedStudent.totalTime = completedStudent.endingTimestamp.diff(completedStudent.timestamp, 'minutes')
      completedStudent.contacts = completedStudent.totalTime
      completedStudent.contacts /= 30

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
          student.timestamp2 = new moment(student.timestamp2)
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
  .waitingColor{
    background-color: green;
  }
  .mathColor{
    background-color: #3399FF;
  }
  .physicsColor{
    background-color: orange;
  }
  .pstatColor{
    background-color: #FF3333;
  }
  .eceColor{
    background-color: #FFFF00;
  }
  .astroColor{
    background-color: #FF00FF;
  }
  .otherColor{
    background-color: #784AFF;
  }
  .tableColor{
    color: white;
    background-color: white;
  }
  .permColor{
    background-color: white;
  }
  .Button{
    width: 120px;
    background: #ff3300;
    align-content: center;
  }
  .ButtonYellow{
    width: 75px;
    background: yellow;
    align-content: center;
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
  .zebra tr:nth-of-type(odd){
    background-color: #ccc;
  }
  .studentFinished{
    text-align: center;
  }

  table {
    width: 100%;
    font-size: 18px;
  }
  /*tr:nth-of-type(even){
    background-color: #ccc;
  }*/
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
