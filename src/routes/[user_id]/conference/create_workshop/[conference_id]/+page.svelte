<script>
  import { page } from "$app/stores";
  import { DatePicker } from "@svelte-plugins/datepicker";
  import { goto } from "$app/navigation";

  const conference_id = $page.params.conference_id;

  import NavbarChair from "/src/components/navbar_chair.svelte";
  import NavbarUser from "/src/components/navbar_user.svelte";

  import { onMount } from "svelte";

  let user_id, user_type;

  user_id = $page.params.user_id;

  let researchFields = [];
  let addfield = "";
  let formData = {
    workshop_title: "",
    workshop_description: "",
    workshop_webpage: "",
    related_fields: null,
    workshop_length: "",
  };

  let submission_deadline_date = "";
  let submission_deadline_time = "";

  async function createWorkshop() {
    const req = await fetch("http://localhost:3000/workshop/create", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify(formData),
    });
  }

  function handleSubmit() {
    formData.related_fields = researchFields;

    createWorkshop();
    // alert(JSON.stringify(formData, null, 2));
    goto(`/${user_id}/conference/create_workshop/${conference_id}/success`);

    console.log(user_id);
  }

  function handleAdd() {
    // console.log("hello");
    if (addfield != "") {
      researchFields = [...researchFields, String(addfield)];
      addfield = "";
    }
  }
  function removeItem(index) {
    researchFields = researchFields.filter((_, i) => i != index);
  }
</script>

<main>
  <NavbarChair />

  <h1>Create a workshop</h1>

  <div class="form">
    <div class="form-control">
      <label for="workshop_title">Workshop Title:</label>
      <input
        type="text"
        id="workshop_title"
        bind:value={formData.workshop_title}
      />
    </div>

    <div class="form-control">
      <label for="workshop_description">Description:</label>

      <input
        type="text"
        id="workshop_title"
        bind:value={formData.workshop_description}
      />
    </div>
    <div class="form-control">
      <label for="workshop_description">Probable Length (in hours):</label>

      <input
        type="text"
        id="workshop_length"
        bind:value={formData.workshop_length}
      />
    </div>

    <div class="form-control">
      <label for="resarch_field" style="margin-top: 50px;"
        ><h2>Related Track</h2></label
      >

      <div class="two-column">
        {#each researchFields as item, index (item)}
          <div class="two-column">
            <div>
              <label style="width: 1px;"><h3>{item}</h3></label>
            </div>
            <div>
              <button
                on:click={() => removeItem(index)}
                style="background-color:red;"
              >
                Remove</button
              >
            </div>
          </div>
        {/each}
      </div>
    </div>

    <div class="form-control">
      <input type="text" bind:value={addfield} style="width:30%" />
      <button on:click={handleAdd} style="margin-left: 3%;height:40px;">
        Add
      </button>
    </div>

    <div class="form-control" style="display: block;">
      <button on:click={handleSubmit}>Submit</button>
    </div>
  </div>
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
