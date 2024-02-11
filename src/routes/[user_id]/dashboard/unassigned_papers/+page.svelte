<script>
  import { goto } from "$app/navigation";
  import { page } from "$app/stores";
  import { onMount } from "svelte";
  import NavbarChair from "/src/components/navbar_chair.svelte";

  const user_id = $page.params.user_id;

  let getUnassigned_papersURL = `http://localhost:3000/conference/unassignedPapers/${user_id}`;

  let unassigned_papers = null;
  onMount(async () => {
    try {
      let user_type = sessionStorage.getItem("user_type");

      const response = await fetch(getUnassigned_papersURL);
      if (!response.ok) {
        throw new Error("Failed to fetch data");
      }

      let data = await response.json();

      unassigned_papers = data;

      console.log(unassigned_papers);
    } catch (error) {
      console.error("Error fetching data:", error);
    }
  });

  function handleViewPaper(paper_id) {
    goto(`/${user_id}/conference/assignReviewer/${paper_id}`);
  }
 
</script>

<main>
  <NavbarChair />

  <div>
    <nav>
      <a href="/{user_id}/dashboard">Conferences</a>
      <a href="/{user_id}/dashboard/unassigned_papers"
        >View Unassigned Papers</a
      >
      <a href="/{user_id}/dashboard/pending_papers"
        >View pending papers</a
      >
      <a href="/{user_id}/dashboard/unassigned_workshop"
      >View Unassigned Workshop</a
      >
      <div class="animation start-home"></div>
    </nav>
  </div>
  <div>
    {#if unassigned_papers != null}
      <br />
      <br />
      <br />
      <br />
      <br />
      <hr />
      <h2> Unassigned Papers:</h2>

      {#each unassigned_papers as item}
        <div>
          <hr />
          <h3>Title: {item.paper_title}</h3>
          <h4>Authors: {item.authors}</h4>
          <h4>Related Fields: {item.related_fields}</h4>
          <button
              on:click={() => {
                handleViewPaper(item.paper_id);
              }}>View details</button
            >
        </div>
        <hr />
      {/each}
    {/if}
  </div>
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
