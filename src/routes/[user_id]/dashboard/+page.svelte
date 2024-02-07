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

  function handleUnassignedReviewer(conference_id) {
    goto(`/${user_id}/conference/viewUnassignedPapers/${conference_id}`);
  }
  function handlePendingReview(conference_id) {
    goto(`/${user_id}/conference/viewPendingReviewPapers/${conference_id}`);
  }
  function handleViewConference(conference_id) {
    goto(`/${user_id}/conference/conference_list/all/${conference_id}`);
  }
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
        <h4 style="color: black;">
          Total workshops: {item.workshop_count}
        </h4>

        <button
          on:click={() => {
            handleViewConference(item.conference_id);
          }}>View Conference</button
        >
        {#if item.paper_count_with_no_reviewer_assigned != 0}
          <button
            on:click={() => {
              handleUnassignedReviewer(item.conference_id);
            }}>View papers with unassigned reviewer</button
          >
        {/if}
        {#if item.paper_count_with_pending_review != 0}
          <button
            on:click={() => {
              handlePendingReview(item.conference_id);
            }}>View papers with pending reviews</button
          >
        {/if}
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
