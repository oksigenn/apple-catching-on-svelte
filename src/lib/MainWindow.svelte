<script>
//Импорт картинок
import GiEmptyWoodBucket from 'svelte-icons/gi/GiEmptyWoodBucket.svelte'
import FaAppleAlt from 'svelte-icons/fa/FaAppleAlt.svelte'
import FaSun from 'svelte-icons/fa/FaSun.svelte'
import FaMoon from 'svelte-icons/fa/FaMoon.svelte'
import GoGear from 'svelte-icons/go/GoGear.svelte'
import Modal from './Modal.svelte'
import GiShinyApple from 'svelte-icons/gi/GiShinyApple.svelte'
import GiEmptyMetalBucket from 'svelte-icons/gi/GiEmptyMetalBucket.svelte'
import FaInfoCircle from 'svelte-icons/fa/FaInfoCircle.svelte'

//Импорт OnMount()
import { onMount } from 'svelte';

//Переменная для смены темы (экспорт)
export let isDarkMode = false

//Переменная для смены языка
let language = 'en'

//Переменные для передвижения
let isHoveredRight = false
let isHoveredLeft = false
let bucketX = 50;
let bucketY = 0;
let appleY = 0;
let appleX = Math.random() * 90;
let bucketSpeed = 3;
let termBucketSpeed = bucketSpeed;
let speed = 0.5;
let termSpeed = speed;

//Важные переменные для процесса
let score = 0;
let gameInterval;

//Смена темы яблока и ведра
let appleMode = 'normal'
let bucketMode = 'normal'

//Показ модальных окон
let showLoseModal = false;
let showSettingsModal = false
let showInfoModal = false


//Скрипты, функции

    // Перемещаем яблоко к верхнему краю
    function resetApple() {
        appleY = 0;
        appleX = Math.random() * 100;
        appleX = Math.max(10, Math.min(90, appleX))
    }

function toggleLang(){
    language = language==='en'?'ru':'en'
}

    function updateBucketWidth() {
        if (window.innerWidth <= 768 && window.innerHeight >500) {
            bucketY = 82; // Уменьшаем размер ведра на маленьких экранах
        } else if (window.innerWidth <= 1000 && window.innerWidth > 880) {
            bucketY = 75;
        } else if (window.innerWidth <= 1200 && window.innerWidth > 1000) {
            bucketY = 76;
        } else if (window.innerWidth <= 880 && window.innerWidth > 768) {
            bucketY = 80
        }else if (window.innerWidth <= 500) {
            bucketY = 87;
        }

    }


function isCollision() {
    // Получаем реальные позиции и размеры ведра и яблока
    const bucketElement = document.querySelector(".bucket");
    const appleElement = document.querySelector(".apple");

    if (bucketElement && appleElement) {
        const bucketRect = bucketElement.getBoundingClientRect();
        const appleRect = appleElement.getBoundingClientRect();

        // Проверка, что яблоко касается ведра только сверху и находится прямо над ним
        return (
            appleRect.bottom >= bucketRect.top +30 && // Яблоко касается верхней границы ведра
            appleRect.bottom <= bucketRect.top + 35 && // Оставляем небольшое расстояние для точности попадания
            appleRect.left >= bucketRect.left && // Яблоко по горизонтали полностью над ведром
            appleRect.right <= bucketRect.right
        );
    }
    return false;
}

function updateGame() {
    appleY += speed;

    // Проверка на касание ведра и яблока
    if (isCollision()) {
        score++;
        resetApple();
    }

    // Проверка на нижнюю границу
    if (appleY >= 90) {
        showLoseModal = openModal();
        resetApple()
    }
}


    // Перемещение ведра влево или вправо
function moveBucket(direction) {
    const windowWidth = window.innerWidth;
    const bucketWidth = document.querySelector(".bucket").offsetWidth;

    // Проверка, не выходит ли ведро за левый край
    if (direction === -1 && bucketX > 0) {
        bucketX += direction * bucketSpeed;
    }

    // Проверка, не выходит ли ведро за правый край
    if (direction === 1 && bucketX < 99 - (bucketWidth / windowWidth) * 100) {
        bucketX += direction * bucketSpeed;
    }
}


    function handleKeydown(event) {
        if (event.key === 'ArrowLeft') {
            moveBucket(-1)
            console.log(bucketX)
            isHoveredLeft = true
        } else if (event.key === 'ArrowRight') {
            moveBucket(1)
            isHoveredRight = true
            console.log(bucketX)

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
    const mediaQuery = window.matchMedia('(prefers-color-scheme: dark)');
    isDarkMode = mediaQuery.matches;
    const handleChange = (e) => {
        isDarkMode = e.matches;
    };
    mediaQuery.addEventListener('change', handleChange);

    onMount(() => {
        console.log(bucketX);
        updateBucketWidth()
        gameInterval = setInterval(updateGame, 50);
        window.addEventListener('keydown', handleKeydown);
        window.addEventListener('keyup', handleKeyup);


        // Очистка слушателя при размонтировании компонента
        return () => {

        };

        return () => {
            clearInterval(gameInterval);
            window.removeEventListener('keydown', handleKeydown);
            window.removeEventListener('keyup', handleKeyup);

            console.log(bucketX)
        };
    });




    function openModal() {

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
</script>


<div class="game-container" class:dark={isDarkMode}>
    <div class="score">{language === 'en' ? 'Score' : 'Счёт'}: {score}</div>

    <div class="bucket" style="--bucketX: {bucketX}%; bottom: 2%;">
        {#if bucketMode==='normal'}
            <GiEmptyWoodBucket/>
        {:else if bucketMode==='metal'}
            <GiEmptyMetalBucket/>
        {/if}
    </div>

    <div class="apple" style="--appleY: {appleY}%; --appleX: {appleX}%">
        {#if appleMode==='normal'}
            <FaAppleAlt/>
        {:else}
            <GiShinyApple/>
        {/if}
    </div>

    <div class="controls">
        <button class:hovered={isHoveredLeft} on:click={()=>moveBucket(-1)}>
            {language === 'en' ? '← Left' : '← Влево'}
        </button>
        <button class:hovered={isHoveredRight} on:click={() => moveBucket(1)}>
            {language === 'en' ? 'Right →' : 'Вправо →'}
        </button>
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

    <button class="info" on:click={()=>showInfoModal=openModal()}>
        <FaInfoCircle/>
    </button>

    <Modal isOpen={showLoseModal} buttonText={language === 'en' ? "Try again" : "Попробовать снова"} on:close={()=>{showLoseModal=closeModal(); score=0}}>
        <h2>{language === 'en' ? 'You lost!' : 'Вы проиграли!'}</h2>
        <p>{language === 'en' ? `You scored ${score} ${score === 1 ? 'point' : 'points'}` : `Вы набрали ${score} ${score%10 === 1 ? 'очко' : 'очков'}`}</p>
    </Modal>

    <Modal isOpen={showInfoModal} buttonText={language === 'en' ? 'Close' : 'Закрыть'} on:close={()=>showInfoModal=closeModal()}>
        <h2>{language === 'en' ? 'Information' : 'Информация'}</h2>
        <p><a href="https://t.me/aertydesign">{language === 'en' ? 'Telegram' : 'Телеграм'}</a></p>
        <p><a href="https://discordapp.com/users/1156598346803331285/">{language === 'en' ? 'Discord' : 'Дискорд'}</a></p>
    </Modal>

    <Modal isOpen={showSettingsModal} buttonText={language === 'en' ? 'Close' : 'Закрыть'} on:close={()=>showSettingsModal=closeModal()}>
        <h2>{language === 'en' ? 'Settings' : 'Настройки'}</h2>
        <label>{language === 'en' ? 'Apple falling speed' : 'Скорость падения яблока'}: {termSpeed}</label>
        <input type="range" list="markers" min="1" max="5" step="1" bind:value={termSpeed} />
        <label>{language === 'en' ? 'Bucket falling speed' : 'Скорость падения ведра'}: {termBucketSpeed}</label>
        <input type="range" list="markers" min="1" max="10" step="1" bind:value={termBucketSpeed} />
        <div class="row">
            {language === 'en' ? 'Choose apple style:' : 'Выберите стиль яблока:'}
            <div  style="width: 50px;" class="row-in">
                <div style="width: 30px; height: 20px" class="row-el" on:click={()=>appleMode='normal'}><FaAppleAlt/></div>
                <div style="width: 30px; height: 20px" class="row-el" on:click={()=>appleMode='shiny'}><GiShinyApple/></div>
            </div>
        </div>
        <div class="row">
            {language === 'en' ? 'Choose bucket style:' : 'Выберите стиль ведра:'}
            <div  style="width: 60px;" class="row-in">
                <div   class="row-el" on:click={()=>bucketMode='normal'}><GiEmptyWoodBucket/></div>
                <div  class="row-el" on:click={()=>bucketMode='metal'}><GiEmptyMetalBucket/></div>
            </div>
        </div>
        <button on:click={toggleLang} class="language-toggle">
            {language === 'en' ? 'Switch to Russian' : 'Переключить на английский'}
        </button>
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

    .language-toggle {
        font-size: 16px;
        cursor: pointer;
        padding: 5px 10px;
        background-color: transparent;
        color: var(--text-color);
        border: 1px solid var(--text-color);
        border-radius: 15px;
        font-family: inherit;
        transition: .2s;
        outline: none;
    }

    .language-toggle:hover {
        color: green;
        border: 2px dashed green;
    }

    html, body {
        margin: 0;
        padding: 0;
        overflow: hidden;


    }
    .row{
        display: flex;
        gap: 10px;
    }


    a{
        text-decoration: none;
        transition: .4s ease;
        color: var(--text-color);
    }

    a:hover{
        color: green;
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
        overflow: hidden;
        margin: 0;
        padding: 0;
        width: 100%;
        height: 100vh;
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
        left: calc(var(--bucketX));
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
        outline: none;

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

    .info {
        position: absolute;
        transition: .4s;
        width: 59px;
        top: 11px;
        right: 110px;
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

    @media  screen and (max-width: 1200px) {
        .apple{
            width: 6vw;
        }

        .bucket{
            width: 12vw;
        }
    }

    @media  screen and (max-width: 1000px) {
        .apple{
            width: 6vw;
        }

        .bucket{
            width: 14vw;
        }

        .controls{
            bottom: 14vw;
        }

        .theme-toggle, .settings, .info{
            width: 55px;
        }

        .theme-toggle{
            right: 5px;
        }

        .settings{
            right: 47px;
        }

        .info{
            right: 90px;
        }

        .score{
            font-size: 22px;
        }
    }

    @media screen and (max-width: 768px) {
        .apple{
            width: 8vw;
        }

        .bucket{
            width: 18vw;
        }

        .controls{
            bottom: 22vw;
        }
    }

    @media screen and (max-width: 600px) {
        .apple{
            width: 9vw;
        }

        .bucket{
            width: 22vw;
        }

        .controls{
            bottom: 24vw;
        }
    }

    @media  screen and (max-width: 500px) {
        .apple{
            width: 11vw;
        }

        .bucket{
            width: 25vw;
        }

        .controls{
            bottom: 30vw;
        }

        .theme-toggle, .settings, .info{
            width: 48px;
        }

        .theme-toggle{
            right: 1px;
        }

        .settings{
            right: 35px;
        }

        .info{
            right: 70px;
        }

        .score{
            font-size: 20px;
        }
    }

    @media screen and (max-width: 350px) {
        .apple{
            width: 13vw;
        }

        .bucket{
            width: 27vw;
        }

        .controls{
            bottom: 33vw;
        }
    }
</style>