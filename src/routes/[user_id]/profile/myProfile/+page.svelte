<script>
  import { page } from "$app/stores";

  import { onMount } from "svelte";

  import { goto } from "$app/navigation";

  import NavbarUser from "/src/components/navbar_user.svelte";

  let user_id, user = null, getPapersSummary = null;;
  let unreadCount = null;

  user_id = $page.params.user_id;

  let getPapersURL = `${import.meta.env.VITE_BACKEND}/paper/allPapers/${user_id}`;


  onMount(async () => {
    try {
      let url = `${import.meta.env.VITE_BACKEND}/user/${user_id}`;

      const response = await fetch(url);

      if (!response.ok) {
        throw new Error("Failed to fetch data");
      }

      user = await response.json();

      // console.log(user);

      const response2 = await fetch(getPapersURL);
      if (!response2.ok) {
        throw new Error("Failed to fetch data");
      }

      getPapersSummary = await response2.json();

      console.log(getPapersSummary );


      const unreadNotificationCount = await fetch(
        `${import.meta.env.VITE_BACKEND}/notification/unreadCount/${user_id}`
      );
      unreadCount = await unreadNotificationCount.json();
      unreadCount = unreadCount.unreadCount;

    } catch (error) {
      console.error("Error fetching data:", error);
    }
  });

  function handleViewPaper(conference_id) {
    goto(`/${user_id}/conference/mysubmission/${conference_id}`);
  }

</script>

{#if user != null && unreadCount != null && getPapersSummary != null}
  <main>
    <NavbarUser myVariable={unreadCount} />

    <h1>My Profile</h1>

    <p> <b> Name: </b>  {user.first_name} {user.last_name}
    <br>
    <br> 
    <b> Email: </b> {user.email}
    <br>
    <br>
    <b> Date of Birth: </b> {user.date_of_birth}
    <br>
    <br>
    <b> Current Institution: </b> {user.current_institution}
    <br>
    <br> 
    <b> Personal Links: </b>  <br>
    {#each user.personal_links as link}
      <a href={link}>{link}</a> <br>
    {/each}
    <br>
    <br> 
    <b> Expertise: </b> {#each user.expertise as expertise}
      {expertise} ,
    {/each}
    </p>

    <h3>My Publications: </h3>
    <hr />

    {#each getPapersSummary as item}
    <div>
      {#if item.status == "accepted"}
        <h3>Title: {item.paper_title}</h3>
        <h4>Related fields: {item.related_fields}</h4>
        <h4>Conference: {item.conference_title}</h4>
        <h4>Authors: {item.authors}</h4>
        <button
        on:click={() => {
          handleViewPaper(item.conference_id);
        }}>View Paper</button
      >
      <hr />

      {/if}

      
    </div>
  {/each}
  </main>

  {:else} 
  <h1>Loading...</h1>
{/if}

<style>
  main {
    max-width: 90%;
    margin: 2% auto;
  }

  h1 {
  }

  button {
    margin-top: 0%;
    padding: 8px 20px;
    border: none;
    border-radius: 4px;
    background-color: #007bff;
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
