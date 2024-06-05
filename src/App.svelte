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
      background-color: #10121a;
      background-image: url('https://static.tildacdn.com/tild6263-6330-4165-a663-663864333765/2_81-2-2.png');
      background-size: auto;
      background-position: center;
      background-repeat: repeat;
      overflow: visible;
      padding-left: 10px;
      margin-left: -10px;
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
