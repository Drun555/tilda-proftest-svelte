<script>
  // @ts-nocheck

  import '../assets/styles.scss'
  import { cumulativeOffset } from './helpers'
  let data = window.proftest;
  
  let blackTheme = true;
  $: if (blackTheme) {
    document.documentElement.style.setProperty('--background-color', 'rgba(255, 255, 255, 0.05)');
    document.documentElement.style.setProperty('--text-color', 'white');
  } else {
    document.documentElement.style.setProperty('--background-color', '#e8ecfb');
    document.documentElement.style.setProperty('--text-color', 'black');
  }

  const preload = [
    "https://static.tildacdn.com/tild3438-3830-4232-a335-386262333166/Group_18541.svg"
  ];
  
  //
  // Для этого теста эти прелоды не нужны
  //
  
  let questions = data.QUESTIONS;

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

  // Предзагружаем картинки из вопросов и подставляем img-теги вместо ::-смайликов
  data.QUESTIONS.forEach((q, qIndex) => {
    if (q.hasOwnProperty("questionImageUrl"))
      preload.push(q.questionImageUrl)

    if (q.type && q.type == 'horizontal-with-images') {
      q.answers.forEach((a, aIndex) => {
        preload.push(a.imageUrl)
      })
    }
  })
  preloadImages(preload);
  
  let currentQuestionIndex = 0;
  let timer = '05:00'

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
  data.QUESTIONS.forEach((q,index) => {
    data.QUESTIONS[index]['startAnswerIndex'] = startIndex;
    startIndex += q.answers.length
  })

  let userAnswers = {};
  let answerBlock = false;
  function writeAnswer(index) {
    if (answerBlock) return;
    answerBlock = true;
    
    setTimeout(() => { // Меняем вопрос с задержкой
      userAnswers[currentQuestionIndex] = {
        questionLink: question,
        questionIndex: currentQuestionIndex,
        answerIndex: index,
        // Ответы у нас в одном едином массиве. Например, третий ответ третьего вопроса будет под индексом 8
        // Чтобы потом удобно по нему пробегаться, нам нужно записать общий индекс ответа
        overallAnswerIndex: ( question.startAnswerIndex + index ?? null)
      }

      console.log(userAnswers)
      answerBlock = false;
      
      // Скролл до начала теста  
      let thisProftestOffset = cumulativeOffset(document.querySelector('.z-skypro-proftest-wrapper'))
      let currentTopOffset = document.documentElement.scrollTop || document.body.scrollTop;
      if (currentTopOffset > thisProftestOffset.top)
        window.scrollTo({ top: thisProftestOffset.top, behavior: 'smooth' });

      window.dispatchEvent(
        new CustomEvent("answerPressed", { detail: {
          questionLink: questions[currentQuestionIndex],
          questionIndex: currentQuestionIndex,
          answerIndex: index,
          overallAnswerIndex: questions.startAnswerIndex + index
        }})
      );
      
      if (currentQuestionIndex + 1 !== questions.length) {
        next()
      }
      else {
        proftestCompleted()
      }

    }, ((import.meta.env.DEV) ? 0 : 100));
  }

  function proftestCompleted() {
    
    // debugger;
    // подсчитаем общие баллы по каждой профессии
    for (const key in userAnswers) {
      let a = userAnswers[key]
      data.RESULT_PROFESSION.forEach(prof => {
        prof.userProggress += prof.dataCheck[a.overallAnswerIndex] || 0
      })
    }

    // Отсортируем
    data.RESULT_PROFESSION.sort((a, b) => b.userProggress - a.userProggress);
    console.log(data.RESULT_PROFESSION);
    // Отчитываемся
    var evt = new CustomEvent("proftestCompleted", { detail: {
      answers: userAnswers,
      professions: data.RESULT_PROFESSION
    }});
    window.dispatchEvent(evt);
  }

  let question = null;
  $: question = questions[currentQuestionIndex]

  let currentProgress = 0;
  $: currentProgress = Math.round(currentQuestionIndex / questions.length * 100)

  let screenWidth;

  // таймер
  let duration = 5 * 60 * 1000;
  let startTime = new Date().getTime();
  let timerInterval = setInterval(function () {
    let now = new Date().getTime();
    let elapsedTime = now - startTime;

    if (elapsedTime >= duration) {
        clearInterval(timerInterval);
        timer = ''
    } else {
      let remainingTime = duration - elapsedTime;

      let minutes = Math.floor(
        (remainingTime % (1000 * 60 * 60)) / (1000 * 60)
      );
      let seconds = Math.floor((remainingTime % (1000 * 60)) / 1000);

      let formattedMinutes = minutes.toString().padStart(2, "0");
      let formattedSeconds = seconds.toString().padStart(2, "0");

      timer = formattedMinutes + ":" + formattedSeconds
    }
  }, 1000);

  let switching = false;
  let beforeSwitching = false; 
  function back() {
    switching = true;
    setTimeout(() => {
      switching = false;
      currentQuestionIndex--
    }, 500)
  }

  function next() {
    switching = true;
    setTimeout(() => {
      switching = false;
      beforeSwitching = false;
      currentQuestionIndex++;
    }, 500)
  }

</script>

<!-- Для рендеринга на основе размера окна -->
<svelte:window bind:innerWidth={screenWidth} />

<div class="z-skypro-proftest-main-wrapper">
  <div class="z-proftest-header">
    <div class="z-proftest-header__logo" style={`background-image: url('https://static.tildacdn.com/tild6635-6465-4766-a263-316539366231/Group_1321316717.svg')`}></div>
    {#if timer !== ''}
    <div class="z-proftest-header__timer">{timer}</div>
    {/if}
  </div>
  <div class="z-skypro-proftest-wrapper">
    {#if typeof question.subtitle === 'undefined' && (!question.type || question.type == 'classic')}
      <!-- Разметка для обычных вопросов -->
      {#if !question.questionImageUrl}
      <div class={`z-skypro-proftest-wrapper__header-wrap`}>
          <div class="z-skypro-proftest-wrapper__header">
            Пройдите научный профориентационный тест и узнайте, какой профессии подходите
          </div>
          <div class="z-skypro-proftest-wrapper__5material">
            <div class="z-skypro-proftest-wrapper__5material-text">
              В конце — 5 полезных материалов, которые помогут в работе уже сейчас: <span>советы по резюме, собеседованиям и росту зарплаты</span>
            </div>
            <img alt='подарок' src="https://static.tildacdn.com/tild3164-3237-4662-a163-333965636565/Group_1597880733_1.png">
          </div>
      </div>
      {/if}

      
      {#if question.questionImageUrl}
      <div style={`background-image: url(${question.questionImageUrl})`} class={`z-skypro-proftest-wrapper__question-image`}></div>
      {/if}
      
      <div class={`z-skypro-proftest-main-wrap classicQuestionsWrap`}>
          <div class="z-skypro-proftest">
              <div class={`classicQuestionHeader`}>
                <h3 class="z-skypro-proftest__question">{question.question}</h3>
                <span class="z-skypro-proftest__progress">{currentProgress}%</span>
              </div>
              
              <div class={`z-skypro-proftest__answers`}>
                {#key question}
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
                    <div class="z-skypro-proftest__answer-text">{answer}</div>
                    <div class="z-skypro-proftest__answer-checkbox"></div>
                  </div>
                {/each}
                {/key}
              </div>

              <div class="z-skypro-proftest__buttons">
                {#if currentQuestionIndex > 0 && !beforeSwitching}
                <button on:click={() => back() } class="z-skypro-proftest__button z-skypro-proftest__button--prev">
                    Назад
                </button>
                {/if}
                {#if userAnswers[currentQuestionIndex] && !switching && !beforeSwitching}
                <button on:click={() => next() } class="z-skypro-proftest__button z-skypro-proftest__button--next">
                    Далее
                </button>
                {/if}
              </div>
          </div>
          
      </div>
    {/if}
  </div>
</div>

<style>
  
</style>