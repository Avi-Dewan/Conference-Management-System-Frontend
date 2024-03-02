<script>
  import NavbarChair from "/src/components/navbar_chair.svelte";
  import NavbarUser from "/src/components/navbar_user.svelte";

  import Card from "/src/components/card.svelte";
  import { onMount } from "svelte";

  let data = [];

  import { page } from "$app/stores";

  let user_id, user_type;

  user_id = $page.params.user_id;

  let url = `${import.meta.env.VITE_BACKEND}/reviewer/assignedPapers/${user_id}`;
  let unreadCount = null;

  onMount(async () => {
    try {
      user_type = sessionStorage.getItem("user_type");

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
</script>

{#if data != null && unreadCount != null}
  <main>
    {#if user_type == "chair"}
      <NavbarChair myVariable={unreadCount} />
    {:else}
      <NavbarUser myVariable={unreadCount} />
    {/if}

    <div class="header">
      <h1>Paper Reviews</h1>
    </div>

    <div class="cards">
      {#each data as item}
        <hr />
        <div>
          <h2>{item.paper_title}</h2>
          <h3>Related field: {item.related_fields}</h3>
          <h3>Abstract : {item.abstract}</h3>
          <h3>PDF link : <a href={item.pdf_link}> view pdf </a></h3>

          {#if item.review_status == "reviewed"}
            {#if item.paper_status == "revise"}
              <h3>
                Review status: <b style="color:red"
                  >{item.review_status} but need Revise</b
                >
              </h3>

              <h4>
                <b>Current review: </b>
                <p style="white-space: pre-wrap;">{item.review}</p>
              </h4>
              {#if item.submission_status == "closed"}
                <h3>
                  <a href="/{user_id}/review/{item.paper_id}">Review Again</a>
                </h3>
              {:else}
                <h4 style="color: red;">
                  Note: You can review this paper after Date: {item
                    .submission_deadline.date} & Time: {item.submission_deadline
                    .time}
                </h4>
              {/if}
            {:else}
              <h3>
                Review status: <b style="color:green">{item.review_status}</b>
                <p><b> Rating: </b> {item.rating}</p>
                <p><b> Review: </b> {item.review}</p>
                <h3>
                  <a href="/{user_id}/review/{item.paper_id}">Edit review</a>
                </h3>
              </h3>
            {/if}
          {:else}
            <h3>
              Review status: <b style="color:red">{item.review_status}</b>
            </h3>
            <h3><a href="/{user_id}/review/{item.paper_id}">Give review</a></h3>
          {/if}
        </div>
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
