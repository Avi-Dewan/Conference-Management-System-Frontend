<script>
    import { page } from "$app/stores";
  
    import { onMount } from "svelte";
  
    import { goto } from "$app/navigation";
  
    import NavbarUser from "/src/components/navbar_user.svelte";
  
    let user_id, user, data;
  
    user_id = $page.params.user_id;
  
    onMount(async () => {
      try {
        let url = `http://localhost:3000/author/get_request/${user_id}`; 
  
        const response = await fetch(url);
  
        if (!response.ok) {
          throw new Error("Failed to fetch data");
        }
  
        data = await response.json();
  
        console.log(data);
      } catch (error) {
        console.error("Error fetching data:", error);
      }
    });





  
    async function handleReject(paper_id) {
      const response = await fetch(
        "http://localhost:3000/author/reject_request", // ekhane author er request delete hbe
        {
          method: "POST",
          headers: {
            "Content-Type": "application/json",
          },
          body: JSON.stringify({ user_id: user_id, paper_id: paper_id }), // workshop id jabe ekhane
        }
      );
      const thisPage = window.location.pathname;
  
      goto(`/${user_id}/home`).then(() => goto(thisPage));
    }
  
    async function handleAccept(paper_id) {
      let response = await fetch("http://localhost:3000/author/reject_request", {  // ekhane workshop er request delete hbe
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify({ user_id: user_id, paper_id: paper_id }),
      });
  
      response = await fetch("http://localhost:3000/author/accept_request", { // request accept hoise , tai db te add korlam
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify({ user_id: user_id, paper_id: paper_id }),
      });
  
      const thisPage = window.location.pathname;
  
      goto(`/${user_id}/home`).then(() => goto(thisPage));
    }
  </script>
  
  {#if data != null}
    <main>
      <NavbarUser />
      <h1>Paper For Co-Authorship</h1>
  
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
  