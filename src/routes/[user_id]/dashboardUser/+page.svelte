<script>
  import { goto } from "$app/navigation";
  import { page } from "$app/stores";
  import { onMount } from "svelte";
  import NavbarChair from "/src/components/navbar_user.svelte";

  const user_id = $page.params.user_id;

  let getPapersURL = `http://localhost:3000/paper/allPapers/${user_id}`;

  let getPapersSummary = null;
  onMount(async () => {
    try {
      let user_type = sessionStorage.getItem("user_type");

      const response = await fetch(getPapersURL);
      if (!response.ok) {
        throw new Error("Failed to fetch data");
      }

      let data = await response.json();

      getPapersSummary = data;

      console.log(data);
    } catch (error) {
      console.error("Error fetching data:", error);
    }
  });

  function handleViewPaper(conference_id) {
    goto(`/${user_id}/conference/mysubmission/${conference_id}`);
  }
  let unreadCount = null;
  onMount(async () => {
    try {
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

    {#if getPapersSummary != null}
      <h2>Your papers:</h2>

      {#each getPapersSummary as item}
        <div>
          <hr />
          <h3>Title: {item.paper_title}</h3>
          <h4>Related fields: {item.related_fields}</h4>
          {#if item.status == "accepted"}
            <h4>Status: <b style="color: green;"> accepted</b></h4>
          {:else if item.status == "rejected"}
            <h4>Status: <b style="color: red;">rejected</b></h4>
          {:else}
            <h4>Status: {item.status}</h4>
          {/if}

          <button
            on:click={() => {
              handleViewPaper(item.conference_id);
            }}>View Paper</button
          >
          <!-- {#if item.paper_count_with_pending_review != 0}
          <button
            on:click={() => {
              handlePendingReview(item.conference_id);
            }}>View papers with pending reviews</button
          >
        {/if} -->
        </div>
        <hr />
      {/each}
    {/if}
  {/if}
</main>

<style>
  main {
    max-width: 90%;
    margin: 2% auto;
  }

  h1 {
  }

  button {
    margin-top: 0%;
    padding: 8px 20px;
    border: none;
    border-radius: 4px;
    background-color: #007bff;
    color: #fff;
    font-size: 16px;
    cursor: pointer;
  }
</style>
