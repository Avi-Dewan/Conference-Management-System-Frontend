<script>
  import { page } from "$app/stores";
  import { onMount } from "svelte";
  import NavbarChair from "/src/components/navbar_chair.svelte";
  import NavbarUser from "/src/components/navbar_user.svelte";

  const conference_id = $page.params.conference_id;
  let user_id;

  user_id = $page.params.user_id;
  let url = `http://localhost:3000/conference/${conference_id}/papersWithAuthors`;
  let conference_url = `http://localhost:3000/conference/${conference_id}`;
  let data = null;
  let conf_data = null;

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

  onMount(async () => {
    try {
      const response = await fetch(conference_url);
      if (!response.ok) {
        throw new Error("Failed to fetch data");
      }

      conf_data = await response.json();
    } catch (error) {
      console.error("Error fetching data:", error);
    }
  });
</script>

<main>
  {#if data != null && conf_data != null}
    <NavbarChair />
    <div>
      <h1>{conf_data.conference_title}</h1>
      {#each data as item}
        <div style="margin-top: 70px;">
          <h2>Paper Title: {item.paper_title}</h2>
          <p>Author: {item.authors}</p>
          <p>Track: {item.related_fields}</p>
          <p>Abstract: {item.abstract}</p>
          <p>Assigned Reviewer:</p>
          <p>Requested Reviewer:</p>
          <p>Re:</p>
          <a href={item.pdf_link}>View file</a>
          <a href="/{user_id}/conference/assignReviewer/{item.paper_id}">
            Assign
          </a>
        </div>
      {/each}
    </div>
  {/if}
</main>

<style>
  main {
    max-width: 90%;
    margin: 2% auto;
  }

  h1 {
  }
</style>
