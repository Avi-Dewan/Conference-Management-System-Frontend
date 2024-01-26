<script>
  import { page } from "$app/stores";

  const conference_id = $page.params.conference_id;

  import Navbar from "/src/components/navbar.svelte";

  import { onMount } from "svelte";

  let url = `http://localhost:3000/trial`;

  console.log(url);
  async function fettch() {
    try {
      const response = await fetch("http://localhost:3000/trial");
      if (!response.ok) {
        throw new Error("Failed to fetch data");
      }
      const temp = await response.json();
    } catch (error) {
      console.error("Error fetching data:", error);
    }
    return temp;
  }

  fettch.then((data) => {
    console.log(data);
  });
</script>

<main>
  <Navbar />
  <div>
    <h1>Conference Title:</h1>
    <h3>Venue:</h3>
    <h3>Webpage:</h3>
    <h2>Research Areas:</h2>
    <h3>Start date:</h3>
    <h3>End date:</h3>
    <h2>Status:</h2>
    <h3>Submission Deadline:</h3>
    <h3>Description:</h3>

    <a href="/conference/submitPaper/{conference_id}">Submit a Paper</a>
  </div>
</main>

<style>
  main {
    max-width: 90%;
    margin: 2% auto;
  }

  h1 {
  }
</style>
