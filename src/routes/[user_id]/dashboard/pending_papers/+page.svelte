<script>
  import { goto } from "$app/navigation";
  import { page } from "$app/stores";
  import { onMount } from "svelte";
  import NavbarChair from "/src/components/navbar_chair.svelte";
  import "/src/app.css";

  const user_id = $page.params.user_id;

  let searchQuery = "";
  let filtered_data = [];

  let getPending_papersURL = `${
    import.meta.env.VITE_BACKEND
  }/conference/pendingPapers/${user_id}`;

  let pending_papers = null;
  onMount(async () => {
    try {
      let user_type = sessionStorage.getItem("user_type");

      const response = await fetch(getPending_papersURL);
      if (!response.ok) {
        throw new Error("Failed to fetch data");
      }

      let data = await response.json();

      pending_papers = data;
      filtered_data = data;

      console.log(pending_papers);
    } catch (error) {
      console.error("Error fetching data:", error);
    }
  });

  function handleViewPaper(conference_id, paper_id) {
    goto(`/${user_id}/conference/viewSubmission/${conference_id}/${paper_id}`);
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
      {#if pending_papers != null}
        <br />
        <br />
        <br />
        <br />
        <br />
        <hr />

        <div class="header">
          <h2>Pending Papers:</h2>
        </div>

        <div class="card">
          <div class="mt-8">
            <input
              type="text"
              class="input input-bordered input-accent w-full max-w-xs"
              bind:value={searchQuery}
              placeholder="Search by paper name..."
              on:input={() => {
                filtered_data = pending_papers.filter(
                  (item) =>
                    // item.paper_title
                    //   .toLowerCase()
                    //   .toLowerCase()
                    //   .indexOf(searchQuery.toLowerCase()) !== -1 ||
                    item.paper_title
                      .toLowerCase()
                      .indexOf(searchQuery.toLowerCase()) !== -1
                );

                if (searchQuery == "") {
                  filtered_data = pending_papers;
                }
              }}
            />
            <button class="search-button">Search</button>
          </div>
          {#each filtered_data as item}
            <div class="card bg-gray-200 shadow-xl mt-10">
              <div class="card-body">
                <hr />
                <h3>Paper Title: {item.paper_title}</h3>
                <h4>Authors: {item.authors}</h4>
                <h4>Related Fields: {item.related_fields}</h4>
                <h4>Conference name: {item.conference_title}</h4>

                <div style="margin-top: 20px;" class="card-actions justify">
                  <button
                    class="btn btn-success"
                    on:click={() => {
                      handleViewPaper(item.conference_id, item.paper_id);
                    }}>View details</button
                  >
                </div>
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
