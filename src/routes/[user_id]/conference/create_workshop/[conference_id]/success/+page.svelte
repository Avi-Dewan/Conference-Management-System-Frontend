<script>
  import { createClient } from "@supabase/supabase-js";
  import NavbarChair from "/src/components/navbar_chair.svelte";
  import NavbarUser from "/src/components/navbar_user.svelte";

  import { page } from "$app/stores";

  import { onMount } from "svelte";

  import { goto } from "$app/navigation";

  let user_id = $page.params.user_id;
  let conference_id = $page.params.conference_id;

  function goBack() {
    goto(`/${user_id}/conference/view_workshop/${conference_id}`);
  }

  let unreadCount = null;
  onMount(async () => {
    try {
      user_type = sessionStorage.getItem("user_type");

      const unreadNotificationCount = await fetch(
        `http://localhost:3000/notification/unreadCount/${user_id}`
      );

      unreadCount = await unreadNotificationCount.json();
      unreadCount = unreadCount.unreadCount;
    } catch (error) {
      console.error("Error fetching data:", error);
    }
  });
</script>

<main>
  {#if unreadCount != null}
    <NavbarChair myVariable={unreadCount} />
    <div>
      <div class="success-dialog">
        <h3 style="color: black;">Success! Your workshop created.</h3>

        <button on:click={goBack}>Ok</button>
      </div>
    </div>
  {/if}
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

  main {
    max-width: 90%;
    margin: 2% auto;
  }

  h1 {
  }
</style>
