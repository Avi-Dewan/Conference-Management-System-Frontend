<script>
  import { page } from "$app/stores";

  import { onMount } from "svelte";

  import { goto } from "$app/navigation";

  import NavbarUser from "/src/components/navbar_user.svelte";

  let user_id, user, data;

  let paper_id = $page.params.paper_id;
  let notification_id = $page.params.notification_id;
  user_id = $page.params.user_id;
  let unreadCount = null;

  onMount(async () => {
    try {
      let url = `${import.meta.env.VITE_BACKEND}/request/paper/${paper_id}`;

      const response = await fetch(url);

      if (!response.ok) {
        throw new Error("Failed to fetch data");
      }

      data = await response.json();

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
    let response = await fetch(`${import.meta.env.VITE_BACKEND}/request/delete_request`, {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify({ user_id: user_id, paper_id: paper_id }),
    });

    response = await fetch(
      `${import.meta.env.VITE_BACKEND}/paper/get_conference_chair/${paper_id}`
    );

    if (!response.ok) {
      throw new Error("Failed to fetch data");
    }

    let chair_id = await response.json();

    let res = await fetch(`${import.meta.env.VITE_BACKEND}/user/getFullName/${user_id}`);

    let full_name = await res.json();

    res = await fetch(
      `${import.meta.env.VITE_BACKEND}/paper/getConferenceInfo/${paper_id}`
    );

    let PaperInfoJSON = await res.json();

    let conference_title = PaperInfoJSON.conference_title;
    let paper_title = PaperInfoJSON.paper_title;

    let notification_body =
      full_name +
      ` rejected your request for reviewing
      for Paper Title:${paper_title} 
      in Conference : ${conference_title}`;

    let notification_json = {
      type: "chair_noti_accept_reject",
      outcome: "reject",
      paper_id: paper_id,
    };

    console.log(notification_body);
    response = await fetch(`${import.meta.env.VITE_BACKEND}/notification/send`, {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify({
        user_id: chair_id,
        notification_body: notification_body,
        notification_json: notification_json,
      }),
    });

    const thisPage = window.location.pathname;

    console.log(notification_json);

    goto(`/${user_id}/notification`);
  }

  async function handleAccept(paper_id) {
    let response = await fetch(`${import.meta.env.VITE_BACKEND}/request/delete_request`, {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify({ user_id: user_id, paper_id: paper_id }),
    });

    response = await fetch(`${import.meta.env.VITE_BACKEND}/reviewer/accept`, {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify({ user_id: user_id, paper_id: paper_id }),
    });

    response = await fetch(
      `${import.meta.env.VITE_BACKEND}/paper/get_conference_chair/${paper_id}`
    );

    if (!response.ok) {
      throw new Error("Failed to fetch data");
    }

    let chair_id = await response.json();

    let res = await fetch(`${import.meta.env.VITE_BACKEND}/user/getFullName/${user_id}`);

    let full_name = await res.json();

    res = await fetch(
      `${import.meta.env.VITE_BACKEND}/paper/getConferenceInfo/${paper_id}`
    );

    let PaperInfoJSON = await res.json();

    let conference_title = PaperInfoJSON.conference_title;
    let paper_title = PaperInfoJSON.paper_title;

    let notification_body =
      full_name +
      ` accepted your request for reviewing
      for Paper Title:${paper_title} 
      in Conference : ${conference_title}`;

    let notification_json = {
      type: "chair_noti_accept_reject",
      outcome: "accept",
      paper_id: paper_id,
    };

    response = await fetch(`${import.meta.env.VITE_BACKEND}/notification/send`, {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify({
        user_id: chair_id,
        notification_body: notification_body,
        notification_json: notification_json,
      }),
    });

    const thisPage = window.location.pathname;

    goto(`/${user_id}/notification`);
  }
</script>

{#if data != null  && unreadCount != null}
  <main>
    <NavbarUser myVariable={unreadCount} />
    <h1>Paper For Review</h1>

    <!-- <button on:click={handleClick}>Go to Another Page</button> -->

    <div class="cards">
      {#each data as item}
        <div class="border_style">
          <h2>{item.paper_title}</h2>
          <!-- <h3>Pdf Link: {item.pdf_link}</h3> -->
          <a href={item.pdf_link}>View Paper</a>
          <h3>Related Fields: {item.related_fields}</h3>
          <h3>Abstract: {item.abstract}</h3>
          <a
            href="/{user_id}/conference/conference_list/all/{item.conference_id}"
            >View Conference</a
          >

          <div
            class="two-column"
            style="display: block;margin-top:2%"
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
    margin: 5% 20px;
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
</style>
