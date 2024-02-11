<script>
  import { page } from "$app/stores";
  import { onMount } from "svelte";
  import NavbarChair from "/src/components/navbar_chair.svelte";
  import NavbarUser from "/src/components/navbar_user.svelte";

  const conference_id = $page.params.conference_id;
  let user_id;

  user_id = $page.params.user_id;
  let url = `http://localhost:3000/conference/${conference_id}/viewUnassaignedPapers`;
  let conference_url = `http://localhost:3000/conference/${conference_id}`;
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
  async function handleReject(paper_id) {
    let response = await fetch("http://localhost:3000/chair/reject_paper", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify({ paper_id: paper_id }),
    });

    let data = await response.json();
    console.log(data);
    const thisPage = window.location.pathname;
    goto(`/${user_id}/home`).then(() => goto(thisPage));
  }

  async function handleAccept(paper_id) {
    let response = await fetch("http://localhost:3000/chair/accept_paper", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify({ paper_id: paper_id }),
    });
    let data = await response.json();

    console.log(data);
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

    let response = await fetch("http://localhost:3000/notification/send", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify({
        user_id: reviewer_id,
        notification_body: notification_body,
        notification_json: notification_json,
      }),
    });

    const thisPage = window.location.pathname;

    console.log(notification_json);

    goto(`/${user_id}/home`).then(() => goto(thisPage));
  }
</script>

<main>
  {#if papers != null && conf_data != null}
    <NavbarChair />
    <div>
      <h1>{conf_data.conference_title}</h1>
      {#each papers as item}
        <hr />
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
          <button on:click={() => {window.location.href = item.pdf_link;}}>View file</button>
          <button on:click={() => {window.location.href = `/${user_id}/conference/assignReviewer/${item.paper_id}`;}}>Assign</button>

          <div class="two-column" style="display: block" button-container>
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
