<script>
  import { page } from "$app/stores";
  import { DatePicker } from "@svelte-plugins/datepicker";

  const conference_id = $page.params.conference_id;

  import NavbarChair from "/src/components/navbar_chair.svelte";
  import NavbarUser from "/src/components/navbar_user.svelte";


  import { onMount } from "svelte";

  let user_id, user_type;

  user_id = $page.params.user_id;

  let data = [];

  let url = `http://localhost:3000/conference/${conference_id}`;

  const currentDate = new Date();

  onMount(async () => {
    try {
      
      user_type = sessionStorage.getItem("user_type");

      const response = await fetch(url);
      if (!response.ok) {
        throw new Error("Failed to fetch data");
      }

      data = await response.json();

      const submissionDeadline = new Date(
        `${data.submission_deadline.date}T${data.submission_deadline.time}`
      );
      if (submissionDeadline < currentDate) data.status = "Closed";
      else data.status = "Open";
    } catch (error) {
      console.error("Error fetching data:", error);
    }
  });
</script>

{#if data != ""}
  <main>
    {#if user_type == JSON.stringify('chair')}
      <NavbarChair/>
    {:else}
      <NavbarUser/>
    {/if}
    <div>
      <h1>Conference Title: {data.conference_title}</h1>
      <h3>Venue: {data.venue}</h3>
      <h3>
        Webpage: <a href={data.conference_webpage}>{data.conference_webpage}</a>
      </h3>
      <h2>Research Areas: {data.related_fields}</h2>
      <h3>Start date: {data.start_date}</h3>
      <h3>End date: {data.end_date}</h3>
      {#if data.status == "Open"}
        <h2 style="color:green">Status: {data.status}</h2>
      {:else}
        <h2 style="color:red">Status: {data.status}</h2>
      {/if}
      <h3 style="color: red;">
        Submission Deadline:
        <p>
          Time: {data.submission_deadline.time} , Date: {data
            .submission_deadline.date}
        </p>
      </h3>
      <h3>Description: {data.conference_description}</h3>

      <a href="/{user_id}/conference/submitPaper/{conference_id}"
        >Submit a Paper</a
      >
      <a href="/{user_id}/conference/viewSubmission/{conference_id}"
        >View Submissions</a
      >
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
</style>
