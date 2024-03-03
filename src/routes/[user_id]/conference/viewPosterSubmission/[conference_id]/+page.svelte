<script>
  import { page } from "$app/stores";
  import { onMount } from "svelte";
  import NavbarChair from "/src/components/navbar_chair.svelte";
  import NavbarUser from "/src/components/navbar_user.svelte";
  import { goto } from "$app/navigation";
  import "/src/app.css";

  const conference_id = $page.params.conference_id;
  let user_id;

  user_id = $page.params.user_id;
  let url = `${import.meta.env.VITE_BACKEND}/conference/${conference_id}/postersWithAuthors`;
  let conference_url = `${import.meta.env.VITE_BACKEND}/conference/${conference_id}`;
  let posters = null;
  let conf_data = null;

  onMount(async () => {
    try {
      const response = await fetch(url);
      if (!response.ok) {
        throw new Error("Failed to fetch data");
      }

      posters = await response.json();
      console.log(posters);

      // console.log(papers);
    } catch (error) {
      console.error("Error fetching data:", error);
    }
  });

  onMount(async () => {
    try {
      const response = await fetch(conference_url);
      if (!response.ok) {
        throw new Error("Failed to fetch data");
      }

      conf_data = await response.json();
    } catch (error) {
      console.error("Error fetching data:", error);
    }
  });

  async function handleReject(poster_id, poster_title) {
    let response = await fetch(
      `${import.meta.env.VITE_BACKEND}/chair/reject_poster`,
      {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify({ poster_id: poster_id }),
      }
    );

    let data = await response.json();

    let conf_res = await fetch(
      `${import.meta.env.VITE_BACKEND}/poster/getConferenceInfo/${poster_id}`
    );
    let conf_data = await conf_res.json();

    let conference_title = conf_data.conference_title;

    let notification_body = `Your poster titled "${poster_title}" has been rejected in the conference ${conference_title}`;

    let conference_id = conf_data.conference_id;

    let notification_json = {
      type: "notify_author_poster_accept/reject",
      poster_id: poster_id,
      conference_id: conference_id,
    };

    // console.log(reviewer_id);

    let all_author_res = await fetch(
      `${import.meta.env.VITE_BACKEND}/poster/all_authors/${poster_id}`
    );

    let all_author_id = await all_author_res.json();

    for (let i = 0; i < all_author_id.length; i++) {
      await fetch(`${import.meta.env.VITE_BACKEND}/notification/send`, {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify({
          user_id: all_author_id[i].user_id,
          notification_body: notification_body,
          notification_json: notification_json,
        }),
      });
    }
    const thisPage = window.location.pathname;
    goto(`/${user_id}/home`).then(() => goto(thisPage));
  }

  async function handleAccept(poster_id, poster_title) {
    let response = await fetch(
      `${import.meta.env.VITE_BACKEND}/chair/accept_poster`,
      {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify({ poster_id: poster_id }),
      }
    );
    let data = await response.json();

    let conf_res = await fetch(
      `${import.meta.env.VITE_BACKEND}/poster/getConferenceInfo/${poster_id}`
    );
    let conf_data = await conf_res.json();

    let conference_title = conf_data.conference_title;

    let notification_body = `Your poster titled "${poster_title}" has been accepted in the conference ${conference_title}`;

    let conference_id = conf_data.conference_id;

    let notification_json = {
      type: "notify_author_poster_accept/reject",
      poster_id: poster_id,
      conference_id: conference_id,
    };

    // console.log(reviewer_id);

    let all_author_res = await fetch(
      `${import.meta.env.VITE_BACKEND}/poster/all_authors/${poster_id}`
    );

    let all_author_id = await all_author_res.json();

    for (let i = 0; i < all_author_id.length; i++) {
      await fetch(`${import.meta.env.VITE_BACKEND}/notification/send`, {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify({
          user_id: all_author_id[i].user_id,
          notification_body: notification_body,
          notification_json: notification_json,
        }),
      });
    }
    const thisPage = window.location.pathname;
    goto(`/${user_id}/home`).then(() => goto(thisPage));
  }
  async function handleNotify(reviewer_id, poster_id, poster_title) {
    let notification_body = `You have a pending review \n for Poster title ${poster_title}`;

    let notification_json = {
      type: "notify_reviewer_poster",
      poster_id: poster_id,
    };

    console.log(reviewer_id);

    let response = await fetch(
      `${import.meta.env.VITE_BACKEND}/notification/send`,
      {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify({
          user_id: reviewer_id,
          notification_body: notification_body,
          notification_json: notification_json,
        }),
      }
    );

    const thisPage = window.location.pathname;

    console.log(notification_json);

    goto(`/${user_id}/home`).then(() => goto(thisPage));
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
  {#if posters != null && conf_data != null && unreadCount != null}
    <NavbarChair myVariable={unreadCount} />
    <div>
      <hr class="border-t-2 border-gray-300 my-6" />
      <h1 class="mt-5">{conf_data.conference_title}</h1>
      <hr class="border-t-2 border-gray-300 my-6" />
      {#each posters as item}
        <hr class="border-t-2 border-black-300 my-6 mt-5" />
        <div style="margin-top: 15px;">
          <h2>Poster Title: {item.poster_title}</h2>
          <hr class="border-t-2 border-gray-300 my-6" />
          <p class="mt-5"><b>Author:</b> {item.authors}</p>
          <p class="mt-5"><b>Research Field:</b> {item.related_fields}</p>
          <p class="mt-5"><b>Abstract:</b> {item.abstract}</p>
          {#if item.status == "accepted"}
            <p class="mt-5" style="color:green;">
              <b>Status:</b>
              {item.status}
            </p>
          {:else}
            <p class="mt-5" style="color:red;">
              <b>Status:</b>
              {item.status}
            </p>
          {/if}
          <hr class="border-t-2 border-gray-300 my-6" />
          <p class="mt-5">
            <b> <u> Requested Reviewer: </u></b>
            {#if item.requestedReviewers.length == 0}
              None
            {:else}
              {item.requestedReviewers}
            {/if}
          </p>
          <hr class="border-t-2 border-gray-300 my-6" />
          <p class="mt-5"><b> <u> Assigned Reviewers and reviews: </u></b></p>
          <hr class="border-t-2 border-gray-300 my-6" />
          {#each item.reviews as rev}
            <div class="mt-2">
              <p><b>{rev.full_name}</b></p>
              {#if rev.rating != null}
                <p>Rating: {rev.rating}</p>
              {:else}
                <p>Rating: None</p>
              {/if}
              {#if rev.review != null}
                <p class="mt-2">Review:</p>
                <p style="white-space: pre-wrap;">{rev.review}</p>
              {:else}
                <p style="color:red">Not given review yet</p>
                <div class="mt-5">
                  <button
                    class="mt-5"
                    style="background-color: red;"
                    on:click={() => {
                      handleNotify(
                        rev.user_id,
                        item.poster_id,
                        item.poster_title
                      );
                    }}
                  >
                    Notify {rev.full_name}
                  </button>
                </div>
                <br />
                <br />
              {/if}
              <br />
              <hr class="border-t-2 border-gray-300 my-6" />
            </div>
          {/each}
          <button on:click={() => window.open(item.pdf_link, "_blank")}
            >View file</button
          >

          {#if item.status != "accepted" && item.status != "rejected" && item.status != "revise"}
            <button
              on:click={() =>
                goto(
                  `/${user_id}/conference/assignPosterReviewer/${item.poster_id}`
                )}>Assign</button
            >
            <div
              class="two-column"
              style="display: block;margin-top:20px"
              button-container
            >
              <button
                on:click={handleReject(item.poster_id, item.poster_title)}
                style="background-color:red;">Reject</button
              >
              <button
                on:click={handleAccept(item.poster_id, item.poster_title)}
                style="background-color:green;">Accept</button
              >
              <hr class="border-t-2 border-gray-300 my-6" />
            </div>
          {/if}
        </div>
        <!-- <div style="margin-top: 70px;">
          <h2>Poster Title: {item.poster_title}</h2>
          <p><b>Author:</b> {item.authors}</p>
          <p><b>Related Fields:</b> {item.related_fields}</p>
          <p><b>Abstract:</b> {item.abstract}</p>
          <p><b>Status:</b> {item.status}</p>
          <p><b>Requested Reviewer:</b> {item.requestedReviewers}</p>
          <p><b> <u> Assigned Reviewers and reviews: </u></b></p>

          {#each item.reviews as rev}
            <div>
              <b>{rev.full_name}</b>
              {#if rev.rating != null}
                <p>rating: {rev.rating}</p>
                <p>review: {rev.review}</p>
              {:else}
                <p style="color:red">Not given review yet</p>
                <button
                  style="background-color: blue;"
                  on:click={() => {
                    handleNotify(
                      rev.user_id,
                      item.poster_id,
                      item.poster_title
                    );
                  }}>Notify {rev.full_name}</button
                >
                <br />
                <br />
              {/if}
              <br />
            </div>
          {/each}
          <button on:click={() => window.open(item.pdf_link, "_blank")}
            >View file</button
          >

          {#if item.status != "accepted" && item.status != "rejected"}
            <button
              on:click={() =>
                goto(
                  `/${user_id}/conference/assignPosterReviewer/${item.poster_id}`
                )}>Assign</button
            >
            <div
              class="two-column"
              style="display: block;margin-top:20px"
              button-container
            >
              <button
                on:click={handleReject(item.poster_id, item.poster_title)}
                style="background-color:red;">Reject</button
              >
              <button
                on:click={handleAccept(item.poster_id, item.poster_title)}
                style="background-color:green;">Accept</button
              >
            </div>
          {/if}
        </div> -->

        <!-- </div> -->
      {/each}
    </div>
  {/if}
</main>

<style>
  main {
    max-width: 90%;
    margin: 2% auto;
  }

  h1 {
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
