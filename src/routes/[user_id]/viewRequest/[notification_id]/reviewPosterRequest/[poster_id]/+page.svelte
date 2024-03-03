<script>
    import { page } from "$app/stores";
  
    import { onMount } from "svelte";
  
    import { goto } from "$app/navigation";
  
    import NavbarUser from "/src/components/navbar_user.svelte";
    import "/src/app.css";
    let user_id, user, data;
  
    let poster_id = $page.params.poster_id;
    let notification_id = $page.params.notification_id;
    user_id = $page.params.user_id;
    let unreadCount = null;
  
    onMount(async () => {
      try {
        let url = `${import.meta.env.VITE_BACKEND}/request/poster/${poster_id}`;
  
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
  
    async function handleReject(poster_id) {
      let response = await fetch(`${import.meta.env.VITE_BACKEND}/poster/request/delete_request`, {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify({ user_id: user_id, poster_id: poster_id }),
      });
  
      response = await fetch(
        `${import.meta.env.VITE_BACKEND}/poster/get_conference_chair/${poster_id}`
      );
  
      if (!response.ok) {
        throw new Error("Failed to fetch data");
      }
  
      let chair_id = await response.json();
  
      let res = await fetch(`${import.meta.env.VITE_BACKEND}/user/getFullName/${user_id}`);
  
      let full_name = await res.json();
  
      res = await fetch(
        `${import.meta.env.VITE_BACKEND}/poster/getConferenceInfo/${poster_id}`
      );
  
      let PosterInfoJSON = await res.json();
  
      let conference_title = PosterInfoJSON.conference_title;
      let poster_title = PosterInfoJSON.poster_title;
  
      let notification_body =
        full_name +
        ` rejected your request for reviewing
        for Poster Title:${poster_title} 
        in Conference : ${conference_title}`;
  
      let notification_json = {
        type: "chair_poster_noti_accept_reject",
        outcome: "reject",
        poster_id: poster_id,
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
  
    async function handleAccept(poster_id) {
      let response = await fetch(`${import.meta.env.VITE_BACKEND}/poster/request/delete_request`, {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify({ user_id: user_id, poster_id: poster_id }),
      });
  
      response = await fetch(`${import.meta.env.VITE_BACKEND}/poster/reviewer/accept`, {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify({ user_id: user_id, poster_id: poster_id }),
      });
  
      response = await fetch(
        `${import.meta.env.VITE_BACKEND}/poster/get_conference_chair/${poster_id}`
      );
  
      if (!response.ok) {
        throw new Error("Failed to fetch data");
      }
  
      let chair_id = await response.json();
  
      let res = await fetch(`${import.meta.env.VITE_BACKEND}/user/getFullName/${user_id}`);
  
      let full_name = await res.json();
  
      res = await fetch(
        `${import.meta.env.VITE_BACKEND}/poster/getConferenceInfo/${poster_id}`
      );
  
      let PosterInfoJSON = await res.json();
  
      let conference_title = PosterInfoJSON.conference_title;
      let poster_title = PosterInfoJSON.poster_title;
  
      let notification_body =
        full_name +
        ` accepted your request for reviewing
        for Poster Title:${poster_title} 
        in Conference : ${conference_title}`;
  
      let notification_json = {
        type: "chair_poster_noti_accept_reject",
        outcome: "accept",
        poster_id: poster_id,
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
      <br />
      <br />
      <h1>Poster For Review</h1>
  
      <!-- <button on:click={handleClick}>Go to Another Page</button> -->
  
      <div class="cards">
        {#each data as item}
        <div class="card bg-gray-200 shadow-xl mt-10">
          <div class="card-body">
            <h2 style="margin-top: 20px;">{item.poster_title}</h2>
            <!-- <h3>Pdf Link: {item.pdf_link}</h3> -->

            <div style="margin-top: 20px;" class="card-actions justify">

              <a class="btn btn-info" href={item.pdf_link}>View Poster</a>

            </div>

            <h3 style="margin-top: 20px;" >Related Fields: {item.related_fields}</h3>
            <h3 style="margin-top: 20px;" >Abstract: {item.abstract}</h3>
            
           <div style="margin-top: 20px;" class="card-actions justify"> 
            
            <a class="btn btn-neutral"
              href="/{user_id}/conference/conference_list/all/{item.conference_id}"
              >View Conference</a
            >

           </div>
  
            <div
              class="two-column"
              style="display: block;margin-top:2%"
              button-container
            >
              <button
                on:click={handleReject(item.poster_id)}
                style="background-color:red;">Reject</button
              >
              <button
                on:click={handleAccept(item.poster_id)}
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
      margin: 3% 10px;
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
  