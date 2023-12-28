<script>
    export let question = {
        question: "",
        answers: [],
        progress: 0
    }

    const answersWrapSmall = 'https://static.tildacdn.com/tild3930-6133-4166-b065-623234313863/questionWrap-small.svg';
    const luigiSmall = 'https://static.tildacdn.com/tild3431-3933-4362-b236-373538656365/luigi.png';
    const heartIcon = 'https://static.tildacdn.com/tild6166-3635-4365-a436-353566313938/Heart.png';

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

    let minimalHeight = 350;
    $: if (window.innerWidth > 1000) minimalHeight = 420;
    let answersCalculatedHeight = minimalHeight;
    answersCalculatedHeight = (question.answers.length * 100 > minimalHeight ? question.answers.length * 100 : minimalHeight)

    // $: try {
    //     question = question;
    //     let answersHeight = 0; 
    //     document.querySelectorAll('.answersWrap-svgWrap-answersWrapper-answer').forEach(a => answersHeight += a.offsetHeight)
    //     answersCalculatedHeight = answersHeight + 50;
    // } catch {
    //     answersCalculatedHeight = (question.answers.length * 100 > minimalHeight ? question.answers.length * 100 : minimalHeight)
    // }
</script>

<div class="answersWrap-luigiImg" title="Изображение весёлого робота, Луиджи" style={`background-image: url('${luigiSmall}')`} />
<div class="answersWrap-wrap">
    
    <div class='answersWrap-svgWrap'></div>

    <div class="answersWrap-svgWrap-answersWrapper" style={`height:${answersCalculatedHeight}px`}>
    {#each question.answers as answer, index}
        <!-- svelte-ignore a11y-click-events-have-key-events a11y-no-static-element-interactions-->
        <div 
            class="answersWrap-svgWrap-answersWrapper-answer" 
            class:answersWrap-svgWrap-answersWrapper-answer-active={question.answers[index] == lastAnswerStr}
            on:click|preventDefault={()=> { reportAnswer(index) }}
            >
            <img alt="Иконка сердечка" src={heartIcon} class="answersWrap-svgWrap-answersWrapper-answer-icon" />
            <div class="answersWrap-svgWrap-answersWrapper-answer-text">{answer}</div>
        </div>
    {/each}
    </div>

    
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
        width: 80%;
        padding-top: 25px;
        padding-bottom: 1em;
        z-index: 1;
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

    .answersWrap-svgWrap-answersWrapper-answer-active {
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
        display: flex;
        flex-direction: column;
        align-items: center;
        position: relative;
    }

    .answersWrap-svgWrap {
        width: 92%;
        height: 100%;
        display: flex;
        justify-content: center;
        position: absolute;
        background-repeat: no-repeat;
        background-position-x: center;
        background-color: #2e3d97;
        border: 2px solid #222d7f;
        outline: 15px solid #232f81;
        border-radius: 15px;
        box-shadow: 2px 0px 68px #6044a3;
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
        outline: 9px solid #718bf9;
    }

    @media screen and (min-width: 1200px) {
        .answersWrap-luigiImg {
            background-position: right;
            background-repeat: no-repeat;
            width: auto;
            height: 700px;
            margin-top: -59px;
            margin-bottom: -157px;
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
        }

        .answersWrap-svgWrap-answersWrapper-answer {
            height: 111px;
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