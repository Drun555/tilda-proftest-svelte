<script>
  import Proftest from './lib/Proftest.svelte'

  if (import.meta.env.DEV) {
    console.log('Мы в деве')
  }

</script>

<main>
  <Proftest />
</main>

<style>
</style>
