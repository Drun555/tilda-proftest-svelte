<script>
  import '../assets/styles.scss'
  import data from './questions.js'
  // import ProgressBar from './ProgressBar.svelte.js'
  import Question from './Question.svelte'

  const preload = [
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

  data.QUESTIONS.forEach(q => {
    if (q.hasOwnProperty("questionImageUrl"))
      preload.push(q.questionImageUrl)
  })
  preloadImages(preload);
  
  let currentQuestionIndex = 0;
  let currentAnswerIndex = 0;
  let complete = false;


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

  // проставим стартовые индексы вопросов, чтобы потом было удобно их считать
  let startIndex = 0;
  data.QUESTIONS.forEach(q => {
    q.startAnswerIndex = startIndex;
    startIndex += q.answers.length
  })

  let userAnswers = {};
  function writeAnswer(index) {    
    setTimeout(() => { // Меняем вопрос с задержкой
      userAnswers[currentQuestionIndex] = {
        questionLink: data.QUESTIONS[currentQuestionIndex],
        questionIndex: currentQuestionIndex,
        answerIndex: index,
        // Ответы у нас в одном едином массиве. Например, третий ответ третьего вопроса будет под индексом 8
        // Чтобы потом удобно по нему пробегаться, нам нужно записать общий индекс ответа
        overallAnswerIndex: data.QUESTIONS[currentQuestionIndex].startAnswerIndex + index
      }
      
      window.dispatchEvent(
        new CustomEvent("answerPressed", { detail: {
          questionLink: data.QUESTIONS[currentQuestionIndex],
          questionIndex: currentQuestionIndex,
          answerIndex: index,
          overallAnswerIndex: data.QUESTIONS[currentQuestionIndex].startAnswerIndex + index
        }})
      );

      if (currentQuestionIndex +1 == data.QUESTIONS.length)
        currentQuestionIndex++
      else
        proftestCompleted()

    }, ((import.meta.env.PROD) ? 500 : 0));
  }

  function proftestCompleted() {

    // подсчитаем общие баллы по каждой профессии
    userAnswers.forEach(a => {
      data.RESULT_PROFESSION.forEach(prof => {
        prof.userProggress += prof.dataCheck[a.overallAnswerIndex] || 0
      })
    })

    // Отсортируем
    data.RESULT_PROFESSION.sort((a, b) => b.userProggress - a.userProggress);

    // Отчитываемся
    var evt = new CustomEvent("proftestCompleted", { detail: {
      answers: userAnswers,
      professions: data.RESULT_PROFESSION,
      usedData: data
    }});
    window.dispatchEvent(evt);
  }

  window.addEventListener('answerPressed', e => {
    userAnswers.push( e.detail );
    
    if (currentQuestionIndex+1 < data.QUESTIONS.length) {
      let answer = e.detail;
      currentAnswerIndex += answer.answerIndex
      console.log(`Вопрос ${currentQuestionIndex}, ответ #${currentAnswerIndex}`)

      
      // currentQuestionIndex++
      
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
      

      // Посчитаем проценты
      data.RESULT_PROFESSION.forEach(prof => {
        prof.userProggress = Math.round(prof.userProggress * 3.05 + 20);
        console.log(`${prof.title}, ${prof.userProggress}%`)
      })

      

      document.querySelector(window.luigiQuiz.thisBlock).style.display = "none";
      document.querySelector(window.luigiQuiz.finalBlock).style.display = "block";
    }
  })

  let question = null;
  $: question = data.QUESTIONS[currentQuestionIndex]
</script>

<main>

   <div class="z-skypro-proftest-wrapper">
    {#if !question.questionImageUrl}
    <div class="z-skypro-proftest-wrapper__header-wrap">
        <p class="z-skypro-proftest-wrapper__header">
            Пройдите тест, узнайте какой профессии подходите и&nbsp;получите
            бесплатную карьерную консультацию
        </p>
        <div class="z-skypro-proftest-wrapper__subtitle">
            В конце подарим
            <span class="ddd">скидку до 55%</span> на&nbsp;обучение
        </div>
    </div>
    {/if}

    {#if question.questionImageUrl}
    <div style={`background-image: url(${question.questionImageUrl})`} class="z-skypro-proftest-wrapper__question-image"></div>
    {/if}
    
    <div class="z-skypro-proftest-main-wrap">
        <div class="z-skypro-proftest">
            <span class="z-skypro-proftest__progress">{question.progress}</span>
            <h3 class="z-skypro-proftest__question">{question.question}</h3>

            {#key currentQuestionIndex}
            <div class="z-skypro-proftest__answers">
              {#each question.answers as answer, index}
                <!-- svelte-ignore a11y-click-events-have-key-events a11y-no-static-element-interactions-->
                <div 
                  class={
                    (userAnswers[currentQuestionIndex] && userAnswers[currentQuestionIndex].answerIndex == index) 
                    ? 
                    "z-skypro-proftest__answer z-skypro-proftest__answer--selected" 
                    : 
                    "z-skypro-proftest__answer"
                  } 
                  
                  on:click={function(e) {
                    document.querySelectorAll('.z-skypro-proftest__answer--selected').forEach(s => s.classList.remove('z-skypro-proftest__answer--selected'))
                    e.currentTarget.classList.add('z-skypro-proftest__answer--selected')
                    writeAnswer(index)
                  }}>
                  <span>{answer}</span>
                  <span class="z-skypro-proftest__answer-checkbox"></span>
                </div>
              {/each}
            </div>
            
            <div class="z-skypro-proftest__buttons">
                {#if currentQuestionIndex > 0}
                <button on:click={() => currentQuestionIndex-- } class="z-skypro-proftest__button z-skypro-proftest__button--prev">
                    Назад
                </button>
                {/if}
                {#if userAnswers[currentQuestionIndex]}
                <button on:click={() => currentQuestionIndex++ } class="z-skypro-proftest__button z-skypro-proftest__button--next">
                    Далее
                </button>
                {/if}
            </div>
            {/key}
        </div>
    </div>
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
    display: flex;
    justify-content: center;
  }
</style>