<script>
  import data from './questions.js'
  import ProgressBar from './ProgressBar.svelte'
  import Question from './Question.svelte'

  const backgrounds = [
    "https://static.tildacdn.com/tild3965-6138-4561-a530-313462393561/Group_1321318412.png",
    "https://static.tildacdn.com/tild3764-3237-4930-a461-656661396230/Group_1321318413.png",
    "https://static.tildacdn.com/tild6264-3031-4234-b233-613966356266/Group_1321318414.png",
    "https://static.tildacdn.com/tild6163-3363-4232-a431-306131383062/Group_1321318415.png",
    "https://static.tildacdn.com/tild6430-6435-4466-b633-323463656432/Group_1321318416.png"
  ];

  let currentBackgroundIndex = 0;
  let currentQuestionIndex = 0;

  $: console.log( data.QUESTIONS[currentQuestionIndex] )
  window.addEventListener('answerPressed', e => {
    currentQuestionIndex++
  })
</script>

<main style={`background-image: url(${backgrounds[currentBackgroundIndex]})`}>
  <div class='content'>
    <ProgressBar progress={data.QUESTIONS[currentQuestionIndex].progress} />

    <div class="currentQuestion-wrap">
      <div class="currentQuestion-title">
        {data.QUESTIONS[currentQuestionIndex].question}
      </div>
    </div>
  
    <Question question={data.QUESTIONS[currentQuestionIndex]}/>
  </div>
  
</main>

<style>
  main {
    /* width: 100vw; */
    padding-top: 20px;
    padding-left: 20px;
    padding-right: 20px;
    position: relative;
    background-color: grey;
    /* background-repeat: no-repeat; */
    /* background-size: cover; */
    display: flex;
    justify-content: center;
  }

  .content {
    width: 350px;
  }

  .currentQuestion-wrap {
    display: flex;
    margin-top: 55px;
  }

  .currentQuestion-title {
    background-color: white;
    border-radius: 21px;
    padding: 16px;
    border: 3px solid rgb(87, 105, 226);
  }

  /* .answersWrap-outsideWrapper {
    
  } */
</style>