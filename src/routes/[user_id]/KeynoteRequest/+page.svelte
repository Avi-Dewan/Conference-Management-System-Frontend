<script>
    import { page } from "$app/stores";
  
    import { onMount } from "svelte";
  
    import { goto } from "$app/navigation";
  
    import NavbarUser from "/src/components/navbar_user.svelte";
    import "/src/app.css";
  
    let user_id, user, data;
  
    let conference_id = $page.params.conference_id;
    let notification_id = $page.params.notification_id;
    user_id = $page.params.user_id;
    let unreadCount = null;
  
    onMount(async () => {
      try {
        let url = `${import.meta.env.VITE_BACKEND}/keynote/request/${user_id}`;
  
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
  
    async function handleReject(conference_id) {
      let response = await fetch(`${import.meta.env.VITE_BACKEND}/keynote/request/delete_request`, {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify({ user_id: user_id, conference_id: conference_id }),
      });
  
      response = await fetch(
        `${import.meta.env.VITE_BACKEND}/keynote/get_conference_chair/${conference_id}`
      );
  
      if (!response.ok) {
        throw new Error("Failed to fetch data");
      }
  
      let chair_id = await response.json();
  
      let res = await fetch(`${import.meta.env.VITE_BACKEND}/user/getFullName/${user_id}`);
  
      let full_name = await res.json();
  
      res = await fetch(
        `${import.meta.env.VITE_BACKEND}/keynote/getConferenceInfo/${conference_id}`
      );
  
      let ConferenceInfoJSON = await res.json();
  
      let conference_title = ConferenceInfoJSON.conference_title;
      //let poster_title = PosterInfoJSON.poster_title;
  
      let notification_body =
        full_name +
        ` rejected your request for being a keynote speaker
        in Conference : ${conference_title}`;
  
      let notification_json = {
        type: "chair_keynote_noti_accept_reject",
        outcome: "reject",
        conference_id: conference_id,
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
  
    async function handleAccept(conference_id) {
      let response = await fetch(`${import.meta.env.VITE_BACKEND}/keynote/request/delete_request`, {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify({ user_id: user_id, conference_id: conference_id }),
      });
  
      response = await fetch(`${import.meta.env.VITE_BACKEND}/keynote/reviewer/accept`, {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify({ user_id: user_id, conference_id: conference_id }),
      });
  
      response = await fetch(
        `${import.meta.env.VITE_BACKEND}/keynote/get_conference_chair/${conference_id}`
      );
  
      if (!response.ok) {
        throw new Error("Failed to fetch data");
      }
  
      let chair_id = await response.json();
  
      let res = await fetch(`${import.meta.env.VITE_BACKEND}/user/getFullName/${user_id}`);
  
      let full_name = await res.json();
  
      res = await fetch(
        `${import.meta.env.VITE_BACKEND}/keynote/getConferenceInfo/${conference_id}`
      );
  
      let ConferenceInfoJSON = await res.json();
  
      let conference_title = ConferenceInfoJSON.conference_title;
      //let poster_title = PosterInfoJSON.poster_title;
  
      let notification_body =
        full_name +
        ` accepted your request for being a keynote speaker
        in Conference : ${conference_title}`;
  
      let notification_json = {
        type: "chair_keynote_noti_accept_reject",
        outcome: "accept",
        conference_id: conference_id,
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

      <nav style="margin-top: 2%;">
        <a href="/{user_id}/Request">Paper Review</a>
        <a href="/{user_id}/PosterRequest">Poster Review</a>
        <a href="/{user_id}/workshop_request">Workshop Instructor</a>
        <a href="/{user_id}/author_request">Paper Co authorship</a>
        <a href="/{user_id}/author_poster_request">Poster Co authorship </a>
        <a href="/{user_id}/KeynoteRequest">Keynote Speaker</a>
        <div class="animation start-home"></div>
      </nav>
      <br />
      <br />
      <br />
      <br />
      <br />

      <h1>Conference For Keynote</h1>
  
      <!-- <button on:click={handleClick}>Go to Another Page</button> -->
  
      <div class="cards">
        {#each data as item}
          <div class="card bg-gray-200 shadow-xl mt-10">
            <div class="card-body">
            <h2>{item.conference_title}</h2>

            <div style="margin-top: 20px;" class="card-actions justify">
              
            <a class="btn btn-primary"
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
                on:click={handleReject(item.conference_id)}
                style="background-color:red;">Reject</button
              >
              <button
                on:click={handleAccept(item.conference_id)}
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
      margin: 2% 8px;
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
      width: 1200px;
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
    nav a:nth-child(4) {
    width: 200px;
  }
  nav a:nth-child(4):hover ~ .animation {
    width: 200px;
    left: 600px;
  }
  nav a:nth-child(5) {
    width: 200px;
  }
  nav a:nth-child(5):hover ~ .animation {
    width: 200px;
    left: 800px;
  }
  nav a:nth-child(6) {
    width: 200px;
  }
  nav a:nth-child(6):hover ~ .animation {
    width: 200px;
    left: 1000px;
  }
  </style>
  