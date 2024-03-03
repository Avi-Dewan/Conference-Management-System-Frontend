<script>
  import { goto } from "$app/navigation";
  import { page } from "$app/stores";
  import { onMount } from "svelte";
  import NavbarChair from "/src/components/navbar_user.svelte";
  import "/src/app.css";
  const user_id = $page.params.user_id;

  let searchQuery = "";

  let filteredPapers = [];

  function handleSearch() {
    console.log(`Searching for: ${searchQuery}`);
  }

  let getPapersURL = `${
    import.meta.env.VITE_BACKEND
  }/paper/allPapers/${user_id}`;

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
      filteredPapers = data;

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
        `${import.meta.env.VITE_BACKEND}/notification/unreadCount/${user_id}`
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

    <div class="search-container">
      <input
        class="search-input"
        bind:value={searchQuery}
        type="text"
        placeholder="Search by conference name or paper title..."
        on:input={() => {
          filteredPapers = getPapersSummary.filter(
            (item) =>
              item.paper_title
                .toLowerCase()
                .toLowerCase()
                .indexOf(searchQuery.toLowerCase()) !== -1 ||
              item.conference_title
                .toLowerCase()
                .indexOf(searchQuery.toLowerCase()) !== -1
          );

          if (searchQuery == "") {
            filteredPapers = getPapersSummary;
          }
        }}
      />
      <button class="search-button" on:click={handleSearch}>Search</button>
    </div>
    {#if filteredPapers != null}
      <h2>Your papers:</h2>

      {#each filteredPapers as item}
        <div class="card bg-gray-200 shadow-xl mt-10">
          <div class="card-body">
            <h2>Paper Title: {item.paper_title}</h2>
            <h4>Authors: {item.authors}</h4>
            <h4 class="mt-4">Conference name: {item.conference_title}</h4>
            <h4 class="mt-4">Related fields: {item.related_fields}</h4>

            {#if item.status == "accepted"}
              <h4 class="mb-4">
                Status: <b style="color: green;"> accepted</b>
              </h4>
            {:else if item.status == "rejected"}
              <h4 class="mb-4">Status: <b style="color: red;">rejected</b></h4>
            {:else}
              <h4 class="mb-4">Status: <b style="color: red;">rejected</b></h4>
            {/if}
            <div class="card-actions">
              <button
                class="btn"
                on:click={() => {
                  handleViewPaper(item.conference_id);
                }}>View Paper</button
              >
            </div>
          </div>
        </div>
        <!-- <div>
          <hr />
          <h3>Paper Title: {item.paper_title}</h3>
          <h4>Authors: {item.authors}</h4>
          <h4>Related fields: {item.related_fields}</h4>
          <h4>Conference name: {item.conference_title}</h4>
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
        {/if}
        </div> -->
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
  .search-container {
    display: flex;
    justify-content: center;
    margin-top: 50px;
  }

  .search-input {
    padding: 10px;
    font-size: 16px;
    border: none;
    border-radius: 4px;
    width: 400px;
  }

  .search-button {
    padding: 10px 20px;
    font-size: 16px;
    border: none;
    border-radius: 4px;
    margin-left: 10px;
    background-color: #4caf50;
    color: white;
    cursor: pointer;
  }

  .search-button:hover {
    background-color: #45a049;
  }
</style>
