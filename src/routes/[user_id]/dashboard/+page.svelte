<script>
  import { goto } from "$app/navigation";
  import { page } from "$app/stores";
  import { onMount } from "svelte";
  import NavbarChair from "/src/components/navbar_chair.svelte";
  import "/src/app.css";

  const user_id = $page.params.user_id;
  let filtered_data = [];

  let searchQuery = "";
  let getConferencesSummaryURL = `${import.meta.env.VITE_BACKEND}/conference/${user_id}/all`;

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
      filtered_data = data;

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

    <div>
      <nav>
        <a href="/{user_id}/dashboard">Conferences</a>
        <a href="/{user_id}/dashboard/unassigned_papers"
          >View Unassigned Papers</a
        >
        <a href="/{user_id}/dashboard/pending_papers">View pending papers</a>
        <a href="/{user_id}/dashboard/unassigned_workshop"
          >View Unassigned Workshop</a
        >
        <div class="animation start-home"></div>
      </nav>
    </div>

    <div>
      {#if getConferencesSummary != null}
        <br />
        <br />
        <br />
        <br />
        <br />
        <hr />

        <div class="header"><h2>Your conferences:</h2></div>

        <div class="card">
          <div class="mt-8">
            <input
              type="text"
              class="input input-bordered input-accent w-full max-w-xs"
              bind:value={searchQuery}
              placeholder="Search by conference name..."
              on:input={() => {
                filtered_data = getConferencesSummary.filter(
                  (item) =>
                    // item.paper_title
                    //   .toLowerCase()
                    //   .toLowerCase()
                    //   .indexOf(searchQuery.toLowerCase()) !== -1 ||
                    item.conference_title
                      .toLowerCase()
                      .indexOf(searchQuery.toLowerCase()) !== -1
                );

                if (searchQuery == "") {
                  filtered_data = getConferencesSummary;
                }
              }}
            />
            <button class="search-button">Search</button>
          </div>

          {#each filtered_data as item}
            <div class="card bg-gray-200 shadow-xl mt-10">
              <div class="card-body">
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

                <div style="margin-top: 20px;" class="card-actions justify">
                  <button
                    class="btn btn-success"
                    on:click={() => {
                      handleViewConference(item.conference_id);
                    }}>View Conference</button
                  >
                </div>
                {#if item.paper_count_with_no_reviewer_assigned != 0}
                  <div style="margin-top: 20px;" class="card-actions justify">
                    <button
                      class="btn btn-success"
                      on:click={() => {
                        handleUnassignedReviewer(item.conference_id);
                      }}>View papers with unassigned reviewer</button
                    >
                  </div>
                {/if}
                {#if item.paper_count_with_pending_review != 0}
                  <div style="margin-top: 20px;" class="card-actions justify">
                    <button
                      class="btn btn-success"
                      on:click={() => {
                        handlePendingReview(item.conference_id);
                      }}>View papers with pending reviews</button
                    >
                  </div>
                {/if}
              </div>
            </div>
            <hr />
          {/each}
        </div>
      {/if}
    </div>
  {/if}
</main>

<style>
  main {
    max-width: 90%;
    margin: 2% auto;
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

  nav {
    float: left;
    position: relative;
    margin: 1% 0%;
    width: 850px;
    height: 50px;
    background: #34495e;
    border-radius: 8px;
    font-size: 0;
    box-shadow: 0 2px 3px 0 rgba(0, 0, 0, 0.1);
  }
  nav a {
    font-size: 15px;
    text-transform: uppercase;
    color: white;
    text-decoration: none;
    line-height: 50px;
    position: relative;
    z-index: 1;
    display: inline-block;
    text-align: center;
  }
  nav .animation {
    position: absolute;
    height: 100%;
    /* height: 5px; */
    top: 0;
    /* bottom: 0; */
    z-index: 0;
    background: #1abc9c;
    border-radius: 8px;
    transition: all 0.5s ease 0s;
  }
  nav a:nth-child(1) {
    width: 200px;
  }
  nav .start-home,
  a:nth-child(1):hover ~ .animation {
    width: 200px;
    left: 0;
  }
  nav a:nth-child(2) {
    width: 200px;
  }
  nav a:nth-child(2):hover ~ .animation {
    width: 200px;
    left: 200px;
  }
  nav a:nth-child(3) {
    width: 200px;
  }
  nav a:nth-child(3):hover ~ .animation {
    width: 200px;
    left: 400px;
  }

  nav a:nth-child(4):hover ~ .animation {
    width: 250px;
    left: 600px;
  }
</style>
