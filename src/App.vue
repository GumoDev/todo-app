<script setup>

import iconMoon from './assets/icon-moon.svg'
import iconSun from './assets/icon-sun.svg'
import mobileDark from './assets/bg-mobile-dark.jpg'
import mobileLight from './assets/bg-mobile-light.jpg'
import desktopLight from './assets/bg-desktop-light.jpg'
import desktopDark from './assets/bg-desktop-dark.jpg'

import { ref, reactive, computed, onMounted } from 'vue'

let taskList = reactive([])

let newTask = ref('')

const leftN = computed(() => {
    return taskList.filter(el => el.done == false).length
})

const theme = () => {
    darkmode.value = !darkmode.value

    localStorage.setItem('theme', JSON.stringify(darkmode.value))
}

onMounted(()=>{
    let list = document.querySelector('.task-list')
    
    new Sortable(list,{
        animation: 400
    })
})


const darkmode = ref(JSON.parse(localStorage.getItem('theme')))

const left = ref(leftN)

const filter = ref(true)


function addTask() {

    if (newTask.value.length > 0 && !taskList.find(el => el.todo == newTask.value)) {
        filterAll()

        taskList.push({
            todo: newTask.value,
            done: false
        })

    } else {
        alert('Please, Write a new todo')
    }

    newTask = ref('')

    localStorage.setItem('tl', JSON.stringify(taskList))



}

const checkThis = (task) => {
    task.done = !task.done
    localStorage.setItem('tl', JSON.stringify(taskList))
}

const updateThis = (task) => {
    console.log(
        event.target.value
    );

    task.todo = event.target.value

    localStorage.setItem('tl', JSON.stringify(taskList))
}

const deleteThis = (task, index) => {
    taskList.splice(index, 1)
    filterAll()
    localStorage.setItem('tl', JSON.stringify(taskList))


}

const filterAll = (filter) => {

    filters.forEach(el => {
        if (el == filter) {
            el.active = true
        } else {
            if (el !== filter) {
                el.active = false
            }
        }
    })
    document.querySelectorAll('.task').forEach(el => { el.classList.remove('hidden') })
}

const filterActive = (filter) => {

    filters.forEach(el => {
        if (el == filter) {
            el.active = true
        } else {
            el.active = false
        }
    })
    document.querySelectorAll('.done').forEach(el => { el.classList.add('hidden') })
    document.querySelectorAll('.pending').forEach(el => { el.classList.remove('hidden') })
}

const filterComplete = (filter) => {

    filters.forEach(el => {
        if (el == filter) {
            el.active = true
        } else {
            el.active = false
        }
    })
    document.querySelectorAll('.pending').forEach(el => { el.classList.add('hidden') })
    document.querySelectorAll('.done').forEach(el => { el.classList.remove('hidden') })
}

let filters = reactive([
    { text: 'All', action: filterAll, active: false },
    { text: 'Active', action: filterActive, active: false },
    { text: 'Completed', action: filterComplete, active: false }
])

const clearComplete = () => {

    for (let index = 0; index < taskList.length; index++) {
        const element = taskList[index]


        let initCut = taskList.findIndex(el => el.done == true)

        if (element.done == true) {
            taskList.splice(initCut, 1)
            localStorage.setItem('tl', JSON.stringify(taskList))
        }
    }

    filterAll()

}

if (!localStorage.getItem('tl')) {
    localStorage.setItem('tl', JSON.stringify([]))
} else {
    taskList = reactive(JSON.parse(localStorage.getItem('tl')))
    console.log(JSON.parse(localStorage.getItem('tl')))
}

</script>

<template>
    <div :class="darkmode ? 'dark bg-slate-900' : 'bg-[#f2f2f2]'" class="relative w-screen h-screen mx-auto wrapper">
        <!-- header - darkmode -->
        <figure class="flex justify-center w-full">
            <div class="absolute top-10 flex justify-between items-center w-11/12 min-w-80 md:w-[540px]">
                <h1 class="text-xl ml-2 md:py-10 md:text-4xl uppercase tracking-[10px] font-semibold text-white">Todo</h1>
                <img @click="theme" class="w-5 h-5 mr-3 md:h-6 md:w-6" :src="darkmode ? iconSun : iconMoon" alt="">
            </div>
            <img class="w-full md:hidden" :src="darkmode ? mobileDark : mobileLight " alt="">
            <img class="hidden mx-auto md:flex" :src="darkmode ? desktopDark : desktopLight" alt="">
        </figure>

        <!-- form new task -->
        <div class="absolute flex items-center justify-center w-full -left-3 top-24 md:top-36">
            <div @click="addTask"
                class="relative flex items-center justify-center w-6 h-6 pb-1 border rounded-full circle border-black/20 left-9 dark:border-blue-500">✒</div>
            <input @keyup.enter=addTask() v-model="newTask"
                class="dark:bg-[#25273c] dark:text-white/60 rounded w-11/12 md:w-[540px] md:my-10 h-12 md:h-16 pl-12 pr-5"
                type="text" name="" id="" placeholder="write a new todo">
        </div>
        <!-- task list -->
        <ul
            :list="task"
            class="task-list absolute top-[176px] md:top-[190px] left-1/2 -translate-x-1/2 w-11/12 md:w-[540px] min-h-[368px] md:min-h-[439px] h-[55vh] overflow-y-scroll backdrop-blur-sm md:mt-20">

            <!-- task -->

            <li :class="task.done ? 'done' : 'pending'" v-for="(task, index) in taskList" :key="task"
                class="relative flex items-center my-1 border-b task border-black/20 ">
                
                
                    <figure @click="checkThis(task, index)"
                    class="absolute flex items-center justify-center w-6 h-6 transition duration-700 -translate-y-1/2 border border-blue-300 rounded-full tra from-indigo-500 via-purple-400 to-blue-500 left-3 top-6 md:top-8">
                    <img src="./assets/icon-check.svg" :class="task.done ? 'bg-gradient-to-r w-full h-full rounded-full p-1' : 'bg-none w-0'"
                        class="z-20 transition duration-700" />
                </figure>
                <input @keyup="updateThis(task, index)" :class="task.done ? 'text-black/50 line-through bg-white/30 dark:text-white/30' : 'text-black'"
                    :value="task.todo"
                    class="dark:bg-[#25273c] dark:text-white/60 rounded w-full md:w-[540px] h-12 md:h-16 pl-12  pr-3 flex items-center justify-center active:outline-none focus:outline-none"
                    type="text">
                <img @click="deleteThis(task, index)" src="./assets/icon-cross.svg" alt=""
                    class="absolute right-3 top-4 md:top-6">
                
                    
            </li>

            <!-- pending-tasks  clear-complete -->
            <ul
                
                class="sticky bottom-0 flex items-center justify-between w-full mx-auto h-12 px-4 text-xs bg-white dark:bg-[#25273c] dark:text-white/60 left-completed">
                <span> {{ left }} items left</span>
    
    
                <div class="dark:bg-[#25273c] dark:text-white/60 items-center justify-center hidden h-10 bg-white rounded filters md:flex">
                    <button :class="filter.active ? 'bg-purple-500 text-white p-2 rounded border dark:bg-blue-500' : 'border p-2 rounded'"
                        class="mx-3 transition duration-1000 bg-none text-md" v-for="(filter, index) in filters"
                        :key="index" @click=filter.action(filter)>
                        {{ filter.text }}
                    </button>
                </div>
    
                <button @click="clearComplete">clear complete</button>
            </ul>
        </ul>

    </div>

    <div
        :class="darkmode ? 'bg-[#25273c] text-white/60' : 'bg-white'"
        class="absolute flex items-center justify-center w-11/12 h-12 -translate-x-1/2 rounded shadow-lg filters md:hidden bottom-10 left-1/2">
        <button :class="filter.active ? 'bg-purple-500 p-2 rounded border text-white' : 'border p-2 rounded'"
            class="mx-3 transition duration-1000 bg-none text-md" v-for="(filter, index) in filters" :key="index"
            @click=filter.action(filter)>
            {{ filter.text }}
        </button>
    </div>

    <!-- drag and drop hint -->
    <span 
        :class="darkmode ? 'text-white/50 ' : ''"
        class="absolute w-full text-center -translate-x-1/2 bottom-3 left-1/2 text-black/40">
        Drag and drop to reorder list
    </span>



    <div class="absolute text-xs attribution text-white/50">
        Challenge by <a class="underline" href="https://www.frontendmentor.io?ref=challenge" target="_blank">Frontend
            Mentor</a>.
        Coded by <a class="underline" href="https://www.frontendmentor.io/profile/GumoDev">GumoDev</a>.
    </div>
</template>

<style>

#app {
    display: flex;
    justify-content: center;
}

html {
    display: flex;
    width: 100vw
}


.filter-active {
    color: blue;
}

</style>
