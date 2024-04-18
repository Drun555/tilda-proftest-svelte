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
