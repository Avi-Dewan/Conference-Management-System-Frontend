<script>
  import { page } from "$app/stores";

  import { onMount } from "svelte";

  import { goto } from "$app/navigation";
  import "/src/app.css";

  import NavbarUser from "/src/components/navbar_user.svelte";

  let user_id, user, data;

  user_id = $page.params.user_id;

  let workshop_id = $page.params.workshop_id;
  let unreadCount = null;

  onMount(async () => {
    try {
      let url = `${import.meta.env.VITE_BACKEND}/workshop/get_request/${user_id}/${workshop_id}`; // ekhane workshop er link hbe

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

      console.log(data);
    } catch (error) {
      console.error("Error fetching data:", error);
    }
  });

  async function handleReject(workshop_id, workshop_title) {
    let response = await fetch(
      `${import.meta.env.VITE_BACKEND}/workshop/reject_request`, // ekhane workshop er request delete hbe
      {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify({ user_id: user_id, workshop_id: workshop_id }), // workshop id jabe ekhane
      }
    );

    let conf_res = await fetch(
      `${import.meta.env.VITE_BACKEND}/workshop/get_conference/${workshop_id}`
    );

    let conf_data = await conf_res.json();

    let conference_title = conf_data.conference_details.conference_title;

    let chair_id = conf_data.chair_id;

    let res = await fetch(`${import.meta.env.VITE_BACKEND}/user/getFullName/${user_id}`);

    let full_name = await res.json();

    let notification_body = `${full_name} has rejected your request to be an instructor for Workshop: "${workshop_title}" in the conference "${conference_title}"`;

    let notification_json = {
      type: "notify_chair_workshop_accept/reject",
      workshop_id: workshop_id,
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

    goto(`/${user_id}/home`).then(() => goto(thisPage));
  }

  async function handleAccept(workshop_id, workshop_title) {
    let response = await fetch(
      `${import.meta.env.VITE_BACKEND}/workshop/reject_request`,
      {
        // ekhane workshop er request delete hbe
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify({ user_id: user_id, workshop_id: workshop_id }),
      }
    );

    response = await fetch(`${import.meta.env.VITE_BACKEND}/workshop/accept_request`, {
      // request accept hoise , tai db te add korlam
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify({ user_id: user_id, workshop_id: workshop_id }),
    });

    let conf_res = await fetch(
      `${import.meta.env.VITE_BACKEND}/workshop/get_conference/${workshop_id}`
    );

    let conf_data = await conf_res.json();

    let conference_title = conf_data.conference_details.conference_title;

    let chair_id = conf_data.chair_id;

    let res = await fetch(`${import.meta.env.VITE_BACKEND}/user/getFullName/${user_id}`);

    let full_name = await res.json();

    let notification_body = `${full_name} has accepted your request to be an instructor for Workshop: "${workshop_title}" in the conference "${conference_title}"`;

    let notification_json = {
      type: "notify_chair_workshop_accept/reject",
      workshop_id: workshop_id,
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

    goto(`/${user_id}/home`).then(() => goto(thisPage));
  }
</script>

{#if data != null && unreadCount != null}
  <main>
    <NavbarUser myVariable={unreadCount} />

    <h1>Workshop For Review</h1>

    <!-- <button on:click={handleClick}>Go to Another Page</button> -->
    <div class="cards">
      {#each data as item}
      <div class="card bg-gray-200 shadow-xl mt-10">
        <div class="card-body">
          <h2 style="margin-top: 20px;">{item.workshop_title}</h2>
          <!-- <h3>Pdf Link: {item.pdf_link}</h3> -->
          <!-- <a href={item.pdf_link}>View Paper</a> -->
          <h3 style="margin-top: 20px;">Related Fields: {item.related_fields}</h3>
          <h3 style="margin-top: 20px;">Description: {item.workshop_description}</h3>
          <div style="margin-top: 20px;" class="card-actions justify">

          <a class="btn btn-neutral"
              href="/{user_id}/conference/conference_list/all/{item.conference_id}"
              >View Conference</a
            >
            </div>

          <div
            class="two-column"
            style="display: block;margin-top:1%"
            button-container
          >
            <button
              on:click={handleReject(item.workshop_id, item.workshop_title)}
              style="background-color:red;">Reject</button
            >
            <button
              on:click={handleAccept(item.workshop_id, item.workshop_title)}
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
