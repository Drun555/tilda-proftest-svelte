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
      background-color: white;
      background-image: url('https://static.tildacdn.com/tild3365-6235-4731-a132-653135303436/Ellipse_2444_1.png');
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
    min-height: 90vh;
    overflow: hidden;
    padding-left: 20px;
    padding-right: 20px;
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
