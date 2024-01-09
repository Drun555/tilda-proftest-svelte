<script>
  import Quiz from './lib/Quiz.svelte'

  if (import.meta.env.PROD) {
    document.querySelector(window.luigiQuiz.thisBlock).style.display = "none";
    document.querySelector(window.luigiQuiz.finalBlock).style.display = "none";

    document.querySelector(window.luigiQuiz.startClass).addEventListener('click', () => {
      document.querySelector(window.luigiQuiz.thisBlock).style.display = "block";
      document.querySelector(window.luigiQuiz.startBlock).style.display = "none";

      try {
        dataLayer.push({
          'event' : 'GTMevent',
          'eventCategory' : 'button',
          'eventAction' : 'click',
          'eventLabel' : 'first_page'
        });
      } catch {
        console.warn('error dataLayer')
      }
        
    })
  }

  if (import.meta.env.DEV) {
    console.log('Мы в деве')
  }

</script>

<main>
  <Quiz />
</main>

<style>
</style>
