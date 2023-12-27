<script>
    export let question = {
        question: "",
        answers: [],
        progress: 0
    }

    const answersWrapSmall = 'https://static.tildacdn.com/tild3930-6133-4166-b065-623234313863/questionWrap-small.svg';
    const luigiSmall = 'https://static.tildacdn.com/tild3431-3933-4362-b236-373538656365/luigi.png';
    const heartIcon = 'https://static.tildacdn.com/tild6166-3635-4365-a436-353566313938/Heart.png';

    function reportAnswer(i) {
        var evt = new CustomEvent("answerPressed", { detail: {
            answerIndex: i
        }});
        window.dispatchEvent(evt);
    }
</script>
<!-- <img alt="Изображение весёлого робота, Луиджи" src={luigiSmall} /> -->
<div class="answersWrap-luigiImg" title="Изображение весёлого робота, Луиджи" style={`background-image: url('${luigiSmall}')`} />
<div class="answersWrap-wrap">
    
    <div class='answersWrap-svgWrap' style={`background-image: url('${answersWrapSmall}'`}>
        <div class="answersWrap-svgWrap-answersWrapper">
        {#each question.answers as answer, index}
            <!-- svelte-ignore a11y-click-events-have-key-events a11y-no-static-element-interactions-->
            <div class="answersWrap-svgWrap-answersWrapper-answer" on:click={()=>reportAnswer(index)}>
                <img alt="Иконка сердечка" src={heartIcon} class="answersWrap-svgWrap-answersWrapper-answer-icon" />
                <div class="answersWrap-svgWrap-answersWrapper-answer-text">{answer}</div>
            </div>
        {/each}
        </div>
    </div>
    <div class='answersWrap-svgWrap' style={`background-image: url('${answersWrapSmall}'`}></div>
    
</div>

<style>

    .answersWrap-svgWrap-answersWrapper-answer-icon {
        object-fit: contain;
        padding-right: 17px;
        padding-left: 15px;
    }

    .answersWrap-svgWrap-answersWrapper {
        display: flex;
        flex-direction: column;
        gap: 10px;
        width: 71%;
        padding-top: 2em;
        padding-bottom: 1em;
    }

    .answersWrap-svgWrap-answersWrapper-answer {
        font-size: 14px;
        background-color: white;
        padding: 15px;
        border-radius: 10px;
        display: flex;
        gap: 10px;
        line-height: 20.063px;
        color: #1E2752;
        font-weight: 500;
        border: 3px solid #FFF;
        transition: 0.4s;
    }

    .answersWrap-svgWrap-answersWrapper-answer:hover {
        border: 3px solid #F989FF;
        transition: 0.4s;
        cursor: pointer;
    }

    .answersWrap-luigiImg {
        background-repeat: no-repeat;
        width: 100%;
        height: 290px;
        margin-top: -35px;
    }
    .answersWrap-wrap {
        width: 100%;
        display: flex;
        flex-direction: column;
        align-items: center;
        width: 100%;
        position: relative;
    }

    .answersWrap-svgWrap {
        width: 100%;
        height: 100%;
        display: flex;
        align-items: center;
        justify-content: center;
        position: relative;
        background-repeat: no-repeat;
        background-position-x: center;
        
    }
</style>