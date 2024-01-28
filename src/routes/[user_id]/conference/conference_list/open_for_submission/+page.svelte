<script>
  import NavbarChair from "/src/components/navbar_chair.svelte";
  import NavbarUser from "/src/components/navbar_user.svelte";

  import Card from "/src/components/card.svelte";
  import { onMount } from "svelte";
  import { page } from "$app/stores";

  let data = [];

  let user_id, user_type;

  $: user_id = $page.params.user_id;

  let url = `http://localhost:3000/conference/open`;

  const currentDate = new Date();

  onMount(async () => {
    try {

      user_type = sessionStorage.getItem("user_type");

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
    } catch (error) {
      console.error("Error fetching data:", error);
    }
  });
</script>

<main>
  {#if user_type == "chair"}
      <NavbarChair/>
    {:else}
      <NavbarUser/>
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
      <Card>
        <h2>{item.conference_title}</h2>
        <h3>Related Fields: {item.related_fields}</h3>
        <h3>Webpage: {item.conference_webpage}</h3>
        <h3>Status: {item.status}</h3>
        <a href="/{user_id}/conference/conference_list/all/{item.conference_id}"
          >View Details</a
        >
      </Card>
    {/each}
  </div>
</main>

<style>
  @import url("https://fonts.googleapis.com/css?family=Open+Sans&display=swap");
  .header {
    margin: 3% 1% 0% 0%;
  }

  body {
    font-family: "Open Sans", sans-serif;
    background: #2c3e50;
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
    margin: 10% 0%;
    display: block;
    flex-direction: column; /* Display elements vertically */
  }

  h1 {
  }
</style>
