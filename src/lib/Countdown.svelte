<script lang="ts">
    import { createEventDispatcher } from "svelte";
    const dispatch = createEventDispatcher();
    import { onMount } from 'svelte';
    let baseSeconds = 10;
    

    let halo: HTMLElement = document.createElement('circle');
    let repeat = null;

    window.addEventListener("answerPressed", e => {
        startAnimation();
    })

    onMount(() => startAnimation())

    let countDown = setInterval(() => {}, 9999999);
    function startAnimation() {
        clearInterval(countDown);
        let svgWrapper = document.querySelector('.svgWrapper')
        svgWrapper.parentElement.classList.remove('shaking')

        baseSeconds = 10;
        halo = document.querySelector('circle')
        halo.classList.remove('circleAnimation')
        setTimeout(() => halo.classList.add('circleAnimation'), 50)
        repeat = baseSeconds;
        

        countDown = setInterval(() => {
            baseSeconds--;
            if (baseSeconds == 0) {
                svgWrapper.parentElement.classList.add('shaking')
                clearInterval(countDown);
                baseSeconds = 0;
                

                dispatch("cleared");
            }
        }, 1000);
    }
    let screenWidth;

    let circleRadius = 25;
    $: circleRadius = (screenWidth > 1260 ? 40 : 30)
</script>
<svelte:window bind:innerWidth={screenWidth} />

{#key repeat}
<div class="timer">
    <div class="svgWrapper">
        <svg width="100%" style="overflow: visible;" height="100%" version="1.1" preserveAspectRatio="none" >
            <circle class="circleAnimation" cx="50%" cy="50%" r={circleRadius} fill="#df1b1229" stroke="#DF1B12" stroke-width="5" stroke-dasharray="0, 361" />
        </svg>
        <div class="timer-number">
            {baseSeconds}
        </div>
    </div>
</div>
{/key}

<style lang="scss">

    .circleAnimation {
        animation: circleCountdown 10s 1, svgWrapperRotate 10s infinite;
        animation-timing-function: linear;
        transform-origin: center;
    }

    :global(.shaking) {
        animation: shakeAnim 0.3s infinite;
        animation-timing-function: linear;
        transform-origin: center;
    }

    @keyframes -global-circleCountdown {
        from {
            stroke-dasharray: 245,360;
        }
        to {
            stroke-dasharray: 0,360;
        }
    }

    @keyframes -global-svgWrapperRotate {
        from {
            transform: rotate(0deg);
        }
        to {
            transform: rotate(-361deg);
        }
    }

    @keyframes shakeAnim {
        0% {
            transform: translate(3px, 3px);
        }
        50% {
            transform: translate(-3px, -3px);
        }
        100% {
            transform: translate(3px, 3px);
        }
    }

    .svgWrapper {
        animation: scaleBounce 1s cubic-bezier(0.175, 0.885, 0.32, 1.275) infinite;
        transform-origin: center;
        transition: 0.3s;
        position: relative;
        display: flex;
        align-items: center;
        justify-content: center;
    }

    .timer-number {
        position: absolute;
        color: #DF1B12;
        text-align: center;
        font-family: StratosSkyeng;
        font-size: 20px;
        font-weight: 500;
    }

    @keyframes scaleBounce {
        from {
            transform: scale(1.1);
        }
        to {
            transform: scale(1);
        }
    }

    .timer{
        display:flex;
        align-items:center;
        justify-content:center;
        height: 60px;
        width: 60px;
        aspect-ratio: 1 / 1;
    }

    @media screen and (min-width: 1260px) { 
        .timer{
            height: 80px;
            width: 80px;
        }

        .timer-number {
            font-size: 32px;
        }
    }
</style>
