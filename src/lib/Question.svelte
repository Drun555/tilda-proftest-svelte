<script>
    import '../assets/styles.scss'
    export let question = {
        question: "",
        answers: [],
        progress: 0
    }

    // Короче, я не придумал как нормально красить кнопку после нажатия, поэтому сделал костыль
    let lastAnswerStr = "";
    let lastQuestionStr = "";
    function reportAnswer(i) {
        if (lastQuestionStr == question.question) return;
        var evt = new CustomEvent("answerPressed", { detail: {
            answerIndex: i,
            answersLength: question.answers.length,
            questionString: question.question,
            answerString: question.answers[i]
        }});
        lastAnswerStr = question.answers[i];
        lastQuestionStr = question.question;
        window.dispatchEvent(evt);
    }
</script>

<div class="answersWrap-luigiImg" title="Изображение весёлого робота, Луиджи"  />
<div class="answersWrap-wrap">
    
    <div class='answersWrap-svgWrap'></div>

    <div class="answersWrap-svgWrap-answersWrapper">
    {#each question.answers as answer, index}
        <!-- svelte-ignore a11y-click-events-have-key-events a11y-no-static-element-interactions-->
        <div 
            class="answersWrap-svgWrap-answersWrapper-answer" 
            class:answersWrap-svgWrap-answersWrapper-answer-active={question.answers[index] == lastAnswerStr}
            on:click|preventDefault={()=> { reportAnswer(index) }}
            >
            <img alt="Иконка сердечка" class="answersWrap-svgWrap-answersWrapper-answer-icon" />
            <div class="answersWrap-svgWrap-answersWrapper-answer-text">{answer}</div>
        </div>
    {/each}
    </div>

    
</div>

<style>

    .answersWrap-svgWrap-answersWrapper-answer-icon {
        object-fit: contain;
        padding-right: 7px;
        /* padding-left: 15px; */
    }

    .answersWrap-svgWrap-answersWrapper {
        display: flex;
        flex-direction: column;
        gap: 10px;
        width: 84%;
        padding-top: 17px;
        padding-bottom: 1em;
        transition: 0.3s;
        z-index: 1;
    }

    .answersWrap-svgWrap-answersWrapper-answer {
        font-size: 17px;
        background-color: white;
        padding: 15px;
        border-radius: 10px;
        display: flex;
        align-items: center;
        gap: 10px;
        line-height: 20.063px;
        color: #1E2752;
        font-weight: 400;
        border: 3px solid #FFF;
        transition: 0.4s;
        min-height: 80px;
    }

    .answersWrap-svgWrap-answersWrapper-answer-active {
        border: 3px solid #F989FF;
        transition: 0.4s;
        cursor: pointer;
    }

    .answersWrap-luigiImg {
        background-repeat: no-repeat;
        width: 83%;
        height: 297px;
        background-size: cover;
        position: absolute;
        top: 1px;
    }

    .answersWrap-wrap {
        display: flex;
        flex-direction: column;
        align-items: center;
        position: relative;
        margin-bottom: 50px;
        width: 100%;
    }

    .answersWrap-svgWrap {
        width: 92%;
        height: 1000px;
        display: flex;
        justify-content: center;
        position: absolute;
        background-repeat: no-repeat;
        background-position-x: center;
        background-color: #2e3d97;
        border: 2px solid #222d7f;
        box-shadow: 0 0 0 15px #232f81;
        border-radius: 15px;
        /* box-shadow: 2px 0px 66px #6044a3, 0 0 0 15px #232f81; */
        box-shadow: 2px 0px 66px #6044a3;
        position: absolute;
    }

    .answersWrap-svgWrap::before {
        content: ' ';
        top: -5px;
        width: 100%;
        height: 100%;
        position: absolute;
        border: 5px solid #4456da;
        border-radius: 20px;
        box-shadow: 0 0 0 6px #718bf9;
    }

    @media screen and (min-width: 1200px) {
        .answersWrap-svgWrap-answersWrapper-answer:hover {
            border: 3px solid #F989FF;
            transition: 0.4s;
            cursor: pointer;
        }
        
        .answersWrap-luigiImg {
            background-position: right;
            background-repeat: no-repeat;
            width: 100%;
            height: 594px;
            background-size: contain;
        }

        .answersWrap-svgWrap {
            width: 97%;
        }

        .answersWrap-svgWrap-answersWrapper {
            width: 93%;
            gap: 16px;
            display: grid;
            grid-template-columns: 1fr 1fr 1fr;
            grid-auto-rows: max-content;
            padding-top: 23px;
        }

        .answersWrap-svgWrap-answersWrapper-answer {
            min-height: 83px;
        }

        .answersWrap-svgWrap-answersWrapper-answer-icon {
            max-width: 20px;
        }

        .answersWrap-svgWrap-answersWrapper-answer-text {
            display: flex;
            align-items: center;
            font-size: 18px;
            line-height: 120%;
        }
    }
</style>