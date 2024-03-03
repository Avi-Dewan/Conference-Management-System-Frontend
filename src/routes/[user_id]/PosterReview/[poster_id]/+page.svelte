<script>
    import NavbarChair from "/src/components/navbar_chair.svelte";
    import NavbarUser from "/src/components/navbar_user.svelte";
    import { goto } from "$app/navigation";
    import { onMount } from "svelte";
  
    import { page } from "$app/stores";
  
    let user_id, poster_id;
  
    user_id = $page.params.user_id;
    poster_id = $page.params.poster_id;
    let user_type;
    let data = null;
  
    let rating, review;
  
    let url = `${import.meta.env.VITE_BACKEND}/poster/${poster_id}`;
    let unreadCount = null;
  
    onMount(async () => {
      try {
        user_type = sessionStorage.getItem("user_type");
  
        const response = await fetch(url);
  
        if (!response.ok) {
          throw new Error("Failed to fetch data");
        }
  
        data = await response.json();
        data = data[0];
  
        const unreadNotificationCount = await fetch(
          `${import.meta.env.VITE_BACKEND}/notification/unreadCount/${user_id}`
        );
        unreadCount = await unreadNotificationCount.json();
        unreadCount = unreadCount.unreadCount;
  
      } catch (error) {
        console.error("Error fetching data:", error);
      }
    });
  
    async function handleSubmit(poster_title) {
      let response = await fetch(
        `${import.meta.env.VITE_BACKEND}/poster/get_conference/${poster_id}`
      );
  
      let conference_id = await response.json();
  
      conference_id = conference_id[0].conference_id;
  
      response = await fetch(
        `${import.meta.env.VITE_BACKEND}/poster/get_conference_chair/${poster_id}`
      );
  
      let chair_id = await response.json();
  
      response = await fetch(`${import.meta.env.VITE_BACKEND}/user/getFullName/${user_id}`);
  
      let reviewer_full_name = await response.json();
  
      let notification_body =
        reviewer_full_name + ` has reviewed the poster ${poster_title}`;
  
      let notification_json = {
        type: "chair_poster_noti_from_reviewer",
        poster_id: poster_id,
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
  
      response = await fetch(`${import.meta.env.VITE_BACKEND}/poster/reviewer/review`, {
        method: "PUT",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify({
          user_id: user_id,
          poster_id: poster_id,
          rating: rating,
          review: review,
        }),
      });
  
      goto(`/${user_id}/review`);
    }
  </script>
  
  <main>
    {#if unreadCount != null}
      <NavbarUser myVariable={unreadCount} />
  
    {#if data != null}
      <div>
        <h2>{data.poster_title}</h2>
        <p>Related field: {data.related_fields}</p>
        <p>Abstract : {data.abstract}</p>
        <h3>PDF link : <a href={data.pdf_link}> view pdf </a></h3>
      </div>
  
      <h1>Write review</h1>
  
      <div class="form">
        <div class="form-control">
          <label for="rating">Rating:</label>
          <input type="text" id="rating" bind:value={rating} />
        </div>
  
        <div class="form-control">
          <label for="review">Review:</label>
  
          <textarea
            style="height: 200px; width:890px;border-radius:.3em"
            id="review"
            bind:value={review}
          />
        </div>
  
        <div class="form-control" style="display: block;">
          <button
            on:click={() => {
              handleSubmit(data.poster_title);
            }}>Submit</button
          >
        </div>
      </div>
    {/if}
  
    {/if}
  </main>
  
  <style>
    .two-column {
      column-count: 2; /* Split into two columns */
      column-gap: 30%; /* Adjust gap between columns */
    }
    .two-column button {
      margin-left: -10%;
    }
    .two-column div {
      break-inside: avoid; /* Prevent breaking elements between columns */
    }
    input {
      height: 50px;
      width: 500px;
    }
    main {
      max-width: 90%;
      margin: 2% auto;
    }
  
    h1 {
    }
  
    .form {
      width: 900px;
      margin: 0 auto;
      padding: 20px;
      border: 1px solid #ccc;
      border-radius: 8px;
      background-color: #f9f9f9;
      float: left;
    }
  
    .form-control {
      margin-bottom: 20px;
      min-height: 20px;
      min-width: 50px;
      clear: both;
    }
  
    .form-control label {
      display: block;
      margin-bottom: 5px;
      font-weight: bold;
    }
  
    .form-control input,
    .form-control select {
      width: 100%;
      padding: 8px;
      border: 1px solid #ccc;
      border-radius: 4px;
      box-sizing: border-box;
      font-size: 16px;
    }
  
    .form-control button {
      margin-top: 5%;
      padding: 8px 20px;
      border: none;
      border-radius: 4px;
      background-color: #007bff;
      color: #fff;
      font-size: 16px;
      cursor: pointer;
    }
  
    .form-control button:hover {
      background-color: #0056b3;
    }
  
    .form-control-submission {
      margin-bottom: 20px;
    }
  
    .form-control-submission label {
      display: block;
      margin-bottom: 5px;
      font-weight: bold;
    }
  
    .form-control-submission input,
    .form-control-submission select {
      width: 95%;
      padding: 8px;
      border: 1px solid #ccc;
      border-radius: 4px;
      box-sizing: border-box;
      font-size: 16px;
    }
    .column {
      float: left;
      width: 50%;
    }
    .column-field {
      align-items: flex-start;
      float: left;
      width: 50%;
      max-width: 400px;
      margin: 0 auto;
      padding: 20px;
  
      border-radius: 8px;
      background-color: #f9f9f9;
      display: flex;
      flex-direction: column;
    }
    .column button {
      margin-left: 18%;
      margin-top: 1%;
      background-color: green;
      height: 40px;
    }
  
    /* Clear floats after the columns */
    .row:after {
      content: "";
      display: table;
      clear: both;
    }
    .redFont {
      color: red;
    }
    .desc input {
      height: 150px;
    }
  </style>
  