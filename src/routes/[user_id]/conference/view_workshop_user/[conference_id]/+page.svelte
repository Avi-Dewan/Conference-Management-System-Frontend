<script>
  import { page } from "$app/stores";
  import { DatePicker } from "@svelte-plugins/datepicker";

  import { goto } from "$app/navigation";

  const conference_id = $page.params.conference_id;

  import NavbarChair from "/src/components/navbar_chair.svelte";
  import NavbarUser from "/src/components/navbar_user.svelte";

  import { onMount } from "svelte";

  let user_id, user_type;
  let button_label = "Interested";

  user_id = $page.params.user_id;

  let getAllWorkshopURL = `http://localhost:3000/workshop/all/${conference_id}`;

  let getInterstedPeople = `http://localhost:3000/workshop/interested/${conference_id}/${user_id}`;

  let allWorkshops = null;

  let getAllworkshopInterestURL = `http://localhost:3000/workshop/interested/${conference_id}`;

  let interested_workshops = null;

  let instructorFetchUrl = `http://localhost:3000/workshop/suggestTeachers`;

  let suggestedInstructors = null;

  let filterText = "";
  let showAuto = false;

  let filteredSuggestedInstructor = null;

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
        }

        console.log(allWorkshops);
      }

      const interest_response = await fetch(getInterstedPeople);

      if (!interest_response.ok) {
        throw new Error("Failed to fetch data");
      }

      data = await interest_response.json();
      interested_workshops = data;

      console.log(interested_workshops);

      allWorkshops = allWorkshops.map((item) => {
        const found = interested_workshops.some(
          (element) => element.workshop_id === item.workshop_id
        );
        return { ...item, interest: found ? "yes" : "no" };
      });

      console.log(allWorkshops);

      // const interest_response = await fetch(getAllworkshopInterestURL);
      // if(!interest_response.ok){
      //     throw new Error("Failed to fetch interest data");
      // }
      // let interest_data = await interest_response.json();
      // allworkshopInterest = interest_data;

      // if(allworkshopInterest !=  null){
      //     for(let i = 0;i< allworkshopInterest.length;i++){
      //         allworkshopInterest.showSuggest = false;
      //     }
      // }
    } catch (error) {
      console.error("Error fetching data:", error);
    }
  });

  async function updateValue(workshop_id, button_label) {
    let updateData;
    if (button_label === "Interested") {
      updateData = {
        workshop_id: workshop_id,
        user_id: user_id,
        value: "1",
      };
    } else {
      updateData = {
        workshop_id: workshop_id,
        user_id: user_id,
        value: "0",
      };
    }
    const req = await fetch(getAllworkshopInterestURL, {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify(updateData),
    });

    let data = await req.json();

    console.log("eta frontend workshop");

    const thisPage = window.location.pathname;

    goto(`/${user_id}/home`).then(() => goto(thisPage));
  }

  // async function handleAssign(related_fields, workshop_id, item) {
  //   let formData = {
  //     related_fields: related_fields,
  //     workshop_id: workshop_id,
  //   };
  //   const req = await fetch(instructorFetchUrl, {
  //     method: "POST",
  //     headers: {
  //       "Content-Type": "application/json",
  //     },
  //     body: JSON.stringify(formData),
  //   });

  //   let data = await req.json();
  //   suggestedInstructors = data;

  //   filteredSuggestedInstructor = data;

  //   item.showSuggest = !item.showSuggest;

  //   console.log(data);
  // }

  // async function requestInstructor(workshop_id, user_id) {
  //   console.log(workshop_id);
  //   console.log(user_id);
  // }
  let unreadCount = null;
  onMount(async () => {
    try {
      const unreadNotificationCount = await fetch(
        `http://localhost:3000/notification/unreadCount/${user_id}`
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
          <h2>{item.workshop_title}</h2>
          <h3 style="color:black">Related Fields: {item.related_fields}</h3>
          <h4>Description: {item.workshop_description}</h4>

          {#if item.workshop_time != null}
            <h4>Time: {item.workshop_time.time}</h4>
            <h4>Date: {item.workshop_time.date}</h4>
          {:else}
            <h4 style="color: red;">Time: Not yet assigned</h4>
            <h4 style="color: red;">Date: Not yet assigned</h4>

            <!-- <h3>
              Assign time:
              <input type="time" id="workshop_time" />
            </h3>
            <h3>
              Assign Date:
              <input type="date" id="workshop_date" />
            </h3> -->

            <!-- <h3>
              <button
                on:click={() =>
                  handleAssign(item.related_fields, item.workshop_id, item)}
                >Assign instructor</button
              >
            </h3> -->

            <!-- <div class="form-control">
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
            </div> -->

            {#if item.interest == "no"}
              <button on:click={updateValue(item.workshop_id, "Interested")}
                >Interested</button
              >
            {:else}
              <button on:click={updateValue(item.workshop_id, "no")}
                >Not Interested</button
              >
            {/if}

            <!-- <button on:click={updateValue(item.workshop_id)} >Not Interested</button> -->
          {/if}

          <hr />
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
