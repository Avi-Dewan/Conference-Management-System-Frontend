<script>
  import NavbarChair from "/src/components/navbar_chair.svelte";
  import NavbarUser from "/src/components/navbar_user.svelte";

  import Card from "/src/components/card.svelte";
  import { onMount } from "svelte";
  import { goto } from "$app/navigation";

  let data = [];

  import { page } from "$app/stores";

  let user_id, user_type;

  user_id = $page.params.user_id;

  let url = `${import.meta.env.VITE_BACKEND}/conference/all`;

  const currentDate = new Date();
  let unreadCount = null;

  onMount(async () => {
    try {
      user_type = sessionStorage.getItem("user_type");

      console.log(user_type, "in conference");
      const response = await fetch(url);
      if (!response.ok) {
        throw new Error("Failed to fetch data");
      }

      data = await response.json();

      for (let i = 0; i < data.length; i++) {
        const submissionDeadline = new Date(
          `${data[i].submission_deadline.date}T${data[i].submission_deadline.time}`
        );
        if (submissionDeadline < currentDate) data[i].status = "Closed";
        else data[i].status = "Open";
      }

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

{#if data != null && unreadCount != null}
  <main>
    {#if user_type == "chair"}
      <NavbarChair myVariable={unreadCount} />
    {:else}
      <NavbarUser myVariable={unreadCount} />
    {/if}

    <div class="header">
      <h1>Conferences</h1>
    </div>

    <nav>
      <a href="/{user_id}/conference/conference_list/all">All</a>
      <a href="/{user_id}/conference/conference_list/open_for_submission"
        >Open for submission</a
      >
      <div class="animation start-home"></div>
    </nav>

    <div class="cards">
      {#each data as item}
        <h2>{item.conference_title}</h2>
        <h3 style="color:black">Related Fields: {item.related_fields}</h3>
        <h4>Webpage:</h4>
        <a href={item.conference_webpage}>{item.conference_webpage}</a>
        {#if item.status == "Open"}
          <h3 style="color: green;">Status: {item.status}</h3>
        {:else}
          <h3 style="color: red;">Status: {item.status}</h3>
        {/if}

        <button
          on:click={() =>
            goto(
              `/${user_id}/conference/conference_list/all/${item.conference_id}`
            )}
        >
          View Details
        </button>
        <hr />
      {/each}
    </div>
  </main>
{/if}

<style>
  @import url("https://fonts.googleapis.com/css?family=Open+Sans&display=swap");
  .header {
    margin: 3% 1% 0% 0%;
  }

  body {
    font-family: "Open Sans", sans-serif;
    background: #2c3e50;
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
    width: 280px;
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
    width: 100px;
  }
  nav .start-home,
  a:nth-child(1):hover ~ .animation {
    width: 100px;
    left: 0;
  }
  nav a:nth-child(2) {
    width: 180px;
  }
  nav a:nth-child(2):hover ~ .animation {
    width: 180px;
    left: 100px;
  }

  main {
    max-width: 90%;
    margin: 2% auto;
  }
  .cards {
    margin: 120px 0%;
    display: block;
    flex-direction: column; /* Display elements vertically */
  }

  h1 {
  }
</style>
