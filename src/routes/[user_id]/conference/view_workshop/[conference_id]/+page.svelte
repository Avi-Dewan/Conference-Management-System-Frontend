<script>
  import { page } from "$app/stores";
  import { DatePicker } from "@svelte-plugins/datepicker";
  import { goto } from "$app/navigation";
  const conference_id = $page.params.conference_id;
  const main_user_id = $page.params.user_id;

  import NavbarChair from "/src/components/navbar_chair.svelte";
  import NavbarUser from "/src/components/navbar_user.svelte";

  import "/src/app.css";
  import { onMount } from "svelte";

  let user_id, user_type;

  let workshop_time, workshop_date;

  user_id = $page.params.user_id;

  let getAllWorkshopURL = `${import.meta.env.VITE_BACKEND}/workshop/all/${conference_id}`;

  let allWorkshops = null;

  // let instructor_auto_url = `${import.meta.env.VITE_BACKEND}/workshop/suggestTeachers`;

  let suggestedInstructors = null;

  let filterText = "";
  let showAuto = false;

  let filteredSuggestedInstructor = null;

  let request_instructor_url = `${import.meta.env.VITE_BACKEND}/workshop/request`;

  let instructor_auto_url = `${import.meta.env.VITE_BACKEND}/workshop/auto_suggest`;

  let updateData_url = `${import.meta.env.VITE_BACKEND}/workshop/updateData`;

  onMount(async () => {
    try {
      user_type = sessionStorage.getItem("user_type");

      console.log(user_type, "in conference");
      const response = await fetch(getAllWorkshopURL);
      if (!response.ok) {
        throw new Error("Failed to fetch data");
      }

      let data = await response.json();
      allWorkshops = data;

      if (allWorkshops != null) {
        for (let i = 0; i < allWorkshops.length; i++) {
          allWorkshops[i].showSuggest = false;
          allWorkshops[i].workshop_time = JSON.parse(
            allWorkshops[i].workshop_time
          );

          let already_sent_request_url = `${import.meta.env.VITE_BACKEND}/workshop/sent_request/${allWorkshops[i].workshop_id}`;
          const workshop_response = await fetch(already_sent_request_url);
          if (!workshop_response.ok) {
            throw new Error("Failed to fetch data");
          }

          let alreadyRequestedInstructor = await workshop_response.json();

          let already_assigned_reviewer_url = `${import.meta.env.VITE_BACKEND}/workshop/accepted_request/${allWorkshops[i].workshop_id}`;
          const accepted_response = await fetch(already_assigned_reviewer_url);
          if (!accepted_response.ok) {
            throw new Error("Failed to fetch data");
          }

          let alreadyAssignedInstructor = await accepted_response.json();

          console.log(alreadyAssignedInstructor);

          console.log(alreadyRequestedInstructor);

          allWorkshops[i]["requested"] = [];
          allWorkshops[i]["assigned"] = [];

          alreadyRequestedInstructor.forEach((obj, index) => {
            allWorkshops[i]["requested"].push(obj.full_name);
          });

          alreadyAssignedInstructor.forEach((obj, index) => {
            allWorkshops[i]["assigned"].push(obj.full_name);
          });

          console.log("hahahahahahahahahahahahah");
        }

        console.log(allWorkshops);
      }
    } catch (error) {
      console.error("Error fetching data:", error);
    }
  });

  //   //kader kader request kora hoise
  //   onMount(async () => {
  //   try {

  //     let already_sent_request_url = `${import.meta.env.VITE_BACKEND}/assign/sent_request/${paper_id}`;
  //     const response = await fetch(already_sent_request_url);
  //     if (!response.ok) {
  //       throw new Error("Failed to fetch data");
  //     }

  //     alreadyRequestedReviewer = await response.json();

  //     console.log(alreadyRequestedReviewer);
  //   } catch (error) {
  //     console.error("Error fetching data:", error);
  //   }
  // });

  async function handleAssign(related_fields, workshop_id, item) {
    let formData = {
      related_fields: related_fields,
      workshop_id: workshop_id,
    };
    const req = await fetch(instructor_auto_url, {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify(formData),
    });

    let data = await req.json();
    suggestedInstructors = data;

    filteredSuggestedInstructor = data;

    item.showSuggest = !item.showSuggest;

    console.log(data);
  }

  async function requestInstructor(workshop_id, user_id) {
    // identical to requestReviewer
    console.log(workshop_id);
    console.log(user_id);

    const response = await fetch(request_instructor_url, {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify({ workshop_id: workshop_id, user_id: user_id }),
    });

    const thisPage = window.location.pathname;

    console.log(user_type, "before");
    goto(`/${main_user_id}/home`).then(() => goto(thisPage));
  }

  async function updateData(workshop_id) {
    const response = await fetch(updateData_url, {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify({
        workshop_id: workshop_id,
        workshop_time: workshop_time,
        workshop_date: workshop_date,
      }),
    });

    const thisPage = window.location.pathname;

    console.log(user_type, "before");
    goto(`/${main_user_id}/home`).then(() => goto(thisPage));
  }

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

<main>
  {#if unreadCount != null}
    {#if user_type == "chair"}
      <NavbarChair myVariable={unreadCount} />
    {:else}
      <NavbarUser myVariable={unreadCount} />
    {/if}
    {#if allWorkshops != null}
      <div class="cards">
        {#each allWorkshops as item}
          <hr class="border-t-2 border-gray-300 my-6" />
          <h2 class="mt-6">{item.workshop_title}</h2>
          <div class="mt-4">
            <b class="mt-5" style="color:black"
              >Related Fields: {item.related_fields}</b
            >
          </div>
          <hr class="border-t-2 border-gray-300 my-6" />
          <h4 class="mt-5 bg-black-200">Description:</h4>
          <p>{item.workshop_description}</p>
          <hr class="border-t-2 border-gray-300 my-6" />

          <div class="mt-3">
            <b>Interested People: {item.count}</b>
          </div>

          <hr class="border-t-2 border-gray-300 my-6" />
          <div class="mt-5">
            <b>Requested Instructor: {item.requested}</b>
          </div>

          <hr class="border-t-2 border-gray-300 my-6" />
          <div class="mt-5">
            <b>Assigned Instructor: {item.assigned}</b>
          </div>
          <hr class="border-t-2 border-gray-300 my-6" />
          <!-- {#each item.requested as requested}
          <h3>{requested.full_name} , {requested.current_institution}</h3>
        {/each} -->
          <div class="mt-5 mb-5">
            {#if item.workshop_time != null}
              <b style="color:red;">Time: {item.workshop_time.time}</b>
              <b style="color:red;">Date: {item.workshop_time.date}</b>
            {:else}
              <b style="color: red;">Time & Date not yet assigned</b>
            {/if}
            <hr class="border-t-2 border-gray-300 my-6" />
          </div>
          <div>
            <b>
              Assign time:
              <input
                type="time"
                placeholder="Type here"
                class="input input-bordered input-success w-full max-w-xs"
                bind:value={workshop_time}
              />
            </b>
          </div>
          <div class="mt-4">
            <b>
              Assign Date:
              <input
                type="date"
                placeholder="Type here"
                class="input input-bordered input-success w-full max-w-xs"
                bind:value={workshop_date}
              />
            </b>
          </div>

          <div class="form-control">
            {#if item.showSuggest == true}
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
                {#each filteredSuggestedInstructor as item_instructor}
                  <div class="card">
                    <h3>Name: {item_instructor.full_name}</h3>
                    <h4>Affliation: {item_instructor.current_institution}</h4>
                    <h4>Expertise: {item_instructor.expertise}</h4>

                    <button
                      on:click={requestInstructor(
                        item.workshop_id,
                        item_instructor.user_id
                      )}>Request</button
                    >
                  </div>
                {/each}
              </div>
            {/if}
          </div>

          <button on:click={updateData(item.workshop_id)}>Update</button>
          <button
            on:click={() => {
              window.location.href = `/${user_id}/conference/assignInstructor/${item.workshop_id}`;
            }}>Assign</button
          >
          <!-- {/if} -->

          <hr />
          <hr class="border-t-2 border-gray-300 my-6" />
        {/each}
      </div>
    {/if}
  {/if}
</main>

<style>
  main {
    max-width: 90%;
    margin: 2% auto;
  }

  h1 {
  }
  .cards input {
    height: 30px;
    width: 150px;

    padding: 8px;
    border: 1px solid #ccc;
    border-radius: 4px;
    box-sizing: border-box;
    font-size: 16px;
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
    margin-left: 450px;
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
