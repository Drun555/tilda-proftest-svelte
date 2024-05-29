<script>
  import Proftest from './lib/Proftest.svelte'

  if (import.meta.env.DEV) {
    function addStyle(styleString) {
      const style = document.createElement('style');
      style.textContent = styleString;
      document.head.append(style);
    }

    addStyle(`
    :root {
      background-color: black;
      background-image: url('https://static.tildacdn.com/tild3861-3062-4335-a562-303338373166/Frame_2043683093_2_1.png');
      background-size: auto;
      background-position: center;
      background-repeat: repeat;
    }
    `);

    console.log('Мы в деве')
  }
</script>

<main>
  <Proftest />
</main>

<style>
  main {
    overflow: hidden;
    position: relative;
    display: flex;
    align-items: center;
    flex-direction: column;
  }

  @media screen and (max-width: 1200px) {
    main {
      /* justify-content: center; */
    }
  }

  :global(body) {
    margin: 0;
  }
</style>
