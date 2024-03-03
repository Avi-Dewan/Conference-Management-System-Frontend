<script>
  import NavbarChair from "/src/components/navbar_chair.svelte";
  import NavbarUser from "/src/components/navbar_user.svelte";
  import { goto } from "$app/navigation";

  import { page } from "$app/stores";
  import { onMount } from "svelte";

  async function editConference() {
    const req = await fetch(`${import.meta.env.VITE_BACKEND}/conference/${conference_id}`, {
      method: "PUT",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify(formData),
    });
  }

  let user_id, conference_id, conference;

  user_id = $page.params.user_id;
  conference_id = $page.params.conference_id;

  let researchFields = [];
  let addfield = "";
  let formData = {
    conference_title: null,
    conference_description: null,
    conference_webpage: null,
    venue: null,
    start_date: null,
    end_date: null,
    submission_deadline: null,
    chair_id: user_id,
  };

  let submission_deadline_date = "";
  let submission_deadline_time = "";

  function handleSubmit() {
    formData.related_fields = researchFields;

    if (submission_deadline_date != "" || submission_deadline_time != "") {
      formData.submission_deadline = {
        date: submission_deadline_date,
        time: submission_deadline_time,
      };
    }
    
    editConference();
    // alert(JSON.stringify(formData, null, 2));
    goto(`/${user_id}/conference/edit_conference/${conference_id}/success`);
  }

  let unreadCount = null;

  onMount(async () => {
    try {
      const unreadNotificationCount = await fetch(
        `${import.meta.env.VITE_BACKEND}/notification/unreadCount/${user_id}`
      );

      unreadCount = await unreadNotificationCount.json();
      unreadCount = unreadCount.unreadCount;

      const response = await fetch(
        `${import.meta.env.VITE_BACKEND}/conference/${conference_id}`
      );

      conference = await response.json();

      console.log(conference);

    } catch (error) {
      console.error("Error fetching data:", error);
    }
  });
</script>

<main>
  {#if unreadCount != null && conference != null}
    <NavbarChair myVariable={unreadCount} />

    <h1>Edit conference</h1>

    <h2>  {conference.conference_title}</h2>
    <div class="form">
      <div class="form-control">
        <label for="conference_title">Title:</label>
        <input
          type="text"
          id="conference_title"
          bind:value={formData.conference_title}
        />
      </div>

      <div class="form-control">
        <label for="conference_description">Description:</label>

        <input
          type="text"
          id="conference_title"
          bind:value={formData.conference_description}
        />
      </div>

      <div class="form-control">
        <label for="webpage">Webpage:</label>
        <input
          type="text"
          id="webpage"
          bind:value={formData.conference_webpage}
        />
      </div>

      <div class="form-control">
        <label for="venue">Venue:</label>
        <input type="text" id="venue" bind:value={formData.venue} />
      </div>
      <div class="form-control">
        <label for="start_date">Start Date:</label>
        <input type="date" id="venue" bind:value={formData.start_date} />
      </div>
      <div class="form-control">
        <label for="end_date">End Date:</label>
        <input type="date" id="end_date" bind:value={formData.end_date} />
      </div>

      <div class="form-control redFont">
        <label>Submission Deadline</label>
      </div>
      <div class="form-control-submission">
        <div class="column">
          <label for="submission_deadline">Date:</label>
          <input
            type="date"
            id="submission_deadline_date"
            bind:value={submission_deadline_date}
          />
        </div>

        <div class="column">
          <label for="submission_deadline">Time:</label>
          <input
            type="time"
            id="submission_deadline_time"
            bind:value={submission_deadline_time}
          />
        </div>
      </div>

      <div class="form-control" style="display: block;">
        <button on:click={handleSubmit}>Submit</button>
      </div>
    </div>
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
