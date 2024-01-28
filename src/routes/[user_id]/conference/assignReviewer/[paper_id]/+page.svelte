<script>
  import { goto } from "$app/navigation";
  import { page } from "$app/stores";
  import { onMount } from "svelte";
  import NavbarChair from "/src/components/navbar_chair.svelte";

  const user_id = $page.params.user_id;
  const paper_id = $page.params.paper_id;

  console.log(user_id);

  let paper_details_url = `http://localhost:3000/paper/${paper_id}`;
  let paper_author_url = `http://localhost:3000/paper/${paper_id}/author`;
  let reviewer_auto_url = `http://localhost:3000/assign/auto/${paper_id}`;
  let request_reviwer_url = `http://localhost:3000/assign/request`;
  let request_delete_url = `http://localhost:3000/assign/request_delete`;

  let reviewer_manual_url = `http://localhost:3000/assign/manual/${paper_id}`;

  let paper_details = null;
  let paper_author_details = null;

  let paper_authors = null;

  let authorSelect;
  let filterText = "";
  let showAuto = false;
  let authors = [];
  let suggestedReviewer = null;
  let data = [];
  let filteredSuggestedReviewer = [];
  //paper details
  onMount(async () => {
    try {
      const response = await fetch(paper_details_url);
      if (!response.ok) {
        throw new Error("Failed to fetch data");
      }

      paper_details = await response.json();

      paper_details = paper_details[0];
      console.log(paper_details);
    } catch (error) {
      console.error("Error fetching data:", error);
    }
  });

  onMount(async () => {
    try {
      const response = await fetch(paper_author_url);
      if (!response.ok) {
        throw new Error("Failed to fetch data");
      }

      paper_author_details = await response.json();
      paper_author_details = paper_author_details;

      console.log(paper_author_details);
    } catch (error) {
      console.error("Error fetching data:", error);
    }
  });

  //kader kader request kora hoise
  onMount(async () => {
    try {
      const response = await fetch(reviewer_auto_url);
      if (!response.ok) {
        throw new Error("Failed to fetch data");
      }

      suggestedReviewer = await response.json();
      filteredSuggestedReviewer = suggestedReviewer;
    } catch (error) {
      console.error("Error fetching data:", error);
    }
  });

  //   //kader kader assign kora hoise
  //   onMount(async () => {
  //     try {
  //       const response = await fetch(url);
  //       if (!response.ok) {
  //         throw new Error("Failed to fetch data");
  //       }

  //       data = await response.json();

  //       console.log(data);
  //     } catch (error) {
  //       console.error("Error fetching data:", error);
  //     }
  //   });

  //kara kara suggested

  onMount(async () => {
    try {
      const response = await fetch(paper_author_url);
      if (!response.ok) {
        throw new Error("Failed to fetch data");
      }

      paper_author_details = await response.json();
      paper_author_details = paper_author_details;
    } catch (error) {
      console.error("Error fetching data:", error);
    }
  });

  async function requestReviewer(paper_id, user_id) {
    const response = await fetch(request_reviwer_url, {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify({ paper_id: paper_id, user_id: user_id }),
    });

    const thisPage = window.location.pathname;
    goto(`/${user_id}/home`).then(() => goto(thisPage));
  }
  async function removeRequest(paper_id, user_id) {
    const response = await fetch(request_delete_url, {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify({ paper_id: paper_id, user_id: user_id }),
    });

    const thisPage = window.location.pathname;
    goto(`/${user_id}/home`).then(() => goto(thisPage));
  }

  function suggestionSelect() {
    if (filteredSuggestedReviewer != null) {
      showAuto = !showAuto;
    }
  }
</script>

<main>
  {#if paper_details != null && paper_author_details != null && suggestedReviewer != null}
    <NavbarChair />
    <div>
      <h1>Title: {paper_details.paper_title}</h1>
      <h3>Authors: {paper_author_details}</h3>
      <h3>Related Field: {paper_details.related_fields}</h3>
    </div>

    {#if showAuto == true}
      <div class="two-column">
        {#each filteredSuggestedReviewer as item, index (item.user_id)}
          <div class="two-column">
            <div>
              <label style="width: 1px;"><h4>{item.full_name}</h4></label>
            </div>
            <div>
              <button
                on:click={() => removeRequest(paper_id, item.user_id)}
                style="background-color:red;"
              >
                Request</button
              >
            </div>
          </div>
        {/each}
      </div>
    {/if}

    <div class="form-control">
      <h2>Reviewer Suggestion:</h2>

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
      <button on:click={suggestionSelect}>View Suggestion</button>
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
</style>
