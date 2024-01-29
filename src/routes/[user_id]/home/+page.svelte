<script>
  import NavbarChair from "/src/components/navbar_chair.svelte";
  import NavbarUser from "/src/components/navbar_user.svelte";
  import { page } from "$app/stores";
  import { onMount } from "svelte";

  let user_id, user;

  user_id = $page.params.user_id;

  onMount(async () => {
    try {
      let url = `http://localhost:3000/user/${user_id}`;

      const response = await fetch(url);

      if (!response.ok) {
        throw new Error("Failed to fetch data");
      }

      user = await response.json();
      console.log(user, "in home");

      sessionStorage.setItem("user_type", user.user_type);
    } catch (error) {
      console.error("Error fetching data:", error);
    }
  });
</script>

{#if user != null}
  <main>
    {#if user.user_type == "chair"}
      <NavbarChair />
    {:else}
      <NavbarUser />
    {/if}

    <h1>Welcome to Conference Management System</h1>
    <p>
      Lorem ipsum dolor, sit amet consectetur adipisicing elit. Unde ipsum,
      numquam aspernatur earum dolore, libero pariatur voluptatibus itaque qui
      illo alias, tempore ullam? Perferendis mollitia obcaecati pariatur
      molestias exercitationem. Nam?
    </p>
  </main>
{/if}

<style>
  main {
    max-width: 90%;
    margin: 2% auto;
  }

  h1 {
  }
</style>
