<script>
  import { page } from "$app/stores";

  import { onMount } from "svelte";

  import { goto } from "$app/navigation";

  import NavbarUser from "/src/components/navbar_user.svelte";

  let user_id, user, data;

  user_id = $page.params.user_id;

  onMount(async () => {
    try {
      let url = `http://localhost:3000/request/${user_id}`;

      const response = await fetch(url);

      if (!response.ok) {
        throw new Error("Failed to fetch data");
      }

      data = await response.json();
    } catch (error) {
      console.error("Error fetching data:", error);
    }
  });

  async function handleReject(paper_id) {
    let response = await fetch("http://localhost:3000/request/delete_request", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify({ user_id: user_id, paper_id: paper_id }),
    });

    response = await fetch(
      `http://localhost:3000/paper/get_conference_chair/${paper_id}`
    );

    if (!response.ok) {
      throw new Error("Failed to fetch data");
    }

    let chair_id = await response.json();

    let res = await fetch(`http://localhost:3000/user/getFullName/${user_id}`);

    let full_name = await res.json();

    let notification_body = full_name + " rejected your request for reviewing";

    let notification_json = {
      type: "chair_noti_accept_reject",
      paper_id: paper_id,
    };

    console.log(notification_body);
    response = await fetch("http://localhost:3000/notification/send", {
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

    goto(`/${user_id}/home`).then(() => goto(thisPage));
  }

  async function handleAccept(paper_id) {
    let response = await fetch("http://localhost:3000/request/delete_request", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify({ user_id: user_id, paper_id: paper_id }),
    });

    let respons = await fetch("http://localhost:3000/reviewer/accept", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify({ user_id: user_id, paper_id: paper_id }),
    });

    let respon = await fetch(
      `http://localhost:3000/paper/get_conference_chair/${paper_id}`
    );

    if (!respon.ok) {
      throw new Error("Failed to fetch data");
    }

    let chair_id = await respon.json();

    let res = await fetch(`http://localhost:3000/user/getFullName/${user_id}`);

    let full_name = await res.json();

    let notification_body = full_name + " accepted your request for reviewing";

    let notification_json = {
      type: "chair_noti_accept_reject",
      paper_id: paper_id,
    };

    response = await fetch("http://localhost:3000/notification/send", {
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

{#if data != null}
  <main>
    <NavbarUser />

    <nav style="margin-top: 2%;">
      <a href="/{user_id}/Request">Paper Review</a>
      <a href="/{user_id}/workshop_request">Workshop Instructor</a>
      <a href="/{user_id}/author_request">Co authorship</a>
      <div class="animation start-home"></div>
    </nav>
    <br />
    <br />
    <br />
    <br />
    <br />
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
  nav {
    float: left;
    position: relative;
    margin: 1% 0%;
    width: 600px;
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
</style>
