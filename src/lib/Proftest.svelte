<script>
  // @ts-nocheck

  import '../assets/styles.scss'

  let data = window.proftest;
  
  const preload = [
    "https://static.tildacdn.com/tild6635-6465-4766-a263-316539366231/Group_1321316717.svg"
  ];
  
  const threeEmojis = [
    "https://static.tildacdn.com/tild3439-6435-4632-b630-616661646362/photo.png", // Да
    "https://static.tildacdn.com/tild3835-6234-4566-b536-323165663563/photo.png", // Отчасти
    "https://static.tildacdn.com/tild6530-6533-4562-a235-623262343263/photo.png", // Нет
    
  ]

  const fourEmojis = [
    "https://static.tildacdn.com/tild3438-6336-4061-b038-313836613539/photo.png", // Flatlined
    "https://static.tildacdn.com/tild6635-3264-4363-a534-663966343933/photo.png", // Без лица
    "https://static.tildacdn.com/tild6166-3330-4662-b434-336561633731/photo.png", // Happy
    "https://static.tildacdn.com/tild6233-3864-4036-b136-653437383839/photo.png", // Lovely
  ]

  const inlineEmojis = {
    moneybag: "https://static.tildacdn.com/tild6661-3564-4439-b038-356366343038/money-bag_1f4b0.png", // Moneybag
    seeding: "https://static.tildacdn.com/tild6166-3431-4262-a337-396664383062/seedling_1f331.png" // Seeding
  }
  preload.push(...threeEmojis)
  preload.push(...fourEmojis)
  preload.push(...Object.values(inlineEmojis))

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

        Object.keys(inlineEmojis).forEach(key => {
          if ( a.text.indexOf(`:${key}:`) !== -1 )
          data.QUESTIONS[qIndex].answers[aIndex].text = a.text.replaceAll(`:${key}:`, `<img class='inline-emoji' src='${inlineEmojis[key]}' />`)
        })
        
      })
    } else {
      q.answers.forEach((a, aIndex) => {
        Object.keys(inlineEmojis).forEach(key => {
          if ( a.indexOf(`:${key}:`) !== -1 )
            data.QUESTIONS[qIndex].answers[aIndex] = a.replaceAll(`:${key}:`, `<img class='inline-emoji' src='${inlineEmojis[key]}' />`)
        })
      })
    }
  })
  preloadImages(preload);
  
  let currentQuestionIndex = 0;
  let timer = '05:00'


  // Новые типы вопросов ломают всё, что можно сломать. Автоматически добавим туда 5 или 3 выбора ответа.
  data.QUESTIONS.forEach(q => {
    if (q.type && q.type == "spectrum" && q.answers.length == 0)
      q.answers = ["1","2","3","4","5"]
    if (q.type && q.type == "emoji" && q.answers.length == 0)
      q.answers = ["Да","Отчасти","Нет"]
  })

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
  let answerBlock = false;
  function writeAnswer(index) {
    if (answerBlock) return;
    answerBlock = true;
    
    setTimeout(() => { // Меняем вопрос с задержкой
      userAnswers[currentQuestionIndex] = {
        questionLink: data.QUESTIONS[currentQuestionIndex],
        questionIndex: currentQuestionIndex,
        answerIndex: index,
        // Ответы у нас в одном едином массиве. Например, третий ответ третьего вопроса будет под индексом 8
        // Чтобы потом удобно по нему пробегаться, нам нужно записать общий индекс ответа
        overallAnswerIndex: data.QUESTIONS[currentQuestionIndex].startAnswerIndex + index
      }
      answerBlock = false;
      window.scrollTo({ top: 0, behavior: 'smooth' });
      window.dispatchEvent(
        new CustomEvent("answerPressed", { detail: {
          questionLink: data.QUESTIONS[currentQuestionIndex],
          questionIndex: currentQuestionIndex,
          answerIndex: index,
          overallAnswerIndex: data.QUESTIONS[currentQuestionIndex].startAnswerIndex + index
        }})
      );

      if (currentQuestionIndex +1 !== data.QUESTIONS.length)
        currentQuestionIndex++
      else
        proftestCompleted()

    }, ((import.meta.env.PROD) ? 500 : 500));
  }

  function proftestCompleted() {

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
      professions: data.RESULT_PROFESSION,
      usedData: data
    }});
    window.dispatchEvent(evt);
  }

  let question = null;
  $: question = data.QUESTIONS[currentQuestionIndex]

  let currentProgress = 0;
  $: currentProgress = Math.round(currentQuestionIndex / data.QUESTIONS.length * 100)

  let screenWidth;

  // Нажатие на кругляши
  document.body.addEventListener('click', function (evt) {
      if (evt.target.className.includes('spectrum-circle')) {
        document.querySelectorAll('.spectrum-circle-selected').forEach(s => s.classList.remove('spectrum-circle-selected'))
        evt.target.classList.add('spectrum-circle-selected')
        writeAnswer(evt.target.dataset.index)
      }
  }, false);

  // таймер
  let duration = 5 * 60 * 1000;
  let startTime = new Date().getTime();
  let timerInterval = setInterval(function () {
      let now = new Date().getTime();
      let elapsedTime = now - startTime;

      if (elapsedTime >= duration) {
          clearInterval(timerInterval);
          timer = '00:00'
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
</script>

<!-- Для рендеринга на основе размера окна -->
<svelte:window bind:innerWidth={screenWidth} />

<div class="z-skypro-proftest-main-wrapper">
  <div class="z-proftest-header">
    <div class="z-proftest-header__logo" style={`background-image: url('https://static.tildacdn.com/tild6635-6465-4766-a263-316539366231/Group_1321316717.svg')`}></div>
    <div class="z-proftest-header__timer">{timer}</div>
  </div>
  {#key currentQuestionIndex}
  <div class="z-skypro-proftest-wrapper">
    {#if !question.type || question.type == 'classic'}
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
      
      <div class="z-skypro-proftest-main-wrap classic">
          <div class="z-skypro-proftest">
              <span class="z-skypro-proftest__progress">{currentProgress}%</span>
              <h3 class="z-skypro-proftest__question">{question.question}</h3>
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
          </div>
      </div>
    {/if}
    {#if question.type == "horizontal-with-images"}
      <div class="z-skypro-proftest-wrapper-modern">
        <div class="z-skypro-proftest-wrapper__header">
          {question.question}
        </div>
        <div class="z-skypro-proftest-wrapper-modern-questions">
          {#each question.answers as answer, index}
          <!-- svelte-ignore a11y-click-events-have-key-events a11y-no-static-element-interactions-->
          <div class="z-skypro-proftest-wrapper-modern-question-answer">
            <div 
              style={`background-image: url(${answer.imageUrl})`} 
              class="z-skypro-proftest-wrapper-modern-question-image" 
              on:click={function(e) {
                document.querySelectorAll('.z-skypro-proftest__answer--selected').forEach(s => s.classList.remove('z-skypro-proftest__answer--selected'))
                e.currentTarget.classList.add('z-skypro-proftest__answer--selected')
                writeAnswer(index)
              }}
            >
              <div class="z-skypro-proftest__modern-answer__wrapper">
                <div 
                  class={
                    (userAnswers[currentQuestionIndex] && userAnswers[currentQuestionIndex].answerIndex == index) 
                    ? 
                    "z-skypro-proftest__modern-answer z-skypro-proftest__answer--selected" 
                    : 
                    "z-skypro-proftest__modern-answer"
                  } 
                >
                  <div>{answer.text}</div>
                  <div class="z-skypro-proftest__modern-answer-checkbox"></div>
                </div>
              </div> 
            </div>
          </div>
          
          {/each}
        </div>
      </div>
    {/if}
    {#if question.type == "spectrum"}
      <div class="z-skypro-proftest-wrapper-modern">
        <div class="z-skypro-proftest-wrapper__header">
          {question.question}
        </div>
        {#if screenWidth > 1160}
        <div class="spectrum-wrapper">
          <div>Не про меня</div>
          <div class='spectrum-circle' data-index="0" style="color: rgba(111, 228, 255, 1); width: 150px; height: 150px"></div>
          <div class='spectrum-circle' data-index="1" style="color: rgba(111, 228, 255, 1); width: 110px; height: 110px"></div>
          <div class='spectrum-circle' data-index="2" style="color: rgba(158, 169, 221, 1); width: 80px; height: 80px"></div>
          <div class='spectrum-circle' data-index="3" style="color: rgba(164, 108, 255, 1); width: 110px; height: 110px"></div>
          <div class='spectrum-circle' data-index="4" style="color: rgba(164, 108, 255, 1); width: 150px; height: 150px"></div>
          <div>Про меня</div>
        </div>
        {:else}
        <div class="spectrum-wrapper" style="padding-bottom: 10px; padding-top: 20px;">
          <div>Не про меня</div>
          <div>Про меня</div>
        </div>
        <div class="spectrum-wrapper">
          <div class='spectrum-circle' data-index="0" style="color: rgba(111, 228, 255, 1); width: 15vw; height: 15vw"></div>
          <div class='spectrum-circle' data-index="1" style="color: rgba(111, 228, 255, 1); width: 12vw; height: 12vw"></div>
          <div class='spectrum-circle' data-index="2" style="color: rgba(158, 169, 221, 1); width: 10vw; height: 10vw"></div>
          <div class='spectrum-circle' data-index="3" style="color: rgba(164, 108, 255, 1); width: 12vw; height: 12vw"></div>
          <div class='spectrum-circle' data-index="4" style="color: rgba(164, 108, 255, 1); width: 15vw; height: 15vw"></div>
        </div>
        {/if}
      </div>
    {/if}
    {#if question.type == "emoji"}
      <div class="z-skypro-proftest-wrapper-modern">
        <div class="z-skypro-proftest-wrapper__header">
          {question.question}
        </div>
        
        <div class="emoji-wrapper">
          <div class="emoji-allign">
            {#if question.answers.length == 3}
            <div class="emojiContainer threeEmojis">
              {#each question.answers as ans, emojiIndex}
              <div class="emoji-flex-wrapper">
                <div class='spectrum-circle emoji' style={`background-image: url('${threeEmojis[emojiIndex]}')`} data-index={emojiIndex}></div>
                <div class="emoji-flex-wrapper-text">{ans}</div>
              </div>  
              {/each}
            </div>
            {/if}
            {#if question.answers.length == 4}
            <div class="emojiContainer fourEmojis">
              {#each question.answers as ans, emojiIndex}
              <div class="emoji-flex-wrapper">
                <div class='spectrum-circle emoji' style={`background-image: url('${fourEmojis[emojiIndex]}')`} data-index={emojiIndex}></div>
                <div class="emoji-flex-wrapper-text">{ans}</div>
              </div>  
              {/each}
            </div>
            {/if}
          </div>
        </div>
      </div>
    {/if}
    {#if question.type == "modern-classic"}
      <div class="z-skypro-proftest-wrapper-modern">
        <div class="z-skypro-proftest-wrapper__header">
          {question.question}
        </div>
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
              <div class="z-skypro-proftest__answer-text">{@html answer}</div>
              <div class="z-skypro-proftest__answer-checkbox"></div>
            </div>
          {/each}
        </div>
      </div>
    {/if}
  </div>
  {/key}
  
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
</div>

<style>
  :root {
    --color-grey: #ebebeb;
    --color-grey2: #bcbcbc;
    --color-white: #ffffff;
    --color-purple: #7334ea;
    --color-green: #bcec30;
    --color-black: #000000;
    font-family: StratosSkyeng;
    width: 100%;
  }
  
  :global(.inline-emoji) {
    height: 1em;
  }

  .z-skypro-proftest__answer-text {
    display: flex;
    gap: 2px;
  }

  .emoji-allign {
    width: 100%;
  }

  .emojiContainer {
    display: grid;
    grid-auto-columns: auto;
    
    align-content: center;
    justify-content: center;
    margin: auto;
  }

  .threeEmojis {
    grid-template-columns: 1fr 1fr 1fr;
  }

  .fourEmojis {
    grid-template-columns: 1fr 1fr 1fr 1fr;
  }

  .emoji-flex-wrapper-text {
    white-space: break-spaces;
    text-align: center;
  } 

  .z-proftest-header {
    width: 100%;
    height: 100px;
    display: flex;
    flex-direction: row;
    justify-content: space-between;
    align-items: center;
    width: -webkit-fill-available;
    width: -moz-available;
    padding-left: 45px;
    padding-right: 45px;
  }

  .z-proftest-header__timer {
    font-size: 24px;
    color: white;
    padding: 10px;
    border-radius: 10px;
    background: rgba(32, 34, 41, 0.6);
  }

  .z-proftest-header__logo {
    background-size: auto;
    width: 100%;
    background-repeat: no-repeat;
    height: 35px;
  }

  .emoji-flex-wrapper {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 47px;
    width: 150px;
    margin: 0 auto;
  }

  .z-skypro-proftest-wrapper-modern .z-skypro-proftest-wrapper__header {
    width: 100%;
  }

  .emoji {
    color: rgba(164, 108, 255, 1); 
    width: 150px;
    height: 150px;
    background-size: 55%;
    background-repeat: no-repeat;
    background-position: center;
  }

  .spectrum-wrapper {
    width: 100%;
    display: flex;
    flex-direction: row;
    justify-content: space-between;
    align-items: center;
    color: white;
    font-size: 22px;
    text-wrap: nowrap;
    height: 400px;
  }

  .emoji-wrapper {
    width: 100%;
    display: flex;
    flex-direction: row;
    flex-wrap: wrap;
    align-content: center;
    align-items: center;
    color: white;
    font-size: 22px;
    height: 100%;
    min-height: 155px;
  }

  .spectrum-circle {
    border-radius: 99px;
    box-sizing: border-box;
    border: 2px currentColor solid;
    transition: 0.5s;
    background-color: transparent;
  }

  :global(.spectrum-circle-selected) {
    background-color: currentColor !important;
    transition: 0.5s;
  }

  @media (hover: hover) {
    .spectrum-circle:hover {
      border-radius: 99px;
      box-sizing: border-box;
      border: 10px currentColor solid;
      transition: 0.5s;
      cursor: pointer;
    }

    .z-skypro-proftest__modern-answer:hover {
      border-color: var(--color-grey2);
    }
  }

  
  
  .z-skypro-proftest-main-wrapper {
    overflow: hidden;
    padding-top: 10px;
    padding-left: 20px;
    padding-right: 20px;
    position: relative;
    display: flex;
    justify-content: center;
    align-items: center;
    flex-direction: column;
    width: 1160px;
    justify-content: center;
  }

  .z-skypro-proftest__modern-answer-checkbox {
    --color: var(--color-grey);
    --color-inner: var(--color-white);

    box-sizing: border-box;
    width: 16px;
    height: 16px;
    border-radius: 50%;
    border: solid 2px var(--color);

    display: flex;
    align-items: center;
    align-content: center;
    justify-content: center;
}

  .z-skypro-proftest__modern-answer-checkbox::after {
    content: "";
    position: absolute;
    width: 10px;
    height: 10px;
    border-radius: 50%;
    background-color: var(--color-inner);
  }

  :global(.z-skypro-proftest__answer--selected .z-skypro-proftest__modern-answer) {
    border-color: var(--color-purple) !important;
  }

  .z-skypro-proftest__answer--selected .z-skypro-proftest__modern-answer-checkbox {
    --color: var(--color-purple) !important;
    --color-inner: var(--color-purple) !important;
  }

  .z-skypro-proftest-wrapper-modern {
    width: 100%;
    height: 100%;
    display: flex;
    flex-direction: column;
    align-items: center;
  }

  .z-skypro-proftest__modern-answer {
    padding: 20px;
    border-radius: 10px;
    font-size: 20px;
    cursor: pointer;
    transition: border 0.2s ease;
    background: white;
    border: 1px var(--color-grey) solid;
    display: flex;
    align-content: center;
    align-items: center;
    justify-content: space-between;
  }

  

  .z-skypro-proftest-wrapper-modern-question-image {
    height: 370px;
    width: 100%;
    background-size: cover;
    background-position: center;
    background-repeat: no-repeat;
    display: flex;
    align-items: end;
    border-radius: 20px;
  }

  .z-skypro-proftest-wrapper-modern-question-image:hover {
    cursor: pointer;
  }
  .z-skypro-proftest-wrapper-modern-questions {
    display: flex;
    flex-direction: row;
    justify-content: space-between;
    gap: 20px;
    width: 100%;
  }

  .z-skypro-proftest__modern-answer__wrapper {
    position: relative;
    padding: 16px;
    width: 100%;
    height: fit-content;
  }
  .z-skypro-proftest-wrapper-modern-question-answer {
    width: 100%;
  }

  

  @media screen and (max-width: 1200px) {
    .z-skypro-proftest-wrapper {
      width: calc(100%);
      height: unset;
      min-height: unset;
      padding: 20px;
    } 

    .z-skypro-proftest-main-wrapper {
      width: 100%;
    }

    .z-skypro-proftest-wrapper-modern-question-image {
      height: 150px;
      background-size: contain;
      background-color: #f4f5f6;
      background-position-y: -10px;
    }

    .z-proftest-header {
      padding-bottom: 15px;
    }

    .z-proftest-header__logo {
      background-position: left;
      width: unset;
    }

    .spectrum-wrapper {
      width: 100%;
      font-size: 14px;
      font-weight: 400;
      padding-bottom: 20px;
      height: unset;
    }

    .emoji-flex-wrapper-text {
      font-size: 14px;
      font-weight: 400;
    }

    .emoji {
      width: 125px;
      height: 125px;
    }

    .emoji-flex-wrapper {
      gap: 10px;
      width: 125px;
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

    

    .z-skypro-proftest-wrapper-modern .z-skypro-proftest-wrapper__header {
      width: 100%;
    }

    .spectrum-circle {
      border-radius: 99px;
      box-sizing: border-box;
      border: 2px currentColor solid;
      transition: 0.5s;
      background-color: transparent;
    }

    :global(.spectrum-circle-selected) {
      background-color: currentColor !important;
      transition: 0.5s;
    }

    .z-skypro-proftest__modern-answer-checkbox {
      --color: var(--color-grey);
      --color-inner: var(--color-white);

      box-sizing: border-box;
      width: 16px;
      height: 16px;
      border-radius: 50%;
      border: solid 2px var(--color);

      display: flex;
      align-items: center;
      align-content: center;
      justify-content: center;
    }

    .z-skypro-proftest__modern-answer-checkbox::after {
      content: "";
      position: absolute;
      width: 10px;
      height: 10px;
      border-radius: 50%;
      background-color: var(--color-inner);
    }

    :global(.z-skypro-proftest__answer--selected .z-skypro-proftest__modern-answer) {
      border-color: var(--color-purple) !important;
    }

    .z-skypro-proftest__answer--selected .z-skypro-proftest__modern-answer-checkbox {
      --color: var(--color-purple) !important;
      --color-inner: var(--color-purple) !important;
    }

    .z-skypro-proftest-wrapper-modern {
      width: 100%;
      height: 100%;
      display: flex;
      flex-direction: column;
      align-items: center;
    }

    .z-skypro-proftest__modern-answer {
      padding: 12px;
      border-radius: 10px;
      font-size: 14px;
      cursor: pointer;
      transition: border 0.2s ease;
      background: white;
      border: 1px var(--color-grey) solid;
      display: flex;
      align-content: center;
      align-items: center;
      justify-content: space-between;
    }

    .z-skypro-proftest__modern-answer:hover {
      border-color: var(--color-grey2);
    }

    .z-skypro-proftest-wrapper-modern-questions {
      flex-direction: column;
      gap: 10px;
      width: 100%;
    }

    .z-skypro-proftest__modern-answer__wrapper {
      position: relative;
      padding: 16px;
      width: 100%;
      height: fit-content;
    }
    .z-skypro-proftest-wrapper-modern-question-answer {
      width: 100%;
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
    }

    .z-proftest-header {
      padding: 0;
      padding-left: 15px;
      height: 75px;
    }

    .z-proftest-header__logo {
      background-size: 175px;
    }
    .z-skypro-proftest-main-wrapper {
      width: calc(100%);
      padding: 13px;
      padding-top: 0;
    }

    .emoji-flex-wrapper {
      width: 80px;
    }

    .emoji {
      width: 80px;
      height: 80px;
    }
  }
</style>