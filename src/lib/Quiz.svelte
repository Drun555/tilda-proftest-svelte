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
  
  // это хэндлеры GTM на определённые вопросы
  const GTM_QuestionEvents = [
    {
      question: "Что ты предпочитешь?",
      dataLayer: {
          'event' : 'GTMevent',
          'eventCategory': 'button',
          'eventAction': 'click',
          'eventLabel': 'first_page'
      }
    },
    {
      question: "Что тебе больше нравилось в школе?",
      dataLayer: {
          'event' : 'GTMevent',
          'eventCategory': 'answer',
          'eventAction': 'success',
          'eventLabel': 'first_page_question'
      }
    },
    {
      question: "В разговоре тебе интереснее:",
      dataLayer: {
          'event' : 'GTMevent',
          'eventCategory': 'answer',
          'eventAction': 'success',
          'eventLabel': 'middle_question'
      }
    },
    {
      question: "Насколько легко ты идёшь на компромисс?",
      dataLayer: {
          'event' : 'GTMevent',
          'eventCategory': 'answer',
          'eventAction': 'success',
          'eventLabel': '60_percent_question'
      }
    },
    {
      question: "В каком ритме тебе комфортнее работать?",
      dataLayer: {
          'event' : 'GTMevent',
          'eventCategory': 'answer',
          'eventAction': 'success',
          'eventLabel': '70_percent_question'
      }
    },
    {
      question: "Есть ли айти-профессия, которая уже тебе приглянулась?",
      dataLayer: {
          'event' : 'GTMevent',
          'eventCategory': 'answer',
          'eventAction': 'success',
          'eventLabel': '80_percent_question'
      }
    },
    {
      question: "Сколько ты зарабатываешь сейчас?",
      dataLayer: {
          'event' : 'GTMevent',
          'eventCategory': 'answer',
          'eventAction': 'success',
          'eventLabel': '90_percent_question'
      }
    }
  ]
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

  // Проверяем, есть ли триггер на текущий вопрос - и если да, то пушим в dataLayer
  $: currentQuestionIndex && analytics() 
  function analytics() {
    // в dev-режиме у нас нет dataLayer
    if (import.meta.env.DEV) return;

    for (const index in GTM_QuestionEvents) {
      if (GTM_QuestionEvents[index].question !== data.QUESTIONS[currentQuestionIndex].question) 
        continue;
      
      try {
        dataLayer.push( GTM_QuestionEvents[index].dataLayer );
      } catch {
        console.warn('Ошибка dataLayer.push')
      }
    } 
  }

  window.addEventListener('answerPressed', e => {
    userAnswers.push( e.detail );
    
    if (currentQuestionIndex+1 < data.QUESTIONS.length) {
      let answer = e.detail;
      currentAnswerIndex += answer.answerIndex
      console.log(`Вопрос ${currentQuestionIndex}, ответ #${currentAnswerIndex}`)

      // Меняем вопрос с задержкой
      setTimeout(() => currentQuestionIndex++, 300);
      // currentQuestionIndex++
      // Меняем обои
      if (currentQuestionIndex % 6 == 1) {
        if (currentBackgroundIndex+1 < backgrounds.length)
          currentBackgroundIndex++;
        else currentBackgroundIndex = 0;
      }

      // прямо во время ответов считаем веса
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
      
      // Аналитика
      if (import.meta.env.PROD)
        dataLayer.push({
          'event' : 'GTMevent',
          'eventCategory' : 'viewing',
          'eventAction' : 'success',
          'eventLabel' : 'contact_page'
        });

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

      document.querySelector(window.luigiQuiz.thisBlock).style.display = "none";
      document.querySelector(window.luigiQuiz.finalBlock).style.display = "block";
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
    display: flex;
    flex-direction: column;
    align-items: center;
    position: relative;
  }

  .currentQuestion-wrap {
    display: flex;
    margin-top: 110px;
    margin-bottom: 36px;
    width: 100%;
    z-index: 1;
  }

  .currentQuestion-title {
    background-color: white;
    border-radius: 18px;
    padding: 16px;
    border: 4px solid rgb(87, 105, 226);
    padding-top: 13px;
    padding-bottom: 12px;
    padding-right: 22px;
    color: #1e2752;
    font-weight: 400;
    font-size: 17px;
  }

  @media screen and (min-width: 1200px) {
    main {
      background-size: cover;
      padding-top: 45px;
    }

    .content {
      width: 1060px;
    }

    .currentQuestion-wrap {
      margin-top: 140px;
      margin-bottom: 130px;
    }

    .currentQuestion-title {
      font-size: 24px;
      font-weight: 400;
    }
  }
</style>