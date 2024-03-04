<script>
  import { page } from "$app/stores";

  import { onMount } from "svelte";

  import { goto } from "$app/navigation";

  import NavbarUser from "/src/components/navbar_user.svelte";
  import "/src/app.css";

  let user_id,
    user = null;
  let unreadCount = null;

  user_id = $page.params.user_id;

  let getPapersURL = `${import.meta.env.VITE_BACKEND}/paper/allPapers/${user_id}`;

  onMount(async () => {
    try {
      let url = `${import.meta.env.VITE_BACKEND}/user/${user_id}`;

      const response = await fetch(url);

      if (!response.ok) {
        throw new Error("Failed to fetch data");
      }

      user = await response.json();

      console.log(user);

      const unreadNotificationCount = await fetch(
        `${import.meta.env.VITE_BACKEND}/notification/unreadCount/${user_id}`
      );
      unreadCount = await unreadNotificationCount.json();
      unreadCount = unreadCount.unreadCount;
    } catch (error) {
      console.error("Error fetching data:", error);
    }
  });

  async function editProfile() {
    const response = await fetch(
      `${import.meta.env.VITE_BACKEND}/user/${user_id}`,
      {
        method: "PUT",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify(formData),
      }
    );

    const data = await response.json();

    goto(`/${user_id}/profile/myProfile`);
  }

  let expertise = [];
  let addfield = "";

  let personal_links = [];
  let link = "";

  let formData = {
    first_name: null,
    last_name: null,
    date_of_birth: null,
    email: null,
    current_institution: null,
    personal_links: null,
    expertise: null,
  };

  function handleSubmit() {
    formData.expertise = expertise;
    formData.personal_links = personal_links;

    // console.log(formData);
    editProfile();
  }

  function handleAddExpertise() {
    // console.log("hello");
    if (addfield != "") {
      expertise = [...expertise, String(addfield)];
      addfield = "";
    }
  }

  function removeItemExpertise(index) {
    expertise = expertise.filter((_, i) => i != index);
  }

  function handleAddLink() {
    // console.log("hello");
    if (link != "") {
      personal_links = [...personal_links, String(link)];
      link = "";
    }
  }

  function removeItemLink(index) {
    personal_links = personal_links.filter((_, i) => i != index);
  }
</script>

<main>
  {#if user != null && unreadCount != null}
    <NavbarUser myVariable={unreadCount} />

    <h1 style="margin-top: 30px;">Edit Profile</h1>

    <div class="form" style="margin-top: 30px;">
      <div class="form-control">
        <label for="first_name">First Name:</label>
        <input type="text" id="first_name" bind:value={formData.first_name} />
      </div>

      <div class="form-control">
        <label for="last_name">Last Name:</label>

        <input type="text" id="clast_name" bind:value={formData.last_name} />
      </div>

      <div class="form-control">
        <label for="date_of_birth">Date of Birth:</label>
        <input
          type="date"
          id="date_of_birth"
          bind:value={formData.date_of_birth}
        />
      </div>

      <div class="form-control">
        <label for="current_institution">Current Affiliation:</label>
        <input
          type="text"
          id="current_institution"
          bind:value={formData.current_institution}
        />
      </div>

      <div class="form-control">
        <label for="email">Email:</label>
        <input type="text" id="email" bind:value={formData.email} />
      </div>

      <div class="form-control">
        <label for="personal_links" style="margin-top: 10px;"
          ><h2>Personal Links</h2></label
        >
        <br />
        <div class="two-column">
          {#each personal_links as item, index (item)}
            <div class="two-column">
              <div style="margin-top: 10px;">
                <label style="width: 1px;"><h3>{item}</h3></label>
              </div>
              <div>
                <button
                  on:click={() => removeItemLink(index)}
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
        <input type="text" bind:value={link} style="width:30%" />
        <div style="margin-top: 1px;" class="card-actions justify">
          <button class="btn btn-success" on:click={handleAddLink}>
            Add
          </button>
        </div>
      </div>

      <div class="form-control">
        <label for="resarch_field" style="margin-top: 10px;"
          ><h2>Expertise</h2></label
        >
        <br />
        <div class="two-column">
          {#each expertise as item, index (item)}
            <div class="two-column">
              <div>
                <label style="width: 1px;"><h3>{item}</h3></label>
              </div>
              <div>
                <button
                  on:click={() => removeItemExpertise(index)}
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
        <div style="margin-top: 1px;" class="card-actions justify">
          <button on:click={handleAddExpertise} class="btn btn-success">
            Add
          </button>
        </div>
      </div>

      <div class="form-control" style="display: block;">
        <button class="btn btn-accent" on:click={handleSubmit}>Submit</button>
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
