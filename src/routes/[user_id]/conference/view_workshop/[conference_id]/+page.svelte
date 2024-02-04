<script>
  import { page } from "$app/stores";
  import { DatePicker } from "@svelte-plugins/datepicker";

  const conference_id = $page.params.conference_id;

  import NavbarChair from "/src/components/navbar_chair.svelte";
  import NavbarUser from "/src/components/navbar_user.svelte";

  import { onMount } from "svelte";

  let user_id, user_type;

  user_id = $page.params.user_id;

  let getAllWorkshopURL = `http://localhost:3000/workshop/all/${conference_id}`;

  let allWorkshops = null;

  onMount(async () => {
    try {
      user_type = sessionStorage.getItem("user_type");

      console.log(user_type, "in conference");
      const response = await fetch(getAllWorkshopURL);
      if (!response.ok) {
        throw new Error("Failed to fetch data");
      }

      data = await response.json();
    } catch (error) {
      console.error("Error fetching data:", error);
    }
  });
</script>

<main>
  {#if allWorkshops != null}
    <h1>allWorkshops[0].workshop_title</h1>
  {/if}
</main>

<style>
</style>
