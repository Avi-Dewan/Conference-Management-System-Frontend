<script>
  import Navbar from "/src/components/navbar.svelte";
  import { goto } from "$app/navigation";

  function handleSubmit() {
    let user_id = 1;
    goto(`/${user_id}/home`);
  }
</script>

<main>
  <button on:click={handleSubmit}></button>
</main>

<style>
  main {
    max-width: 90%;
    margin: 2% auto;
  }

  h1 {
  }
</style>
