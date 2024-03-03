<script>
  import { page } from "$app/stores";
  import { onMount } from "svelte";
  import NavbarChair from "/src/components/navbar_chair.svelte";
  import NavbarUser from "/src/components/navbar_user.svelte";
  import "/src/app.css";
  import { goto } from "$app/navigation";
  const conference_id = $page.params.conference_id;
  let user_id;

  user_id = $page.params.user_id;
  let url = `${import.meta.env.VITE_BACKEND}/conference/${conference_id}/viewUnassaignedPapers`;
  let conference_url = `${import.meta.env.VITE_BACKEND}/conference/${conference_id}`;
  let papers = null;
  let conf_data = null;

  onMount(async () => {
    try {
      const response = await fetch(url);
      if (!response.ok) {
        throw new Error("Failed to fetch data");
      }

      papers = await response.json();

      console.log(papers);
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
  async function handleReject(paper_id, paper_title) {
    let response = await fetch(
      `${import.meta.env.VITE_BACKEND}/chair/reject_paper`,
      {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify({ paper_id: paper_id }),
      }
    );

    let data = await response.json();

    let conf_res = await fetch(
      `${import.meta.env.VITE_BACKEND}/paper/getConferenceInfo/${paper_id}`
    );
    let conf_data = await conf_res.json();

    let conference_title = conf_data.conference_title;

    let notification_body = `Your paper titled "${paper_title}" has been rejected in the conference ${conference_title}`;

    let conference_id = conf_data.conference_id;

    let notification_json = {
      type: "notify_author_accept/reject",
      paper_id: paper_id,
      conference_id: conference_id,
    };

    // console.log(reviewer_id);

    let all_author_res = await fetch(
      `${import.meta.env.VITE_BACKEND}/paper/all_authors/${paper_id}`
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

  async function handleAccept(paper_id, paper_title) {
    let response = await fetch(
      `${import.meta.env.VITE_BACKEND}/chair/accept_paper`,
      {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify({ paper_id: paper_id }),
      }
    );
    let data = await response.json();

    let conf_res = await fetch(
      `${import.meta.env.VITE_BACKEND}/paper/getConferenceInfo/${paper_id}`
    );
    let conf_data = await conf_res.json();

    let conference_title = conf_data.conference_title;

    let notification_body = `Your paper titled "${paper_title}" has been accepted in the conference ${conference_title}`;

    let conference_id = conf_data.conference_id;

    let notification_json = {
      type: "notify_author_accept/reject",
      paper_id: paper_id,
      conference_id: conference_id,
    };

    // console.log(reviewer_id);

    let all_author_res = await fetch(
      `${import.meta.env.VITE_BACKEND}/paper/all_authors/${paper_id}`
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
  async function handleNotify(reviewer_id, paper_id, paper_title) {
    let notification_body = `You have a pending review \n for Paper title ${paper_title}`;

    let notification_json = {
      type: "notify_reviewer",
      paper_id: paper_id,
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
  {#if papers != null && conf_data != null && unreadCount != null}
    <NavbarChair myVariable={unreadCount} />
    <div>
      <hr class="border-t-2 border-gray-300 my-6" />
      <h1 class="mt-5">{conf_data.conference_title}</h1>
      <hr class="border-t-2 border-gray-300 my-6" />
      {#each papers as item}
        <div style="margin-top: 15px;">
          <h2>Paper Title: {item.paper_title}</h2>
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
                        item.paper_id,
                        item.paper_title
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
          <button
            class="btn bg-green-300"
            on:click={() => window.open(item.pdf_link, "_blank")}
            >View file</button
          >
          {#if item.status != "accepted" && item.status != "rejected" && item.status != "revise"}
            <button
              class="btn bg-red-300"
              on:click={() =>
                goto(`/${user_id}/conference/assignReviewer/${item.paper_id}`)}
              >Assign</button
            >
          {/if}
        </div>
        <!-- <hr />
        <div style="margin-top: 70px;">
          <h2>Paper Title: {item.paper_title}</h2>
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
                    handleNotify(rev.user_id, item.paper_id, item.paper_title);
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
              window.location.href = `/${user_id}/conference/assignReviewer/${item.paper_id}`;
            }}>Assign</button
          >

           <div class="two-column" style="display: block" button-container>
            <button
              on:click={handleReject(item.paper_id, item.paper_title)}
              style="background-color:red;">Reject</button
            >
            <button
              on:click={handleAccept(item.paper_id, item.paper_title)}
              style="background-color:green;">Accept</button
            >
          </div> 
        </div> -->
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
</style>
