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
          <th>Subject</th>
          <th>Course</th>
          <th>Status</th>
          <th>Action</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="student in studentsInQueue">
          <td>{{student.timestamp.format('h:mm:ssA')}}</td>
          <td>{{student.name}}</td>
          <td>{{student.subject}}</td>
          <td>{{student.course}}</td>
          <td>{{student.status}}</td>
          <td class="student-actions">
            <select v-if="false">
              <option v-for="tutor in activeTutors">{{tutor.name}}</option>
            </select>
            <a href="#" @click.prevent="statusPriority(student)">Priority Course</a>
            <a href="#" @click.prevent="statusWaiting(student)">Waiting</a>
            <a href="#" @click.prevent="statusTutor(student)">Being Tutored</a>
            <a href="#" @click.prevent="statusStudying(student)">Studying</a>
            <a href="#" @click.prevent="statusDone(student)">Done</a>
          </td>
        </tr>
      </tbody>
    </table>

  </div>
</template>

<script>
import moment from 'moment'
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
      studentsInQueue: [],
      studentsInProgress: [],
      studentsCompleted: [],
      subjects: ['Math', 'Physics', 'PStat', "Other"],
      courses: {
        'Math': ['Math 3A', 'Math 3B', 'Math 4A', 'Math 4B', 'Math 6A', 'Math 6B', 'Math 34A', 'Math 34B'],
        'Physics': ['Physics 1', 'Physics 2', 'Physics 3', 'Physics 4', 'Physics 5', 'Physics 6A', 'Physics 6B', 'Physics 6C'],
        'PStat': ['PStat 5A', 'PStat 109'],
        'Other': ['ECE 10A', 'ECE 10B', 'ECE 10C', 'Engr 3', 'Astro 1'],
      },
      activeTutors: [
        {
          name: 'Lee',
          subjects: ['Math 23', 'Math 34']
        },
        {
          name: 'Andrew',
          subjects: ['Math 23', 'Math 34']
        },
        {
          name: 'Alex',
          subjects: ['Math 23', 'Math 34']
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
  watch: {
    'studentPerm': 'resetError'
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
        return 'Perm must be 7 digits'
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
    },
    statusStudying(student) {
      student.status = 'studying'
    },
    statusTutor(student) {
      student.status = 'being tutored'
    },
    statusWaiting(student) {
      student.status = 'Waiting'
    },
    statusDone(student) {
      student.status = 'done'
    },
    resetError() {
      this.errors = false
    }
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
