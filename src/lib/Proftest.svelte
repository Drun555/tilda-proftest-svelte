<script>
  // @ts-nocheck

  import '../assets/styles.scss'
  import { cumulativeOffset } from './helpers'
  let data = window.proftest;
  
  let blackTheme = true;
  $: if (blackTheme) {
    document.documentElement.style.setProperty('--background-color', 'rgba(32, 34, 41, 0.6)');
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
      
      if ( professionalQuestions == null ) {
        if (currentQuestionIndex + 1 !== questions.length) {
          next()
        }
        else {
          proftestCompleted()
        }
      } else {
        beforeSwitching = true;
        setTimeout(next, 3000)

        if (currentQuestionIndex + 1 == questions.length) {
          professionalTestCompleted()
        }
      }

    }, ((professionalQuestions) ? 0 : 100));
  }

  function professionalTestCompleted() {
    currentTrueAnswers = 0;
    professionalCompleted = true;

    for (const key in userAnswers) {
      let a = userAnswers[key]

      if (a.questionLink.answers[a.answerIndex])
        currentTrueAnswers++;
    }

    window.dispatchEvent(
      new CustomEvent("professionalTestCompleted", { detail: {
        answers: userAnswers,
        profession: professionalQuestions
      }})
    );
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

  let currentTrueAnswers = 0;
  let professionalQuestions = null;
  let professionalCompleted = false;
  window.addEventListener('formFilled', (e) => {
    // Переключаемся на белую тему
    blackTheme = false;
    professionalCompleted = false;
    currentTrueAnswers = 0;
    userAnswers = {}
    currentQuestionIndex = 0;
    professionalQuestions = e.detail;
    questions = data.SPECIALIZED_QUESTIONS[professionalQuestions].questions
    questions.forEach(q => {
      q.startAnswerIndex = startIndex;
      startIndex += q.answers.length
    })
  })

  // Сразу переходим в профессиональные вопросы
  // var evt = new CustomEvent("formFilled", { detail: "Разработчик" });
  // window.dispatchEvent(evt);

  let currentProfessionalAnswer = null;
  $: try {
    let currentAnswerIndex = userAnswers[currentQuestionIndex].answerIndex;
    currentProfessionalAnswer = data.SPECIALIZED_QUESTIONS[professionalQuestions].questions[currentQuestionIndex].answers[currentAnswerIndex]
  } catch {
    currentProfessionalAnswer = {
      text: 'null',
      imageUrl: null,
      trueAnswer: false,
      hintAfter: 'error',
    }
  }

  let switching = false;
  let beforeSwitching = false; 
  function back() {
    switching = true;
    setTimeout(() => {
      switching = false;
      currentQuestionIndex--
    }, 320)
  }

  function next() {
    switching = true;
    setTimeout(() => {
      switching = false;
      beforeSwitching = false;
      currentQuestionIndex++;
    }, 320)
  }

</script>

<!-- Для рендеринга на основе размера окна -->
<svelte:window bind:innerWidth={screenWidth} />

<div class="z-skypro-proftest-main-wrapper">
  
  <div class="z-proftest-header">
    <div class="z-proftest-header__logo" style={`background-image: url('https://static.tildacdn.com/tild3438-3830-4232-a335-386262333166/Group_18541.svg')`}></div>
    <div class="z-proftest-header__timer">{timer}</div>
  </div>
  
  <!-- 
  <div class="z-proftest-second-header">
    <div class="z-proftest-second-header-h1">
      Пройдите тест и узнайте
      {#if screenWidth > 960}
      <br/>
      {/if}
      свой реальный уровень зарплаты
    </div>
    {#if screenWidth >= 1260}
    <div class="z-proftest-second-header-badge">
      <img src="https://static.tildacdn.com/tild6635-3332-4665-a266-636332653530/Icon.png" alt="">
      Подарим разбор ваших навыков и&nbsp;персональную консультацию по развитию карьеры
    </div>
    {/if}
    
  </div>
  -->
  <div class="z-skypro-proftest-wrapper">
    {#if professionalQuestions}
      <!-- Разметка для профессиональных вопросов -->
      <div class={`leftSide opacity100 ${switching ? 'opacity0' : 'opacity100'}`}>
        <div class="professionalQuestions-title">
          {@html data.SPECIALIZED_QUESTIONS[professionalQuestions].title }
        </div>
        <div class="professionalQuestions-subtitle">
          {@html data.SPECIALIZED_QUESTIONS[professionalQuestions].subtitle }
        </div>
        
        {#if question}
          {#if question.leftText}
            <div class="professionalQuestions-leftText">
              {@html question.leftText}
            </div>
          {/if}
          
          {#if question.imageAfterText}
            <img alt='' class="professionalQuestions-imageAfterText" src={(question.mobileImageAfterText && screenWidth < 1260) ? question.mobileImageAfterText : question.imageAfterText} />
          {/if}
          
          {#if question.greyTextAfter}
          <div class="professionalQuestions-greyTextAfter">
            {@html question.greyTextAfter}
          </div>
          {/if}
        {/if}
      </div>
      <div class={`rightSide opacity100 ${switching ? 'opacity0' : 'opacity100'}`}>
        <div class="z-skypro-proftest-main-wrap">
          {#if question}
          <div class="z-skypro-proftest">
              <h3 class="z-skypro-proftest__question">{question.rightTextBeforeAnswers}</h3>
              <div class={`z-skypro-proftest__answers ${question.onlyImages ? 'imagesBlock' : ''}`}>
                {#each question.answers as answer, index}
                  <!-- svelte-ignore a11y-click-events-have-key-events a11y-no-static-element-interactions-->
                  <div 
                    style={
                      (answer.imageUrl ? `background-image: url('${answer.imageUrl}') ` : '')
                      +
                      (answer.backgroundColor ? `background-color: ${answer.backgroundColor} ` : '')
                    }
                    class={
                      (
                        (userAnswers[currentQuestionIndex] && userAnswers[currentQuestionIndex].answerIndex !== null) 
                        ? 
                        `z-skypro-proftest__answer z-skypro-proftest__answer--${answer.trueAnswer}` 
                        : 
                        `z-skypro-proftest__answer`
                      )
                      +
                      ' '
                      +
                      (
                        (userAnswers[currentQuestionIndex] && userAnswers[currentQuestionIndex].answerIndex == index)
                        ?
                        `z-skypro-proftest__answer--selected`
                        :
                        ''
                      )
                    } 
                    
                    on:click={function(e) { writeAnswer(index) }}>
                    <div class="z-skypro-proftest__answer-text">{answer.text}</div>
                    <div class="z-skypro-proftest__answer-checkbox">
                      {#if answer.imageUrl || answer.backgroundColor}
                        <div class="z-skypro-proftest__answer-checkbox-background"></div>
                      {/if}
                    </div>
                  </div>
                {/each}
              </div>            
          </div>
          {:else}

          <div class="professionalQuestions-finalTitle">
            {@html data.SPECIALIZED_QUESTIONS[professionalQuestions].finalTitle}
          </div>
          {#if data.SPECIALIZED_QUESTIONS[professionalQuestions].finalImage}
            <img alt='' class="professionalQuestions-finalImage" src={data.SPECIALIZED_QUESTIONS[professionalQuestions].finalImage} />
          {/if}

          {/if}
          {#if !question && currentTrueAnswers < 2}
          <div class={`answer-hint answer-hint-false`}>
            <img src="https://static.tildacdn.com/tild3263-3238-4436-b661-313132336239/Icon.svg" alt="">
            <div>
              {@html data.SPECIALIZED_QUESTIONS[professionalQuestions].finalRedBlock}
            </div>
          </div>
          {:else if !question && currentTrueAnswers >= 2}
          <div class={`answer-hint answer-hint-true`}>
            <img src="https://static.tildacdn.com/tild3263-3238-4436-b661-313132336239/Icon.svg" alt="">
            <div>
              {@html data.SPECIALIZED_QUESTIONS[professionalQuestions].finalGreenBlock}
            </div>
          </div>
          {/if}
          {#if userAnswers[currentQuestionIndex] && userAnswers[currentQuestionIndex].answerIndex !== null}
          <div class={`answer-hint answer-hint-${currentProfessionalAnswer.trueAnswer}`}>
            <img src="https://static.tildacdn.com/tild3263-3238-4436-b661-313132336239/Icon.svg" alt="">
            <div>
              {currentProfessionalAnswer.hintAfter}
            </div>
          </div>
          {/if}
        </div>
      </div>

    {:else if typeof question.subtitle === 'undefined' && (!question.type || question.type == 'classic')}
      <!-- Разметка для обычных вопросов -->
      {#if !question.questionImageUrl}
      <div class={`z-skypro-proftest-wrapper__header-wrap`}>
          <p class="z-skypro-proftest-wrapper__header">
            Подходит ли вам IT-сфера?
          </p>
          <div class={`z-skypro-proftest-wrapper__subtitle`}>
            Пройдите тест и узнайте, получится ли у вас работать в IT
          </div>
          <div class="z-skypro-proftest-wrapper__giftBox">
            <img alt='подарок' src="https://static.tildacdn.com/tild6636-3238-4332-b831-613138613336/Group_1597880045.png">
            <div class="z-skypro-proftest-wrapper__giftBox-text">
              В конце подарим бесплатную консультацию с карьерным экспертом и книгу «Первые шаги в IT» для начинающих
            </div>
          </div>
      </div>
      {/if}

      
      {#if question.questionImageUrl}
      <div style={`background-image: url(${question.questionImageUrl})`} class={`z-skypro-proftest-wrapper__question-image ${switching ? 'opacity0' : 'opacity100'}`}></div>
      {/if}
      
      <div class={`z-skypro-proftest-main-wrap classicQuestionsWrap ${switching ? 'opacity0' : 'opacity100'}`}>
          <div class="z-skypro-proftest">
              <div class="classicQuestionHeader">
                <h3 class="z-skypro-proftest__question">{question.question}</h3>
                <span class="z-skypro-proftest__progress">{currentProgress}%</span>
              </div>
              
              <div class="z-skypro-proftest__answers">
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
          </div>
      </div>
    {/if}
  </div>
  <!-- 
  {#if screenWidth < 1260}
    <div class="z-proftest-second-header-badge">
      <img src="https://static.tildacdn.com/tild6635-3332-4665-a266-636332653530/Icon.png" alt="">
      Подарим разбор ваших навыков и&nbsp;персональную консультацию по развитию карьеры
    </div>
  {/if}
  -->
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
    {:else if professionalCompleted && !beforeSwitching}
    <button 
      on:click={() => {
        window.dispatchEvent(
          new CustomEvent("formFilled", { detail: professionalQuestions })
        );
      }} 
      class="z-skypro-proftest__button z-skypro-proftest__button--next"
    >
      Пройти заново
    </button>
    {/if}
    
  </div>
</div>

<style>
  :root {
    --color-grey: #ebebeb;
    --color-grey2: #bcbcbc;
    --color-white: #ffffff;
    --color-purple: #7334ea;
    --color-green: #bcec30;
    --color-black: #000000;
    --blur: blur(10px);
    font-family: StratosSkyeng;
    width: 100%;
  }

  :global(.inline-emoji) {
    height: 1em;
    padding-right: 10px;
    transform: translateY(3px);
  }

  .z-proftest-header {
    width: 100%;
    display: flex;
    flex-direction: row;
    justify-content: space-between;
    align-items: center;
    width: -webkit-fill-available;
    width: -moz-available;
    padding-bottom: 30px;
    padding-top: 21px;
  }

  .z-proftest-header__timer {
    font-size: 24px;
    padding: 10px;
    border-radius: 10px;
    background: var(--background-color);
    /* клёвый блюр */
    /* background: rgba(255, 255, 255, 0.6); */
    /* backdrop-filter: blur(80px); */
    color: var(--text-color);
  }

  .z-proftest-header__logo {
    background-size: auto;
    width: 100%;
    background-repeat: no-repeat;
    background-position: left center;
    height: 35px;
  }

  :global(.spectrum-circle-selected) {
    background-color: currentColor !important;
    transition: 0.5s;
  }

  :global(.spectrum-numbers .spectrum-circle-selected) {
    background-color: rgb(188, 147, 255) !important;
    transition: 0.5s;
  }

  @media (hover: hover) {
  }

  .z-skypro-proftest-main-wrapper {
    overflow: hidden;
    padding-top: 50px;
    padding-left: 20px;
    padding-right: 20px;
    position: relative;
    display: flex;
    justify-content: center;
    align-items: center;
    flex-direction: column;
    width: 1260px;
    justify-content: center;
  }

  :global(.z-skypro-proftest__answer--selected .z-skypro-proftest__modern-answer) {
    border-color: var(--color-purple) !important;
  }

  .z-proftest-second-header {
    width: 100%;
    padding-bottom: 30px;
    display: flex;
    justify-content: space-between;
    flex-direction: row;
  }

  .z-proftest-second-header-badge {
    background: linear-gradient(266deg, #9EA9DD -5.4%, #5E6AA4 111.87%);
    width: 640px;
    padding-top: 20px;
    padding-bottom: 20px;
    border-radius: 20px;
    background-blend-mode: overlay;
    background-position-x: -10px;
    background-position-y: -25px;
    background-size: 140%;
    display: flex;
    align-items: center;
    gap: 20px;
    color: var(--text-color);
    font-size: 21px;
    font-style: normal;
    line-height: 115%;
    font-weight: 300;
  }

  .z-proftest-second-header-badge img {
    padding-left: 20px;
  }
  .z-proftest-second-header-h1 {
    font-size: 32px;
    font-weight: 500;
  }

  @media screen and (max-width: 1260px) {
    .z-proftest-second-header-h1 {
      font-size: 20px;
    }
    
    .z-skypro-proftest__answers {
      margin-top: 0;
    }

    .z-proftest-second-header {
      flex-direction: column;
      gap: 30px;
      padding-bottom: 20px;
    } 

    .z-proftest-second-header-badge {
      width: 100%;
      font-size: 14px;
      font-weight: 300;
      padding-right: 19px;
      box-sizing: border-box;
      gap: 15px;
      padding: 16px;
      line-height: 120%;
      margin-top: 10px;
    }

    .z-proftest-second-header-badge img {
      width: 30px;
      padding-left: 3px;
    }

    .z-skypro-proftest-wrapper {
      height: unset;
      min-height: unset;
      padding: 20px;
    } 

    .z-skypro-proftest-main-wrapper {
      width: 100%;
    }

    .z-proftest-header {
      padding-bottom: 15px;
    }

    .z-proftest-header__logo {
      background-position: left;
      width: unset;
    }

    .z-proftest-header__timer {
      font-size: 24px;
    }

    .z-proftest-header__logo {
      background-size: auto;
      width: 100%;
      background-repeat: no-repeat;
      height: 35px;
    }

    :global(.z-skypro-proftest__answer--selected .z-skypro-proftest__modern-answer) {
      border-color: var(--color-purple) !important;
    }
  }

  @media screen and (max-width: 640px) {
    

    .z-skypro-proftest-wrapper {
      width: calc(100%);
      height: unset;
      min-height: unset;
      padding: 20px;
    } 

    .fourEmojis {
      grid-template-columns: 1fr 1fr;
      grid-template-rows: 1fr 1fr;
    }

    .z-proftest-header__timer {
      font-size: 16px;
      display: none;
    }

    .z-proftest-header {
      padding: 0;
      height: 75px;
    }

    .z-proftest-header__logo {
      background-size: 145px;
    }
    .z-skypro-proftest-main-wrapper {
      width: calc(100%);
      padding: 13px;
      padding-top: 10px;
    }
  }
  
</style>