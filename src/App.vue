<script setup>
  import {ref, watch, onMounted, nextTick } from "vue";

  const showModal = ref(false)
  const newNote = ref("")
  const notes = ref([])
  const errorMsg = ref("")
  const textarea = ref(null)

  // Get Random Color
  function getRandomColor() {
    return "hsl(" + Math.random() * 360 + ", 100%, 75%)";
  }
  
  // Get Date
  function formatDate(date) {
    var hours = date.getHours();
    var minutes = date.getMinutes();
    var ampm = hours >= 12 ? 'pm' : 'am';
    hours = hours % 12;
    hours = hours ? hours : 12; // the hour '0' should be '12'
    minutes = minutes < 10 ? '0'+minutes : minutes;
    var strTime = hours + ':' + minutes + ' ' + ampm;
    return (date.getDate() + "/" + date.getMonth()+1) + "/" + date.getFullYear() + "  " + strTime;
  }


  // Add Note
  const addNote = () => {
    if (newNote.value.length < 5){
      errorMsg.value = "Text should be minimum of 5 characters"
      setTimeout(() => {
        errorMsg.value = false;
      }, 3000);
      return
    }
    notes.value.push({
      id: Math.floor(Math.random() * 1000000),
      text: newNote.value,
      time: formatDate(new Date()),
      backgroundColor: getRandomColor()
    })

    showModal.value = false
    newNote.value = ""
    errorMsg.value = ""
    
  }

  // Open Modal on Btn Click
  const openModal = () => {
    showModal.value = true
    nextTick(() => {
        textarea.value.focus();
    });
  }

  const deleteCard = (id) => {
    notes.value = notes.value.filter( note => note.id !== id )
  }
  
  // Saving Notes to LocalStorage
  watch(notes, (newVal) => {
    localStorage.setItem("notes", JSON.stringify(newVal))
  }, {deep:true})

  onMounted(() => {
    // Getting Notes from LS
    notes.value = JSON.parse(localStorage.getItem("notes")) || []
    
  })
  
</script>

<template>
  <main>
    <!-- Modal -->
    <div v-show="showModal" class="overlay">
      <div class="modal">
        <textarea @keypress.enter="addNote" v-model.trim="newNote" ref="textarea" placeholder="What's on your mind?" name="note" cols="30" rows="5"></textarea>
        
        <transition name="shake">
          <p v-if="errorMsg">{{ errorMsg }}</p>
        </transition>
        
        <button @click="addNote">Pin iT</button>
        <button class="close" @click="showModal=false">Close</button>
      </div>
    </div>

    <!-- Main Page -->
    <div class="container">

      <!-- Header Section -->
      <header  @dblclick="openModal">
        <h1>Pin iT</h1>
        <button @click="openModal">+</button>
      </header>

      <!-- Cards Section -->
      <transition name="switch" mode="out-in">
   
      <div v-if="notes.length" class="cards-container">
        <transition-group name="card" appear>
        <div
          v-for="note in notes" 
          :key="note.id" 
          :style="{backgroundColor:note.backgroundColor}" 
          @dblclick="deleteCard(note.id)"
          class="card">
          <p class="main-text">{{ note.text }}</p>
          <p class="date">{{note.time}}</p>
        </div>
      </transition-group>
      </div>

      <div class="msg" v-else>
          <h1>Let's Pin</h1>
          <div class="tut">
            <h3>TIP</h3>
            <p>Double click to delete cards</p>
          </div>
      </div>
         
    </transition>
    </div>
  </main>
</template>

<style scoped>
 
  main{
    margin:0;
    padding:0;
    box-sizing: border-box;
    width: 100%;
    min-height: 100vh;
    background: linear-gradient(azure, rgb(216, 252, 252)) ;
    color: rgb(44, 44, 44);

  }
  
  .container,section{
    max-width: 1000px;
    padding: 1rem;
    margin: 0 auto;

  }
  section input{
    border:none;
    outline:none;
    background: transparent;
    font-size: 1.4rem;
  }
  header{
    position: sticky;
    top: 0;
    display: flex;
    justify-content: space-between;
    align-items: center;
    background-color: darkslategray;
    border-radius: 5px;
    margin: 0 1rem;
    padding: 0 2rem;
    user-select: none;
    z-index: 10;
  }
  h1{
    font-family: Verdana, Geneva, Tahoma, sans-serif;
    font-weight: bold;
    font-size: 2rem;
    letter-spacing: -1px;
    color: azure;
  }
  header button{
    border: none;
    margin: 1rem;
    width: 40px;
    height: 40px;
    cursor: pointer;
    background-color: azure;
    color: darkslategray;
    font-weight: 500;
    font-size: 2.1rem;
    border-radius: 100%;
    transition: all .2s ease-in-out;

  }
  .msg{
    text-align: center;
    margin-top: 1rem;
    
  }
  .msg h1{
    font-size: 3rem;
    user-select: none;
    color:rgba(61, 141, 137, 0.512);
  }
  .tut{
    margin: 10% auto;
    color:rgba(61, 141, 137, 0.512);
  }
  .tut h3{
    font-weight: bold;
  }
 
  .cards-container{
    display:flex;
    justify-content: center;
    align-items: center;
    flex-wrap: wrap;
    margin: 2rem 1rem;
  }
  .card{
    width: 225px;
    min-height: 225px;
    background-color: antiquewhite;
    color: rgb(30, 30, 30);
    padding: 10px;
    border-radius: 15px;
    display: flex;
    flex-direction: column;
    justify-content: space-between;
    margin: 1rem 1.5rem 1rem 0;
    font-weight: 600;
    border: 2px solid rgba(0, 0, 0, 0);
    white-space: pre-wrap;
    white-space: -moz-pre-wrap;
    white-space: -pre-wrap;
    white-space: -o-pre-wrap;
    word-wrap: break-word;
    transition: all .1s ease-in-out
  }
  .card:hover{
    border: 2px solid rgba(36, 38, 38, 0.525);
    transform: rotate(5deg);
  }
  .main-text{
    font-size: 1.2rem;
  }
  .date{
    font-size: 1rem;
    font-family: monospace;
    font-weight: 500;
    user-select: none;
    text-align: center;
    color: rgba(29, 29, 29, 0.933);
  }
  .overlay{
    position: absolute;
    width: 100%;
    min-height: 100%;
    background-color: rgba(0, 0, 0, .77);
    z-index: 1000;
    display:flex;
    align-items: center;
    justify-content: center;
  }

  .modal{
    width: 750px;
    background-color: azure;
    padding: 2rem;
    position: relative;
    display:flex;
    flex-direction: column;
    border-radius: 6px;
  }
  .modal p{
    color:rgba(239, 29, 29, 0.867);
    font-weight: bold;
    text-align: center;
    margin: .5rem 0 0rem;
    letter-spacing: 1px;
  }
  .modal button{
    border: none;
    padding: 1rem .5rem;
    margin: 1rem 0;
    border-radius: 6px;
    cursor: pointer;
    background-color: darkslategray;
    color: azure;
    font-weight: 600;
    font-size: 2.1rem;
    transition: all .2s ease-in-out;
  }
  button:hover{
    background-color: darkslategray;
    color:rgb(244, 252, 252)
  }
  button {
    -webkit-tap-highlight-color: transparent; /* remove tap highlight */
  }
  button:focus {
      outline: none; /* remove outline */
      box-shadow: none; /* remove box shadow */
  }
  textarea{
    resize: none;
    border: 1px solid darkslategray;
    border-radius: 6px;
    padding: 1rem;
    font-size: 1.3rem;
    font-family: monospace;
    margin-top: 1rem;
  }
  textarea:focus{
    outline: 1px solid darkslategray;
  }
  .modal .close{
    position: absolute;
    top:0;
    right: 0;
    background:none;
    padding: 10px;
    margin: 10px;
    cursor: pointer;
    color: rgba(47, 79, 79, 0.681);
    font-weight: 600;
    font-size: 1rem;
  }
  .modal .close:hover{
    color:darkslategray
  }

  /* Animations */
  .fade-enter-from{
    opacity:0;
    transform: translateY(-60px)
  }
  .fade-enter-active{
    transition:all .5s ease
  }
  .shake-enter-active{
    animation: shake .3s ease
  }
  @keyframes shake {
    0%{ transform: translateY(60px); opacity:0 }
    50%{ transform: translateY(0px); opacity:1 }
    60%{ transform: translateX(8px) }
    70%{ transform: translateX(-8px) }
    80%{ transform: translateX(4px) }
    90%{ transform: translateX(-4px) }
    100%{ transform: translateX(0px) }
  }
  .shake-leave-to{
    opacity:0;
    transform: translateY(10px)
  }
  .shake-leave-active{
    transition:all .5s ease
  }
  /* Card Animations */
  .card-enter-from{
    opacity:0;
    transform:scale(0.6)
  }
  .card-enter-active{
    transition:all .4s ease
  }
  .card-leave-to{
    opacity:0;
    transform: scale(.6)
  }
  .card-leave-active{
    transition: all .4s ease;
    position:absolute;
  }
  .card-move{
    transition: all .4s ease;
  }
  .switch-enter-from,
  .switch-leave-to{
    opacity:0;
    transition: translateY(20px);
  }
  .switch-enter-to,
  .switch-leave-from{
    opacity:1;
    transition: translateY(0);
  }
  .switch-enter-active,
  .switch-leave-active{
    transition:all .3s ease
  }

  /* Media Query */
  @media screen and (max-width:768px) {
    .container{
      width:100%;
      padding:0;
    }
    header{
      width: 100%;
      border-radius: 0;
      margin:0;
    }
    .modal {
      width:90%;
    }
    .card{
      width: 150px;
      min-height: 150px;
      font-size: 1rem;
    }
    .date{
      font-size: .6rem;
    }
  }
</style>