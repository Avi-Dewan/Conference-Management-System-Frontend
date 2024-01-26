<script>
  import Navbar from "/src/components/navbar.svelte";
  import Card from "/src/components/card.svelte";
  import { onMount } from "svelte";

  async function doPost() {
    const req = await fetch("http://localhost:3000/trial", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify({
        id: 3,
        name: "rakib",
      }),
    });
    console.log(100);
    // const json = await res.json();
    // result = JSON.stringify(json);
  }
</script>

<main>
  <Navbar />
  <div class="header">
    <h1>Conferences</h1>
  </div>

  <button type="button" on:click={doPost}> Post it. </button>
  <nav>
    <a href="/conference/conference_list/all">All</a>
    <a href="/conference/conference_list/open_for_submission"
      >Open for submission</a
    >
    <div class="animation start-home"></div>
  </nav>
  <div class="cards">
    {#each { length: 3 } as _, i}
      <Card>
        <h1>Conference Name</h1>
        <h1>Conference details</h1>
        <h1>Conference details</h1>
        <h1>Conference details</h1>
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
