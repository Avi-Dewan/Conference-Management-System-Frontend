<script>
  import Navbar from "/src/components/navbar.svelte";

  async function createConference() {

    const req = await fetch("http://localhost:3000/conference/create", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify(formData),
    });
  }

  let researchFields = ["aa", "ab", "ac"];
  let addfield = "";
  let formData = {
    conference_title: "",
    conference_description: "",
    conference_webpage: "",
    venue: "",
    start_date: null,
    end_date: null,
    submission_deadline: null,
    related_fields: null,
  };

  let submission_deadline_date = "";
  let submission_deadline_time = "";


  function handleSubmit() {
    formData.related_fields = researchFields;
    formData.submission_deadline = {
          date: submission_deadline_date,
          time: submission_deadline_time,
    }

    createConference();
    // alert(JSON.stringify(formData, null, 2));
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
  <Navbar />

  <h1>Create a conference</h1>

  <div class="form">
    <div class="form-control">
      <label for="conference_title">Title:</label>
      <input type="text" id="conference_title" bind:value={formData.conference_title} />
    </div>

    <div class="form-control">
      <label for="conference_description">Description:</label>
      <input type="text" id="conference_title" bind:value={formData.conference_description} />
    </div>

    <div class="form-control">
      <label for="webpage">Webpage:</label>
      <input type="text" id="webpage" bind:value={formData.conference_webpage} />
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

    <div class="form-control">
      <label for="resarch_field" style="margin-top: 150px;"
        ><h2>Related Fields</h2></label
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
</style>
