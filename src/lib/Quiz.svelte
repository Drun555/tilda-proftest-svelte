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

  // сразу загрузим картинки в кэш
  function preloadImages(array) {
      if (!preloadImages.list) {
          preloadImages.list = [];
      }
      var list = preloadImages.list;
      for (var i = 0; i < array.length; i++) {
          var img = new Image();
          img.onload = function() {
              var index = list.indexOf(this);
              if (index !== -1) {
                  // remove image from the array once it's loaded
                  // for memory consumption reasons
                  list.splice(index, 1);
              }
          }
          list.push(img);
          img.src = array[i];
      }
  }

  preloadImages(backgrounds);

  let currentBackgroundIndex = 0;
  let currentQuestionIndex = 0;
  let currentAnswerIndex = 0;
  let complete = false;
  const userAnswers = [];


  // Проверка весов
  let overallAnswers = 0;
  data.QUESTIONS.forEach(question => {
    overallAnswers += question.answers.length;
  })
  data.RESULT_PROFESSION.forEach(prof => {
    if (prof.dataCheck.length !== overallAnswers) {
      console.warn(`Несоответствие весов в ${prof.title}. Количество ответов всего - ${overallAnswers}, кол-во весов в dataCheck - ${prof.dataCheck.length}`)
    }
  })

  $: console.log( data.QUESTIONS[currentQuestionIndex] )

  window.addEventListener('answerPressed', e => {
    userAnswers.push( e.detail );
    
    if (currentQuestionIndex+1 < data.QUESTIONS.length) {
      let answer = e.detail;
      currentAnswerIndex += answer.answerIndex
      console.log(`Вопрос ${currentQuestionIndex}, ответ #${currentAnswerIndex}`)
      setTimeout(() => currentQuestionIndex++, 500);

      if (currentQuestionIndex % 6 == 1) currentBackgroundIndex++;
      data.RESULT_PROFESSION.forEach(prof => {
        prof.userProggress += prof.dataCheck[currentAnswerIndex] || 0
        if (typeof prof.dataCheck[currentAnswerIndex] === 'undefined')
          console.warn(`Нет веса в ${prof.title} на ответ #${currentAnswerIndex}`)

        console.log(`${prof.title}, ${prof.userProggress}`)
      })
      currentAnswerIndex = currentAnswerIndex - answer.answerIndex + answer.answersLength
    }
    
    window.scrollTo({ top: 0, behavior: 'smooth' });

    if (currentQuestionIndex + 1 == data.QUESTIONS.length) {
      // Если уже закончили, то шлём подальше
      if (complete) return;
      complete = true;
      
      // Посчитаем проценты
      data.RESULT_PROFESSION.forEach(prof => {
        prof.userProggress = Math.round(prof.userProggress * 3.05 + 20);
        console.log(`${prof.title}, ${prof.userProggress}%`)
      })

      // Отсортируем
      data.RESULT_PROFESSION.sort((a, b) => b.userProggress - a.userProggress);

      // Отчитываемся
      var evt = new CustomEvent("quizCompleted", { detail: {
        answers: userAnswers,
        professions: data.RESULT_PROFESSION
      }});
      window.dispatchEvent(evt);
    }
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
    min-height: 100vh;
    overflow: hidden;
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

  @media screen and (min-width: 1200px) {
    main {
      background-size: cover;
    }

    .content {
      width: 1160px;
    }

    .currentQuestion-wrap {
      position: absolute;
      top: 430px;
      margin-top: 0;
    }

    .currentQuestion-title {
      font-size: 24px;
    }
  }
</style>