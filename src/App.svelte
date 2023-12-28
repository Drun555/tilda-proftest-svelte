<script>
  import Quiz from './lib/Quiz.svelte'

  document.querySelector(window.luigiQuiz.thisBlock).style.display = "none";
  document.querySelector(window.luigiQuiz.finalBlock).style.display = "none";

  document.querySelector(window.luigiQuiz.startClass).addEventListener('click', () => {
    document.querySelector(window.luigiQuiz.thisBlock).style.display = "block";
    document.querySelector(window.luigiQuiz.startBlock).style.display = "none";
  })
</script>

<main>
  <Quiz />
</main>

<style>
</style>
