<script>
  import { page } from "$app/stores";
  import { onMount } from "svelte";
  import NavbarChair from "/src/components/navbar_chair.svelte";

  const paper_id = $page.params.paper_id;

  //paper details
  onMount(async () => {
    try {
      const response = await fetch(url);
      if (!response.ok) {
        throw new Error("Failed to fetch data");
      }

      data = await response.json();

      console.log(data);
    } catch (error) {
      console.error("Error fetching data:", error);
    }
  });

  //kader kader request kora hoise
  onMount(async () => {
    try {
      const response = await fetch(url);
      if (!response.ok) {
        throw new Error("Failed to fetch data");
      }

      data = await response.json();

      console.log(data);
    } catch (error) {
      console.error("Error fetching data:", error);
    }
  });

  //kader kader assign kora hoise
  onMount(async () => {
    try {
      const response = await fetch(url);
      if (!response.ok) {
        throw new Error("Failed to fetch data");
      }

      data = await response.json();

      console.log(data);
    } catch (error) {
      console.error("Error fetching data:", error);
    }
  });
</script>

<main></main>

<style>
</style>
