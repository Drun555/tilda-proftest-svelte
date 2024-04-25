<script>
	import { createEventDispatcher } from 'svelte';
	const dispatch = createEventDispatcher();

    export let answers = ["1", "2", "3"]
    let open = false;
    let choosedOption = "Выберите из списка";
	function answerPressed(answer, index) {
        choosedOption = answer;
        open = false;
		dispatch('answerPressed', index);
	}
</script>

<!-- svelte-ignore a11y-click-events-have-key-events a11y-no-static-element-interactions-->
<div class="proftest-select" on:click={() => open = !open}>
    <span class="select-text">
        {choosedOption}
    </span>
    <div class={!open ? "arrow_down" : "arrow_down arrow_reverse"}></div>
</div>
<div class={!open ? "proftest-select-options hidden" : "proftest-select-options"}>
    {#each answers as answer, index}
        <!-- svelte-ignore a11y-click-events-have-key-events a11y-no-static-element-interactions-->
        <div class="proftest-select-options__option select-text" on:click={() => answerPressed(answer, index)}>
            {answer}
        </div>
    {/each}
</div>

<style>
    .hidden .proftest-select-options__option {
        pointer-events: none;
    } 
    
    .proftest-select-options__option {
        height: calc(1em + 20px);
        display: flex;
        align-items: center;
        transition: 0.1s;
        padding-left: 25px;
        padding-right: 25px;
    }

    .proftest-select-options__option:hover {
        cursor: pointer;
        background-color: #f7f2f9;
        transition: 0.1s;
    }

    .proftest-select-options::before {
        width: calc(100% - 50px);
        content: '';
        height: 1px;
        display: block;
        position: relative;
        top: -18px;
        background-color: #efefef;
        align-self: center;
    }

    .proftest-select-options {
        max-height: unset;
        overflow: visible;
        opacity: 1;
        transition: 0.3s;
        width: 100%;
        background-color: white;
        position: relative;
        top: -9px;
        border: 1px solid #7334EA;
        border-top: 0px;
        box-sizing: border-box;
        border-bottom-right-radius: 10px;
        border-bottom-left-radius: 10px;
        padding-top: 25px;
        padding-bottom: 25px;
        display: flex;
        flex-direction: column;
    }

    .hidden {
        max-height: 0;
        overflow: hidden;
        opacity: 0;
        padding-top: 0;
        transition: 0.3s;
    }

    .proftest-select {
        width: 100%;
        height: 65px;
        border-radius: 10px;
        border: 1px solid #7334EA;
        background: #FFF;
        display: flex;
        justify-content: space-between;
        align-items: center;
        padding-left: 25px;
        padding-right: 25px;
        box-sizing: border-box;
    }

    .proftest-select:hover {
        cursor: pointer;
    }
    .select-text {
        font-size: 20px;
        font-style: normal;
        font-weight: 400;
    }

    .arrow_down {
        width: 10px;
        height: 10px;
        border-width: 3px;
        border-style:solid;
        border-color:transparent #7334ea #7334ea transparent;
        transform: rotate(45deg) translateY(-6px);
        transition: 0.3s;
    }

    .arrow_reverse {
        transform: rotate(225deg) translateY(-6px);
        transition: 0.3s;
    }
</style>