<script>
    import {FaApple} from "svelte-icons/fa";


    export let isDarkMode = false
    import { onMount } from 'svelte';
    import GiEmptyWoodBucket from 'svelte-icons/gi/GiEmptyWoodBucket.svelte'
    import FaAppleAlt from 'svelte-icons/fa/FaAppleAlt.svelte'
    import FaSun from 'svelte-icons/fa/FaSun.svelte'
    import FaMoon from 'svelte-icons/fa/FaMoon.svelte'
    import GoGear from 'svelte-icons/go/GoGear.svelte'
    import Modal from './Modal.svelte'
    import GiShinyApple from 'svelte-icons/gi/GiShinyApple.svelte'
    import GiEmptyMetalBucket from 'svelte-icons/gi/GiEmptyMetalBucket.svelte'


    let isHoveredRight = false
    let isHoveredLeft = false
    let appleMode = 'normal'
    let bucketMode = 'normal'
    let score = 0;
    let bucketX = 50;
    let appleY = 0;
    let appleX = Math.random() * 90;
    let showLoseModal = (false);
    let showSettingsModal = false
    let bucketSpeed = 3
    let termBucketSpeed = bucketSpeed

    let speed = 0.5;
    let termSpeed = speed// Скорость падения яблока
    let gameInterval;
    // Переменная для отслеживания темы

    // Перемещаем яблоко к верхнему краю после поимки или при падении
    function resetApple() {
        appleY = 0;
        appleX = Math.random() * 100;
        appleX = Math.max(10, Math.min(90, appleX))
    }

    // Обновляем игровой процесс
    function updateGame() {
        appleY += speed;
        // Проверка на попадание в ведро
        if (appleY >= 80) {
            if (Math.abs(appleX - bucketX) < 10){
                score++;
                resetApple();
            }
            else{
                if (appleY >= 90) {
                    showLoseModal = openModal()
                    resetApple()
                }
            }
        }


    }

    // Перемещение ведра влево или вправо
    function moveBucket(direction) {
        bucketX += direction * bucketSpeed;
        bucketX = Math.max(5, Math.min(97, bucketX)); // Ограничение на движение
    }


    function handleKeydown(event) {
        if (event.key === 'ArrowLeft') {
            moveBucket(-1)
            isHoveredLeft = true
        } else if (event.key === 'ArrowRight') {
            moveBucket(1)
            isHoveredRight = true
        }
    }

    function handleKeyup(event){
        if (event.key === 'ArrowLeft') {
            isHoveredLeft = false
        }
        else if (event.key === 'ArrowRight'){
            isHoveredRight = false
        }
    }

    function toggleTheme() {
        isDarkMode = !isDarkMode;
    }


    onMount(() => {
        gameInterval = setInterval(updateGame, 50);
        window.addEventListener('keydown', handleKeydown);
        window.addEventListener('keyup', handleKeyup);
        const mediaQuery = window.matchMedia('(prefers-color-scheme: dark)');
        isDarkMode = mediaQuery.matches;
        const handleChange = (e) => {
            isDarkMode = e.matches;
        };
        mediaQuery.addEventListener('change', handleChange);

        // Очистка слушателя при размонтировании компонента
        return () => {

        };

        return () => {
            clearInterval(gameInterval);
            window.removeEventListener('keydown', handleKeydown);
            window.removeEventListener('keyup', handleKeyup);
            mediaQuery.removeEventListener('change', handleChange);
        };
    });




    function openModal() {
        score=0
        window.removeEventListener('keydown', handleKeydown);
        window.removeEventListener('keyup', handleKeyup);
        speed = 0
        return true
    }

    function closeModal() {
        speed = termSpeed
        bucketSpeed = termBucketSpeed
        window.addEventListener('keydown', handleKeydown);
        window.addEventListener('keyup', handleKeyup);
        return false
    }
   console.log(isHoveredLeft)
</script>


<div class="game-container" class:dark={isDarkMode}>
    <div class="score">Score: {score}</div>
    <div
            class="bucket"
            style="--bucketX: {bucketX}%; bottom: 2%;"
    >
        {#if bucketMode==='normal'}
            <GiEmptyWoodBucket/>
        {:else if bucketMode==='metal'}
            <GiEmptyMetalBucket/>
        {/if}

    </div>
    <div
            class="apple"
            style="--appleY: {appleY}%; --appleX: {appleX}%"
    >
        {#if appleMode==='normal'}
            <FaAppleAlt/>
        {:else}<GiShinyApple/>
        {/if}

    </div>

    <!-- Кнопки управления -->
    <div class="controls">
        <button class:hovered={isHoveredLeft} on:click={()=>moveBucket(-1)}>← Left</button>
        <button class:hovered={isHoveredRight} on:click={() => moveBucket(1)}>Right →</button>
    </div>
    <button class="settings" on:click={()=>showSettingsModal=openModal()}>
        <GoGear/>

    </button>
    <button class="theme-toggle" on:click={toggleTheme}>
        {#if isDarkMode}
            <FaSun/>
        {:else}
            <FaMoon/>
        {/if}

    </button>

    <Modal isOpen={showLoseModal} buttonText="Попробовать снова" on:close={()=>showLoseModal=closeModal()}>
        <h2>Вы проиграли!</h2>
        <p>Вы набрали {score} {score % 10 === 1 ? 'очко' : 'очков'}<br>Попробуй снова, ботик</p>

    </Modal>

    <Modal isOpen={showSettingsModal} buttonText="Закрыть" on:close={()=>showSettingsModal=closeModal()}>
        <h2>Настройки</h2>
        <label>Скорость падения яблока: {termSpeed}</label>
        <input
                type="range"
                list="markers"
                min="1"
                max="5"
                step="1"

                bind:value={termSpeed}
        />
        <label>Скорость движения ведра: {termBucketSpeed}</label>
        <input
                type="range"
                list="markers"
                min="1"
                max="10"
                step="1"

                bind:value={termBucketSpeed}
        />
        <div class="row">
            Выбери стиль яблока:
            <div  style="width: 50px;" class="row-in">
                <div style="width: 30px; height: 20px" class="row-el" on:click={()=>appleMode='normal'}><FaAppleAlt/></div>
                <div style="width: 30px; height: 20px" class="row-el" on:click={()=>appleMode='shiny'}><GiShinyApple/></div>
            </div>
        </div>
        <div class="row">
            Выбери стиль ведра:
            <div  style="width: 60px;" class="row-in">
                <div   class="row-el" on:click={()=>bucketMode='normal'}><GiEmptyWoodBucket/></div>
                <div  class="row-el" on:click={()=>bucketMode='metal'}><GiEmptyMetalBucket/></div>
            </div>
        </div>
    </Modal>

    <datalist id="markers">
        <option value="0"></option>
        <option value="2"></option>
        <option value="4"></option>
        <option value="6"></option>
        <option value="8"></option>
    </datalist>


</div>




<style>

    html, body {
        margin: 0;
        padding: 0;
        overflow: hidden;


    }
    .row{
        display: flex;
        gap: 10px;
    }



    .row-in{
        display: flex;
        gap: 10px;
    }

    .row-el{
        cursor: pointer;
    }

    .row-el:hover{
        color: green;
    }

    /* Основной стиль игры */
    .game-container {
        display: flex;
        flex-direction: column;
        justify-content: center;
        align-items: center;
        margin: 0;
        padding: 0;
        width: 100vw;
        height: 100vh;;
        background-color: var(--background-color);
        font-family: "JetBrains Mono";
        transition: background-color 0.4s;
    }


    .bucket {
        position: absolute;
        font-size: 40px;
        width:11vw;
        height: auto;
        padding: 0;
        margin: 0;
        bottom: 0;
        left: calc(var(--bucketX, 50%) - 5%);
        color: var(--bucket-color);
        transition:0.6s left;



    }


    .apple {
        position: absolute;
        top: var(--appleY, 0%);
        left: calc(var(--appleX, 50%) - 2.5%);
        color: var(--apple-color);
        width: 5vw;
        height: auto;
        transition: .1s top;
    }

    /* Счётчик */
    .score {
        position: absolute;
        top: 10px;
        left: 10px;
        font-size: 24px;
        color: var(--text-color);
    }

    /* Кнопки управления */
    .controls {
        position: absolute;
        bottom: 11vw;
        width: 100%;
        display: flex;
        justify-content: center;
        gap: 10px;
        margin-bottom: 10px;
        border-bottom: 1px solid gray;

    }


    .controls button:hover{
        color: green;
        border: 2px dashed green;
    }

    .hovered{
        color: green;
        border: 2px dashed green;
    }

    button {
        padding: 10px;
        font-size: 18px;
        cursor: pointer;
        background-color: transparent;
        color: var(--text-color);
        border: 1px solid var(--text-color);
        border-radius: 15px;
        font-family: inherit;
        transition: .2s;

    }

    button:hover {
        color: green
    }

    .theme-toggle {
        position: absolute;
        transition: .4s;
        width: 60px;
        top: 10px;
        right: 10px;
        padding: 5px 10px;
        border: none;
    }

    .theme-toggle:hover, .settings:hover{
        color: green;
    }

    .settings {
        position: absolute;
        transition: .4s;
        width: 59px;
        top: 8px;
        right: 60px;
        padding: 5px 10px;
        cursor: pointer;
        border: none;
    }

    /*Стандартная и темная тема*/
    :root {
        --background-color: #f9f9f9;
        --bucket-color: #101010;
        --apple-color: red;
        --text-color: #333;
        --shadow-color: rgba(1, 1, 1, 0.1)
    }

    .dark {
        --background-color: #333;
        --bucket-color: #f0f0f0;
        --apple-color: #ff0000;
        --text-color: #f0f8ff;
        --shadow-color: rgba(1, 1, 1, 0.1)
    }
</style>