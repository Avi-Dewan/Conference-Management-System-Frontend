<script>
  import { createClient } from "@supabase/supabase-js";
  import Navbar from "/src/components/navbar.svelte";

  import { page } from "$app/stores";

  import { onMount } from "svelte";

  import { goto } from "$app/navigation";

  let user_id = $page.params.user_id;
  let conference_id = $page.params.conference_id;
  function goBack() {
    goto(`/${user_id}/conference/conference_list/all`);
  }
</script>

<main>
  <div>
    <div class="success-dialog">
      <h3 style="color: black;">Success! Your File was submitted.</h3>

      <button on:click={goBack}>Ok</button>
    </div>
  </div>
</main>

<style>
  /* Styles for the button */
  button {
    padding: 10px 20px;
    background-color: black;
    color: #fff;
    border: none;
    border-radius: 5px;
    cursor: pointer;
    font-size: 16px;
    transition: background-color 0.3s ease;
  }

  button:hover {
    background-color: #2980b9;
  }

  /* Styles for the success dialogue box */
  .success-dialog {
    position: fixed;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    background-color: #2ecc71;
    color: white;
    padding: 20px;
    border-radius: 10px;
    box-shadow: 0 4px 8px 0 rgba(0, 0, 0, 0.2);
    z-index: 9999;
  }

  .success-dialog p {
    margin: 0;
  }

  /* Animation to fade in and out the success dialogue */
  .success-dialog.show {
    opacity: 0.5;
  }
</style>
