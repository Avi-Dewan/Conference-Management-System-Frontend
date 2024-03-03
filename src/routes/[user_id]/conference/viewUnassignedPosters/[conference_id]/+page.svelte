<script>
  import { page } from "$app/stores";
  import { onMount } from "svelte";
  import NavbarChair from "/src/components/navbar_chair.svelte";
  import NavbarUser from "/src/components/navbar_user.svelte";
  import "/src/app.css";
  const conference_id = $page.params.conference_id;
  let user_id;

  user_id = $page.params.user_id;
  let url = `${import.meta.env.VITE_BACKEND}/conference/${conference_id}/viewUnassaignedPosters`;
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
      <h1>{conf_data.conference_title}</h1>
      {#each posters as item}
        <hr />
        <div style="margin-top: 70px;">
          <h2>Posters Title: {item.poster_title}</h2>
          <p><b>Author:</b> {item.authors}</p>
          <p><b>Track:</b> {item.related_fields}</p>
          <p><b>Abstract:</b> {item.abstract}</p>
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
          <button
            on:click={() => {
              window.location.href = item.pdf_link;
            }}>View file</button
          >
          <button
            on:click={() => {
              window.location.href = `/${user_id}/conference/assignReviewer/${item.poster_id}`;
            }}>Assign</button
          >

          <div class="two-column" style="display: block" button-container>
            <button
              on:click={handleReject(item.poster_id, item.poster_title)}
              style="background-color:red;">Reject</button
            >
            <button
              on:click={handleAccept(item.poster_id, item.poster_title)}
              style="background-color:green;">Accept</button
            >
          </div>
        </div>
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
  button {
    margin: 2% 0%;
    padding: 8px 20px;
    border: none;
    border-radius: 4px;
    background-color: blue;
    color: #fff;
    font-size: 16px;
    cursor: pointer;
  }
</style>
