<script>
  import { page } from "$app/stores";

  import { onMount } from "svelte";

  import { goto } from "$app/navigation";

  import NavbarUser from "/src/components/navbar_user.svelte";
  import "/src/app.css";

  let user_id, user, data;

  let unreadCount = null;

  user_id = $page.params.user_id;

  onMount(async () => {
    try {
      let url = `${import.meta.env.VITE_BACKEND}/author/get_request/${user_id}`;

      const response = await fetch(url);

      if (!response.ok) {
        throw new Error("Failed to fetch data");
      }

      data = await response.json();

      console.log(data);

      const unreadNotificationCount = await fetch(
        `${import.meta.env.VITE_BACKEND}/notification/unreadCount/${user_id}`
      );

      unreadCount = await unreadNotificationCount.json();
      unreadCount = unreadCount.unreadCount;
    } catch (error) {
      console.error("Error fetching data:", error);
    }
  });

  async function handleReject(paper_id) {
    const response = await fetch(
      `${import.meta.env.VITE_BACKEND}/author/reject_request`, // ekhane author er request delete hbe
      {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify({ user_id: user_id, paper_id: paper_id }), // workshop id jabe ekhane
      }
    );
    const thisPage = window.location.pathname;

    goto(`/${user_id}/home`).then(() => goto(thisPage));
  }

  async function handleAccept(paper_id) {
    let response = await fetch(`${import.meta.env.VITE_BACKEND}/author/reject_request`, {
      // ekhane workshop er request delete hbe
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify({ user_id: user_id, paper_id: paper_id }),
    });

    response = await fetch(`${import.meta.env.VITE_BACKEND}/author/accept_request`, {
      // request accept hoise , tai db te add korlam
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify({ user_id: user_id, paper_id: paper_id }),
    });

    const thisPage = window.location.pathname;

    goto(`/${user_id}/home`).then(() => goto(thisPage));
  }
</script>

{#if data != null && unreadCount != null}
  <main>
    <NavbarUser myVariable={unreadCount} />

    <nav style="margin-top: 2%;">
      <a href="/{user_id}/Request">Paper Review</a>
      <a href="/{user_id}/PosterRequest">Poster Review</a>
      <a href="/{user_id}/workshop_request">Workshop Instructor</a>
      <a href="/{user_id}/author_request">Paper Co authorship</a>
      <a href="/{user_id}/author_poster_request">Poster Co authorship   </a>
      <a href="/{user_id}/KeynoteRequest">Keynote Speaker</a>
      <div class="animation start-home"></div>
    </nav>
    <br />
    <br />
    <br />
    <br />
    <br />
    <h1>Paper For Co-Authorship</h1>

    <div class="header">
      {#if data.length == 0}
        <h2  style="margin-top: 30px;">You have no co-authorship request</h2>
      {/if}
    </div>
    <!-- <button on:click={handleClick}>Go to Another Page</button> -->

    <div class="cards">
      {#each data as item}
      <div class="card bg-gray-200 shadow-xl mt-10">
        <div class="card-body">
          <h2 style="margin-top: 20px;">{item.paper_title}</h2>

          <div style="margin-top: 20px;" class="card-actions justify">
            <a class="btn btn-info" href={item.pdf_link}>View Paper</a>
          </div>

          <h3 style="margin-top: 20px;">Related Fields: {item.related_fields}</h3>
          <h3 style="margin-top: 20px;">Abstract: {item.abstract}</h3>
          
          <div style="margin-top: 20px;" class="card-actions justify">
            <a  class="btn btn-neutral"
              href="/{user_id}/conference/conference_list/all/{item.conference_id}"
              >View Conference</a
            >
          </div>  
          <div
            class="two-column"
            style="display: block;margin-top:0%"
            button-container
          >
            <button
              on:click={handleReject(item.paper_id)}
              style="background-color:red;">Reject</button
            >
            <button
              on:click={handleAccept(item.paper_id)}
              style="background-color:green;">Accept</button
            >
          </div>
        </div>
      </div>
      {/each}
    </div>
  </main>
{/if}

<style>
  main {
    max-width: 90%;
    margin: 2% auto;
  }

  h1 {
  }

  button {
    margin: 3% 4px;
    padding: 8px 20px;
    border: none;
    border-radius: 4px;

    color: #fff;
    font-size: 16px;
    cursor: pointer;
  }

  .two-column {
    margin-top: -1%;
    /* column-count: 2; /* Split into two columns */
    /* column-gap: 30%; Adjust gap between columns */
    display: flex;
    justify-content: center;
  }

  .two-column div {
    break-inside: avoid; /* Prevent breaking elements between columns */
  }

  .border_style {
    color: #333;
    /* Add the border property to your existing class */
    border: 2px solid #000;
    /* Optional: Add padding to the content inside the div */
    padding: 10px;
    /* Additional styles */
    background-color: #f4f4f4; /* Set background color */
    text-align: center; /* Set text alignment to center */
    font-family: Arial, sans-serif; /* Set font family */
    margin: 20px; /* Add margin around the div */
  }

  .button-container {
    text-align: center;
  }
  nav {
    float: left;
    position: relative;
    margin: 1% 0%;
    width: 1200px;
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
  nav a:nth-child(4) {
    width: 200px;
  }
  nav a:nth-child(4):hover ~ .animation {
    width: 200px;
    left: 600px;
  }

  nav a:nth-child(5) {
    width: 200px;
  }
  nav a:nth-child(5):hover ~ .animation {
    width: 200px;
    left: 800px;
  }
  nav a:nth-child(6) {
    width: 200px;
  }
  nav a:nth-child(6):hover ~ .animation {
    width: 200px;
    left: 1000px;
  }
</style>
