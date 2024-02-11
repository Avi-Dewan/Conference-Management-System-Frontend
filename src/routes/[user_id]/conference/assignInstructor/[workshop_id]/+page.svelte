<script>
  import { goto } from "$app/navigation";
  import { page } from "$app/stores";
  import { onMount } from "svelte";
  import NavbarChair from "/src/components/navbar_chair.svelte";

  const main_user_id = $page.params.user_id;
  const workshop_id = $page.params.workshop_id;

  // let paper_details_url = `http://localhost:3000/paper/${paper_id}`;
  // let paper_author_url = `http://localhost:3000/paper/${paper_id}/author`;
  // let reviewer_auto_url = `http://localhost:3000/assign/auto/${paper_id}`;

  // let request_reviwer_url = `http://localhost:3000/assign/request`;
  // let request_delete_url = `http://localhost:3000/assign/request_delete`;
  // let already_sent_request_url = `http://localhost:3000/assign/sent_request/${paper_id}`;

  // let reviewer_manual_url = `http://localhost:3000/assign/manual/${paper_id}`;

  // let already_assigned_reviewer_url = `http://localhost:3000/reviewer/assigned/${paper_id}`;

  let workshop_details = null;
  let paper_author_details = null;

  let paper_authors = null;

  let authorSelect;
  let filterText = "";
  let showAuto = false;
  let authors = [];
  let suggestedInstructors = null;
  let suggestedManualReviewer = null;
  let data = [];
  let filteredSuggestedInstructor = [];
  let filteredManualReviewer = [];

  let alreadyRequestedInstructor = null;
  let alreadyAssignedInstructor = null;

  let showManual = false;
  let user_type;
  //workshop details
  onMount(async () => {
    try {
      user_type = sessionStorage.getItem("user_type");

      let workshop_details_url = `http://localhost:3000/workshop/${workshop_id}`;
      const response = await fetch(workshop_details_url);
      if (!response.ok) {
        throw new Error("Failed to fetch data");
      }

      workshop_details = await response.json();

      workshop_details = workshop_details[0];

      console.log(workshop_details);
    } catch (error) {
      console.error("Error fetching data:", error);
    }
  });

  // onMount(async () => {
  //   try {
  //     const response = await fetch(paper_author_url);
  //     if (!response.ok) {
  //       throw new Error("Failed to fetch data");
  //     }

  //     paper_author_details = await response.json();
  //     paper_author_details = paper_author_details;
  //   } catch (error) {
  //     console.error("Error fetching data:", error);
  //   }
  // });

  //kader kader request kora hoise
  onMount(async () => {
    try {
      let already_sent_request_url = `http://localhost:3000/workshop/sent_request/${workshop_id}`;
      const response = await fetch(already_sent_request_url);
      if (!response.ok) {
        throw new Error("Failed to fetch data");
      }

      alreadyRequestedInstructor = await response.json();

      // console.log(alreadyRequestedReviewer);
    } catch (error) {
      console.error("Error fetching data:", error);
    }
  });

  //kader kader assign kora hoise
  onMount(async () => {
    try {
      let already_assigned_Instructor_url = `http://localhost:3000/workshop/accepted_request/${workshop_id}`;

      const response = await fetch(already_assigned_Instructor_url);
      if (!response.ok) {
        throw new Error("Failed to fetch data");
      }

      alreadyAssignedInstructor = await response.json();

      console.log(alreadyAssignedInstructor);
    } catch (error) {
      console.error("Error fetching data:", error);
    }
  });

  //kara kara suggested
  let unreadCount = null;
  onMount(async () => {
    try {
      let instructor_auto_url = `http://localhost:3000/workshop/auto_suggest/${workshop_id}`;
      const response = await fetch(instructor_auto_url);
      if (!response.ok) {
        throw new Error("Failed to fetch data");
      }

      suggestedInstructors = await response.json();
      filteredSuggestedInstructor = suggestedInstructors;

      const unreadNotificationCount = await fetch(
        `http://localhost:3000/notification/unreadCount/${user_id}`
      );

      unreadCount = await unreadNotificationCount.json();
      unreadCount = unreadCount.unreadCount;
      console.log(suggestedInstructors);
    } catch (error) {
      console.error("Error fetching data:", error);
    }
  });

  // //manually kara kara
  // onMount(async () => {
  //   try {
  //     const response = await fetch(reviewer_manual_url);
  //     if (!response.ok) {
  //       throw new Error("Failed to fetch data");
  //     }

  //     suggestedManualReviewer = await response.json();
  //     filteredManualReviewer = suggestedManualReviewer;
  //   } catch (error) {
  //     console.error("Error fetching data:", error);
  //   }
  // });

  async function requestInstructor(workshop_id, user_id) {
    let request_instructor_url = `http://localhost:3000/workshop/request`;
    const response = await fetch(request_instructor_url, {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify({ workshop_id: workshop_id, user_id: user_id }),
    });

    let conf_data = await fetch(
      `http://localhost:3000/conference/${workshop_details.conference_id}`
    );
    if (!conf_data.ok) {
      throw new Error("Failed to fetch data");
    }

    conf_data = await conf_data.json();

    let notification_body = `Request for being an instructor for a workshop on the title ${workshop_details.workshop_title}, in the conference ${conf_data.conference_title}  `;

    let notification_json = {
      type: "workshop_request_notify",
      workshop_id: workshop_id,
    };

    let notification_response = await fetch(
      "http://localhost:3000/notification/send",
      {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify({
          user_id: user_id,
          notification_body: notification_body,
          notification_json: notification_json,
        }),
      }
    );

    let notification_data = await notification_response.json();

    const thisPage = window.location.pathname;

    console.log(user_type, "before");
    goto(`/${main_user_id}/home`).then(() => goto(thisPage));
  }

  async function removeRequest(workshop_id, user_id) {
    let request_delete_url = `http://localhost:3000/workshop/request_delete`;
    const response = await fetch(request_delete_url, {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify({ workshop_id: workshop_id, user_id: user_id }),
    });

    const thisPage = window.location.pathname;
    goto(`/${main_user_id}/home`).then(() => goto(thisPage));
  }

  function suggestionSelect() {
    if (filteredSuggestedInstructor != null) {
      showAuto = !showAuto;
      showManual = false;
    }
  }

  // function manualsuggestionSelect() {
  //   if (filteredManualReviewer != null) {
  //     showManual = !showManual;
  //     showAuto = false;
  //   }
  // }
</script>

<main>
  {#if workshop_details != null && alreadyAssignedInstructor != null && alreadyRequestedInstructor != null && unreadCount != null}
    <NavbarChair myVariable={unreadCount} />
    <div>
      <h1>Title: {workshop_details.workshop_title}</h1>
      <!-- <hr />
        <h2>Authors:</h2>

        
        {#each paper_author_details as item}
          <h3>{item.full_name} , {item.current_institution}</h3>
        {/each}
        <hr /> -->
      <h2>Related Field: {workshop_details.related_fields}</h2>
      <hr />
    </div>
    <div>
      <h3 style="color:red">Already Requested Instructor</h3>
      <div>
        <div class="two-column">
          {#each alreadyRequestedInstructor as item, index (item.user_id)}
            <div>
              <h3>Name: {item.full_name}</h3>
              <h4>Expertise: {item.expertise}</h4>
              <h4>Affliation: {item.current_institution}</h4>

              <button
                on:click={() => removeRequest(workshop_id, item.user_id)}
                style="background-color:red;margin-left:0%"
              >
                Delete Request</button
              >
            </div>
          {/each}
        </div>
      </div>
    </div>
    <hr />

    <div>
      <h3 style="color: green;">Already Assigned Instructor</h3>

      {#each alreadyAssignedInstructor as item}
        <div>
          <h3>Name: {item.full_name}</h3>
          <h4>Expertise: {item.expertise}</h4>
          <h4>Affliation: {item.current_institution}</h4>
          <hr />
        </div>
      {/each}
    </div>
    <div style="margin-top: 10%;">
      <h2>Workshop Instructor Suggestion:</h2>
      <div class="form-control">
        {#if showAuto == true}
          <div class="scrollable-window">
            <input
              type="text"
              placeholder="Filter by name"
              bind:value={filterText}
              style="max-width: 80%;margin-left:5%"
              on:input={() => {
                filteredSuggestedInstructor = suggestedInstructors.filter(
                  (item) =>
                    item.full_name
                      .toLowerCase()
                      .toLowerCase()
                      .indexOf(filterText.toLowerCase()) !== -1
                );

                if (filterText == "") {
                  filteredSuggestedInstructor = suggestedInstructors;
                }
              }}
            />
            {#each filteredSuggestedInstructor as item (item.user_id)}
              <div>
                <div class="card">
                  <h3>Name: {item.full_name}</h3>
                  <h4>Affliation: {item.current_institution}</h4>
                  <h4>Expertise: {item.expertise}</h4>
                  <button
                    on:click={requestInstructor(workshop_id, item.user_id)}
                    >Request</button
                  >
                </div>
              </div>
            {/each}
          </div>
        {/if}
        <button on:click={suggestionSelect}>View Suggestion</button>
      </div>
    </div>

    <!-- <div class="form-control">
        {#if showManual == true}
          <div class="scrollable-window">
            <input
              type="text"
              placeholder="Filter by name"
              bind:value={filterText}
              style="max-width: 80%;margin-left:5%"
              on:input={() => {
                filteredManualReviewer = suggestedManualReviewer.filter(
                  (item) =>
                    item.full_name
                      .toLowerCase()
                      .toLowerCase()
                      .indexOf(filterText.toLowerCase()) !== -1
                );
  
                if (filterText == "") {
                  filteredManualReviewer = suggestedManualReviewer;
                }
              }}
            />
            {#each filteredManualReviewer as item (item.user_id)}
              <div>
                <div class="card">
                  <h3>Name: {item.full_name}</h3>
                  <h4>Affliation: {item.current_institution}</h4>
                  <h4>Expertise: {item.expertise}</h4>
                  <button on:click={requestReviewer(paper_id, item.user_id)}
                    >Request</button
                  >
                </div>
              </div>
            {/each}
          </div>
        {/if}
        <button on:click={manualsuggestionSelect}>Assign Manually</button>
      </div> -->
  {/if}
</main>

<style>
  main {
    max-width: 90%;
    margin: 2% auto;
  }

  h1 {
  }

  .card {
    margin-left: 5%;
    margin-top: 2%;
    border: 1px solid grey;
    max-width: 80%;
    text-align: center;
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
    margin-bottom: 10px;
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

  .scrollable-window {
    height: 300px; /* Adjust height as needed */
    overflow-y: auto;
    border: 1px solid #ccc;
    padding: 10px;
  }
  .scrollable-window button {
    margin-left: 500px;
    margin-bottom: 20px;
    height: 20%; /* Adjust height as needed */
    overflow-y: auto;
    border: 1px solid #ccc;
    padding: 10px;
  }
  .two-column {
    margin-top: -1%;
    column-count: 2; /* Split into two columns */
    column-gap: 30%; /* Adjust gap between columns */
  }
  .two-column button {
    margin-left: -10%;
  }
  .two-column div {
    break-inside: avoid; /* Prevent breaking elements between columns */
  }
</style>
