<script>
  import { page } from "$app/stores";
  import { DatePicker } from "@svelte-plugins/datepicker";

  const conference_id = $page.params.conference_id;

  import NavbarChair from "/src/components/navbar_chair.svelte";
  import NavbarUser from "/src/components/navbar_user.svelte";

  import { goto } from "$app/navigation";

  import { onMount } from "svelte";

  let user_id, user_type;

  user_id = $page.params.user_id;

  let data = [];

  console.log(user_id);
  let url = `http://localhost:3000/notification/${user_id}`;

  const currentDate = new Date();

  onMount(async () => {
    try {
      user_type = sessionStorage.getItem("user_type");

      console.log(user_type);

      const response = await fetch(url);
      if (!response.ok) {
        throw new Error("Failed to fetch data");
      }

      data = await response.json();

      console.log(data);

      console.log(data);
    } catch (error) {
      console.error("Error fetching data:", error);
    }
  });

  async function handleDetails(notification_id, notification_json) {

      // Make a PUT request to the server
    const response = await fetch(`http://localhost:3000/notification/update_status/${notification_id}`, {
      method: 'PUT',
      headers: {
        'Content-Type': 'application/json'
      },
      // Add any data you want to send with the request
      body: JSON.stringify({  })
    });

    if (!response.ok) {
      console.error('Error:', response.statusText);
      return;
    }

    if (notification_json.type == "reviewer_request") {
      let paper_id = notification_json.requested_paper_id;
      goto(
        `/${user_id}/viewRequest/${notification_id}/reviewRequest/${paper_id}`
      );
    } else if (notification_json.type == "chair_noti_accept_reject") {
      let paper_id = notification_json.paper_id;
      goto(`/${user_id}/conference/assignReviewer/${paper_id}`);
    } else if (notification_json.type == "notify_reviewer") {
      let paper_id = notification_json.paper_id;

      goto(`/${user_id}/review/${paper_id}`);
    } else if (notification_json.type == "chair_noti_from_reviewer") {
      let paper_id = notification_json.paper_id;
      let conference_id = notification_json.conference_id;

      goto(
        `/${user_id}/conference/viewSubmission/${conference_id}/${paper_id}`
      );
    } else if (notification_json.type == "notify_chair_paper") {
      
      let conference_id = notification_json.conference_id;

      goto(
        `/${user_id}/conference/viewSubmission/${conference_id}`
      );
    } else if (notification_json.type == "notify_coauthor_paper"){
      let paper_id = notification_json.paper_id;
      goto(
        `/${user_id}/author_request/${paper_id}`
      );
    }else if (notification_json.type == "workshop_request_notify"){
      let workshop_id = notification_json.workshop_id;
      goto(
        `/${user_id}/workshop_request/${workshop_id}`
      );
    }
  }
</script>

<main>
  {#if user_type == "chair"}
    <NavbarChair />
  {:else}
    <NavbarUser />
  {/if}

  {#if data != null}
    <div>
      <h2>My Notification</h2>
      {#each data as item}
        <h3>{item.notification_body}</h3>

        {#if item.notification_json.type == "reviewer_request"}
          <h4 style="color: red;">
            Title: {item.notification_json.requested_paper_title}
          </h4>
        {/if}

        <!-- {#if item.notification_json.type == "notify_chair_paper"}
        <h4 style="color: red;">
          A new paper on the conference {item.notification_json.conference_id} has been submitted
        </h4>
        {/if} -->

        {#if item.notification_status == "unread"}
          <h4 style="color: red;">Status: Unread</h4>
        {:else}
          <h4 style="color: green;">Status: Read</h4>
        {/if}

        <button
          on:click={() => {
            handleDetails(item.notification_id, item.notification_json);
          }}>Go details</button
        >
      {/each}
      <div></div>
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
