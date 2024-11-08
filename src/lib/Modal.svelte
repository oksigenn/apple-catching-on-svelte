<script>
    import { createEventDispatcher } from 'svelte';
    import isDarkMode from "./MainWindow.svelte";

    export let isOpen = false;
    export let buttonText = 'close'
    let closing = false;
    const dispatch = createEventDispatcher();

    function closeModal() {
        // Устанавливаем состояние закрытия и запускаем анимацию
        closing = true;
        setTimeout(() => {
            closing = false;
            dispatch('close'); // уведомляем родительский компонент
        }, 500); // Продолжительность анимации закрытия
    }
</script>

{#if isOpen}
    <div class="modal-backdrop" on:click={closeModal} class:dark = {isDarkMode}>
        <div class="modal-content {closing ? 'closing' : ''}" on:click|stopPropagation>
            <slot></slot>
            <button class="close-button" on:click={closeModal}>{buttonText}</button>
        </div>
    </div>
{/if}

<style>
    .modal-backdrop {
        position: fixed;

        color: var(--text-color);
        top: 0;
        left: 0;
        width: 100vw;
        height: 100vh;
        background:var(--shadow-color);
        display: flex;
        align-items: center;
        justify-content: center;
        z-index: 1000;
    }

    .modal-content {
        background: var(--background-color);
        display: flex;
        flex-direction: column;
        padding: 20px;
        margin: 15px;
        border-radius: 15px;
        box-shadow: 0 2px 10px rgba(255, 255, 255, 0.7);
        transform: scale(0.1);
        opacity: 0;
        animation: appleGrow 0.6s ease-out forwards;
    }

    .modal-content.closing {
        animation: appleShrink 0.5s ease-out forwards;
    }

    .close-button {
        margin-top: 20px;
        padding: 10px 15px;
        font-size: 18px;
        cursor: pointer;
        background-color: var(--apple-color);
        color: var(--text-color);
        border: none;
        border-radius: 15px;
        transition: .4s;
    }

    .close-button:hover{
        background-color: green;
    }

    /* Анимация открытия */
    @keyframes appleGrow {
        0% {
            transform: scale(0.1) rotate(-20deg);
            opacity: 0;
        }
        50% {
            transform: scale(1.1) rotate(10deg);
            opacity: 0.8;
        }
        100% {
            transform: scale(1) rotate(0deg);
            opacity: 1;
        }
    }

    /* Анимация закрытия */
    @keyframes appleShrink {
        0% {
            transform: scale(1) rotate(0deg);
            opacity: 1;
        }
        100% {
            transform: scale(0.1) rotate(-20deg);
            opacity: 0;
        }
    }
</style>
