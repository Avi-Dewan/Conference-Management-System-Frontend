<script>
  import NavbarChair from "/src/components/navbar_chair.svelte";
  import NavbarUser from "/src/components/navbar_user.svelte";

  import Card from "/src/components/card.svelte";
  import { onMount } from "svelte";
  import "/src/app.css";

  let data = [];

  import { page } from "$app/stores";

  let user_id, user_type;

  user_id = $page.params.user_id;

  let url = `${import.meta.env.VITE_BACKEND}/reviewer/assignedPosters/${user_id}/withAuthors`;
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

    <nav style="margin-top: 2%;">
      <a href="/{user_id}/review">Reviewed Papers</a>
      <a href="/{user_id}/PosterReview">Reviewed Posters</a>

      <div class="animation start-home"></div>
    </nav>
    <br />
    <br />
    <br />
    <br />
    <br />

    <div class="header">
      <h1>Poster Reviews</h1>
    </div>

    {#if data.length == 0}
      <div class="header">
        <h2>You have not reviewed any poster yet</h2>
      </div>
    {/if}

    <div class="cards">
      {#each data as item}
        <div class="card bg-gray-200 shadow-xl mt-10">
          <div class="card-body">
            <h2 style="margin-top: 20px;">{item.poster_title}</h2>
            <h3 style="margin-top: 20px;">
              Related field: {item.related_fields}
            </h3>
            <h3 style="margin-top: 20px;">Abstract : {item.abstract}</h3>
            <h3 style="margin-top: 20px;">
              PDF link : <a href={item.pdf_link}> View File </a>
            </h3>

            <!-- {#if item.review_status == "reviewed"}
              {#if item.poster_status == "revise"}
                <h3 style="margin-top: 20px;">
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
                    <a href="/{user_id}/review/{item.poster_id}">Review Again</a>
                  </h3>
                {:else}
                  <h4 style="color: red;">
                    Note: You can review this poster after Date: {item
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
                    <a href="/{user_id}/PosterReview/{item.poster_id}">Edit review</a>
                  </h3>
                </h3>
              {/if}
            {:else} -->
            <h3 style="margin-top: 20px;">
              Review status: <b style="color:red">{item.review_status}</b>
            </h3>
            <h3 style="margin-top: 20px;">
              <a
                class="btn btn-success"
                href="/{user_id}/PosterReview/{item.poster_id}">Give review</a
              >
            </h3>
            <!-- {/if} -->
          </div>
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

  nav {
    float: left;
    position: relative;
    margin: 1% 0%;
    width: 400px;
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
</style>
