<script>
  import { goto } from "$app/navigation";
  import { page } from "$app/stores";
  import { onMount } from "svelte";
  import NavbarChair from "/src/components/navbar_chair.svelte";

  const user_id = $page.params.user_id;

  let getConferencesSummaryURL = `http://localhost:3000/conference/${user_id}/all`;

  let getConferencesSummary = null;
  onMount(async () => {
    try {
      let user_type = sessionStorage.getItem("user_type");

      const response = await fetch(getConferencesSummaryURL);
      if (!response.ok) {
        throw new Error("Failed to fetch data");
      }

      let data = await response.json();

      getConferencesSummary = data;

      console.log(data);
    } catch (error) {
      console.error("Error fetching data:", error);
    }
  });
</script>

<main>
  <NavbarChair />

  {#if getConferencesSummary != null}
    <h2>Your conferences:</h2>

    {#each getConferencesSummary as item}
      <div>
        <hr />
        <h3>Title: {item.conference_title}</h3>
        <h4>Total submitted papers: {item.submitted_paper_count}</h4>
        <h4 style="color: red;">
          Total Papers without any reviewer assigned: {item.paper_count_with_no_reviewer_assigned}
        </h4>
        <h4 style="color: red;">
          Total Papers with pending reviews: {item.paper_count_with_pending_review}
        </h4>

        <button>View Conference</button>
        <button>View papers with unassigned reviewer</button>
        <button>View papers with pending reviews</button>
      </div>
      <hr />
    {/each}
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
