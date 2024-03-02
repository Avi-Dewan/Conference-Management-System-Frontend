<script>
  import { page } from "$app/stores";
  import { DatePicker } from "@svelte-plugins/datepicker";

  const conference_id = $page.params.conference_id;

  import NavbarChair from "/src/components/navbar_chair.svelte";
  import NavbarUser from "/src/components/navbar_user.svelte";

  import { onMount } from "svelte";
  import { goto } from "$app/navigation";
  import "/src/app.css";

  let user_id, user_type;

  user_id = $page.params.user_id;

  let data = [];

  let url = `${import.meta.env.VITE_BACKEND}/conference/${conference_id}`;

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

      const submissionDeadline = new Date(
        `${data.submission_deadline.date}T${data.submission_deadline.time}`
      );
      if (submissionDeadline < currentDate) data.status = "Closed";
      else data.status = "Open";
    } catch (error) {
      console.error("Error fetching data:", error);
    }
  });
  let unreadCount = null;
  onMount(async () => {
    try {
      const unreadNotificationCount = await fetch(
        `${import.meta.env.VITE_BACKEND}/notification/unreadCount/${user_id}`
      );

      unreadCount = await unreadNotificationCount.json();
      unreadCount = unreadCount.unreadCount;
    } catch (error) {
      console.error("Error fetching data:", error);
    }
  });
</script>

{#if data != "" && unreadCount != null}
  <main>
    {#if user_type == "chair"}
      <NavbarChair myVariable={unreadCount} />
    {:else}
      <NavbarUser myVariable={unreadCount} />
    {/if}
    <div>
      <h1 class="mt-10 mb-2">Conference Title: {data.conference_title}</h1>
      <h4>Venue: {data.venue}</h4>
      <h4>
        Webpage: <a href={data.conference_webpage}>{data.conference_webpage}</a>
      </h4>
      <h3 class="mt-4">Research Areas: {data.related_fields}</h3>
      <h4 class="mt-4">Start date: {data.start_date}</h4>
      <h4>End date: {data.end_date}</h4>
      {#if data.status == "Open"}
        <h3 class="mt-4" style="color:green">Status: {data.status}</h3>
      {:else}
        <h3 class="mt-4" style="color:red">Status: {data.status}</h3>
      {/if}
      <h3 class="mt-4" style="color: red;">
        Submission Deadline:
        <p>
          Time: {data.submission_deadline.time} , Date: {data
            .submission_deadline.date}
        </p>
      </h3>
      <h3 class="mt-4">Description:</h3>
      <p class="mt-4 mb-10">{data.conference_description}</p>

      {#if user_type == "chair"}
        <button
          class="btn btn-success"
          on:click={() =>
            goto(`/${user_id}/conference/viewSubmission/${conference_id}`)}
          >View Submissions</button
        >
        <button
          class="btn btn-neutral"
          on:click={() =>
            goto(`/${user_id}/conference/viewPosterSubmission/${conference_id}`)}
          >View Poster Submissions</button
        >        
        <button
          on:click={() =>
            goto(`/${user_id}/conference/create_workshop/${conference_id}`)}
          >Create a workshop</button
        >
        <button
          class="btn btn-success"
          on:click={() =>
            goto(`/${user_id}/conference/view_workshop/${conference_id}`)}
          >View Workshops</button
        >
        <button
          on:click={() =>
            goto(`/${user_id}/conference/edit_conference/${conference_id}`)}
          >Edit Conference</button
        >
        <!-- <button
          on:click={() =>
            goto(`/${user_id}/conference/popular_workshop/${conference_id}`)}
          >Popular Workshops</button
        > -->
      {:else if data.status == "Open"}
        <button
          class="btn btn-neutral"
          on:click={() =>
            goto(`/${user_id}/conference/submitPaper/${conference_id}`)}
          >Submit a Paper</button
        >
        <button
          class="btn btn-success"
          on:click={() =>
            goto(`/${user_id}/conference/submitPoster/${conference_id}`)}
          >Submit a Poster</button
        >
        <button
          on:click={() =>
            goto(`/${user_id}/conference/mysubmission/${conference_id}`)}
          >My submissions</button
        >
        <button
          class="btn btn-neutral"
          on:click={() =>
            goto(`/${user_id}/conference/myPostersubmission/${conference_id}`)}
          >My Poster submissions</button
        >        
        <button
          on:click={() =>
            goto(`/${user_id}/conference/view_workshop_user/${conference_id}`)}
          >View Workshops</button
        >
      {:else if user_type == "user"}
        <button
          class="btn btn-success"
          on:click={() =>
            goto(`/${user_id}/conference/mysubmission/${conference_id}`)}
          >My submissions</button
        >
        <button
          class="btn btn-neutral"
          on:click={() =>
            goto(`/${user_id}/conference/view_workshop_user/${conference_id}`)}
          >View Workshops</button
        >
      {/if}
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
