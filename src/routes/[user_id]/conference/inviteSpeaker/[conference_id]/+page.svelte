<script>
  import { goto } from "$app/navigation";
  import { page } from "$app/stores";
  import { onMount } from "svelte";
  import NavbarChair from "/src/components/navbar_chair.svelte";
  import "/src/app.css";

  const main_user_id = $page.params.user_id;
  const conference_id = $page.params.conference_id;

  let conference_details_url = `${import.meta.env.VITE_BACKEND}/conference/${conference_id}`;
  //let paper_author_url = `${import.meta.env.VITE_BACKEND}/paper/${paper_id}/author`;
  let reviewer_auto_url = `${import.meta.env.VITE_BACKEND}/keynote/assign/auto/${conference_id}`;

  let request_reviwer_url = `${import.meta.env.VITE_BACKEND}/keynote/assign/request`;
  let request_new_reviwer_url = `${import.meta.env.VITE_BACKEND}/keynote/assign/requestNew`;
  let request_delete_url = `${import.meta.env.VITE_BACKEND}/keynote/assign/request_delete`;
  let already_sent_request_url = `${import.meta.env.VITE_BACKEND}/keynote/assign/sent_request/${conference_id}`;

  let reviewer_manual_url = `${import.meta.env.VITE_BACKEND}/keynote/assign/auto/${conference_id}`;

  let already_assigned_reviewer_url = `${import.meta.env.VITE_BACKEND}/keynote/assigned/${conference_id}`;

  let conference_details = null;
  let paper_author_details = null;

  let paper_authors = null;

  let authorSelect;
  let filterText = "";
  let showAuto = false;
  let authors = [];
  let suggestedReviewer = null;
  let suggestedManualReviewer = null;
  let data = [];
  let filteredSuggestedReviewer = [];
  let filteredManualReviewer = [];

  let alreadyRequestedReviewer = null;
  let alreadyAssignedReviewer = null;

  let showManual = false;
  let user_type;

  let paper_title = null;

  let unreadCount = null;

  let form_co_author_without_account = {
    first_name: "",
    last_name: "",
    email: "",
    affiliation: "",
  };

  async function requestReviewerNew(conference_id, conference_title, coAuthor) {
    const response = await fetch(request_new_reviwer_url, {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify({
        conference_id: conference_id,
        conference_title: conference_title,
        coAuthor: coAuthor,
      }),
    });

    const thisPage = window.location.pathname;

    console.log(user_type, "before");
    goto(`/${main_user_id}/home`).then(() => goto(thisPage));
  }

  function handleAdd_CoAuthor() {
    // console.log("hello");
    if (
      form_co_author_without_account.first_name != "" &&
      form_co_author_without_account.last_name != "" &&
      form_co_author_without_account.email != "" &&
      form_co_author_without_account.affiliation != ""
    ) {
      // conference_id, conference_details.conference_title, form_co_author_without_account

      requestReviewerNew(
        conference_id,
        conference_details.conference_title,
        form_co_author_without_account
      );

      form_co_author_without_account = {
        first_name: "",
        last_name: "",
        email: "",
        affiliation: "",
      };
    }
  }

  //paper details
  onMount(async () => {
    try {
      user_type = sessionStorage.getItem("user_type");
      const response = await fetch(conference_details_url);
      if (!response.ok) {
        throw new Error("Failed to fetch data");
      }

      conference_details = await response.json();
      console.log(conference_details);
      //conference_details = conference_details[0];

      console.log("conf det url");
      console.log(conference_details);

      let user_id = main_user_id;
      const unreadNotificationCount = await fetch(
        `${import.meta.env.VITE_BACKEND}/notification/unreadCount/${user_id}`
      );

      unreadCount = await unreadNotificationCount.json();
      unreadCount = unreadCount.unreadCount;
    } catch (error) {
      console.error("Error fetching data:", error);
    }
  });

  //kader kader request kora hoise
  onMount(async () => {
    try {
      const response = await fetch(already_sent_request_url);
      if (!response.ok) {
        throw new Error("Failed to fetch data");
      }

      alreadyRequestedReviewer = await response.json();

      console.log(alreadyRequestedReviewer);
    } catch (error) {
      console.error("Error fetching data:", error);
    }
  });

  //kader kader assign kora hoise
  onMount(async () => {
    try {
      const response = await fetch(already_assigned_reviewer_url);
      if (!response.ok) {
        throw new Error("Failed to fetch data");
      }

      alreadyAssignedReviewer = await response.json();

      console.log(alreadyAssignedReviewer);
    } catch (error) {
      console.error("Error fetching data:", error);
    }
  });

  //kara kara suggested

  onMount(async () => {
    try {
      const response = await fetch(reviewer_auto_url);
      if (!response.ok) {
        throw new Error("Failed to fetch data");
      }

      suggestedReviewer = await response.json();
      filteredSuggestedReviewer = suggestedReviewer;

      console.log(suggestedReviewer);
    } catch (error) {
      console.error("Error fetching data:", error);
    }
  });

  //manually kara kara
  onMount(async () => {
    try {
      const response = await fetch(reviewer_manual_url);
      if (!response.ok) {
        throw new Error("Failed to fetch data");
      }

      suggestedManualReviewer = await response.json();
      filteredManualReviewer = suggestedManualReviewer;
    } catch (error) {
      console.error("Error fetching data:", error);
    }
  });

  async function requestReviewer(conference_id, user_id, conference_title) {
    const response = await fetch(request_reviwer_url, {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify({
        conference_id: conference_id,
        user_id: user_id,
        conference_title: conference_title,
      }),
    });

    const thisPage = window.location.pathname;

    console.log(user_type, "before");
    goto(`/${main_user_id}/home`).then(() => goto(thisPage));
  }

  async function removeRequest(conference_id, user_id) {
    const response = await fetch(request_delete_url, {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify({
        conference_id: conference_id,
        user_id: user_id,
      }),
    });

    const thisPage = window.location.pathname;
    goto(`/${main_user_id}/home`).then(() => goto(thisPage));
  }

  function suggestionSelect() {
    if (filteredSuggestedReviewer != null) {
      showAuto = !showAuto;
      showManual = false;
    }
  }

  function manualsuggestionSelect() {
    if (filteredManualReviewer != null) {
      showManual = !showManual;
      showAuto = false;
    }
  }
</script>

<main>
  {#if unreadCount != null && conference_details != null && suggestedReviewer != null}
    <!-- {#if unreadCount != null && conference_details != null  && suggestedReviewer != null } -->

    <NavbarChair myVariable={unreadCount} />
    <div class="header">
      <h1 style="margin-top: 20px;">
        Title: {conference_details.conference_title}
      </h1>

      <h2 style="margin-top: 20px;">
        Related Field: {conference_details.related_fields}
      </h2>
      <hr class="border-t-2 border-gray-300 my-6" />
    </div>

    <hr class="border-t-2 border-gray-300 my-6" />

    <div>
      <div>
        <h3 style="margin-top: 20px; margin-bottom:10px color:red">
          Already Requested Keynote Speaker
        </h3>
        <hr class="border-t-2 border-gray-300 my-6" />
      </div>

      <div>
        <div class="two-column" style="margin-top: 5px;">
          {#each alreadyRequestedReviewer as item, index (item.user_id)}
            <div class="mt-4">
              <div>
                <div>
                  <b style="margin-top: 20px;">Name: {item.full_name}</b>
                </div>

                <div>
                  <b style="margin-top: 10px;">Expertise: {item.expertise}</b>
                </div>
                <div>
                  <b style="margin-top: 10px;">
                    Affliation: {item.current_institution}
                  </b>
                </div>
                <hr class="border-t-2 border-gray-300 my-6" />
              </div>
              <div class="mt-3">
                <button
                  on:click={() => removeRequest(conference_id, item.user_id)}
                  style="background-color:red;margin-left:0%"
                >
                  Delete Request</button
                >
              </div>
            </div>
          {/each}
        </div>
      </div>
      <br />
      <hr class="border-t-2 border-gray-300 my-6" />
    </div>

    <div>
      <div>
        <h3 style="color: green; margin-top: 10px; margin-bottom: 10px">
          Already Assigned Keynote Speaker
        </h3>
        <hr class="border-t-2 border-gray-300 my-6" />
      </div>

      {#each alreadyAssignedReviewer as item, index (item.user_id)}
        <div>
          <div>
            <b style="margin-top: 20px;">Name: {item.full_name}</b>
          </div>

          <div>
            <b style="margin-top: 10px;">Expertise: {item.expertise}</b>
          </div>
          <div>
            <b style="margin-top: 10px;">
              Affliation: {item.current_institution}
            </b>
          </div>
          <hr class="border-t-2 border-gray-300 my-6" />
        </div>
      {/each}
    </div>

    <div style="margin-top: 10%;">
      <h2>Keynote Speaker Suggestion:</h2>
      <div class="form-control">
        {#if showAuto == true}
          <div class="scrollable-window">
            <input
              type="text"
              placeholder="Filter by name"
              bind:value={filterText}
              style="max-width: 80%;margin-left:5%"
              on:input={() => {
                filteredSuggestedReviewer = suggestedReviewer.filter(
                  (item) =>
                    item.full_name
                      .toLowerCase()
                      .toLowerCase()
                      .indexOf(filterText.toLowerCase()) !== -1
                );

                if (filterText == "") {
                  filteredSuggestedReviewer = suggestedReviewer;
                }
              }}
            />
            {#each filteredSuggestedReviewer as item (item.user_id)}
              <div class="card w-98 text-black-200">
                <div class="card-body bg-gray-100">
                  <h3>Name: {item.full_name}</h3>
                  <h4>Affliation: {item.current_institution}</h4>
                  <h4>Expertise: {item.expertise}</h4>
                  <div class="card-actions justify-end">
                    <button
                      on:click={requestReviewer(
                        conference_id,
                        item.user_id,
                        conference_details.conference_title
                      )}>Request</button
                    >
                  </div>
                </div>
              </div>
            {/each}
          </div>
        {/if}
        <div class="card-actions">
          <button on:click={suggestionSelect}>View Suggestion</button>
        </div>
      </div>
    </div>

    <div class="form-control">
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
                <button
                  on:click={requestReviewer(
                    conference_id,
                    item.user_id,
                    conference_details.conference_title
                  )}>Request</button
                >
              </div>
            </div>
          {/each}
        </div>
      {/if}
      <!-- <button on:click={manualsuggestionSelect}>Assign Manually</button> -->
    </div>

    <div>
      <br />

      <h2 style="margin-top: 10px;">
        Input speaker details if not in suggestion:
      </h2>

      <br />
    </div>

    <div class="form-control">
      <div>
        <label for="first_name">First Name:</label>
        <input
          type="text"
          id="first_name"
          bind:value={form_co_author_without_account.first_name}
        />
      </div>

      <div>
        <label for="last_name"> Last Name:</label>
        <input
          type="text"
          id="last_name"
          bind:value={form_co_author_without_account.last_name}
        />
      </div>
    </div>

    <div class="form-control">
      <label for="email">Email:</label>
      <input
        type="text"
        id="email"
        bind:value={form_co_author_without_account.email}
      />
    </div>

    <div class="form-control">
      <label for="affiliation">Affiliation:</label>
      <input
        type="text"
        id="affiliation"
        bind:value={form_co_author_without_account.affiliation}
      />
    </div>

    <div>
      <button
        class="btn btn-success"
        on:click={handleAdd_CoAuthor}
        style="height:40px;"
      >
        Invite Speaker
      </button>
    </div>
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
