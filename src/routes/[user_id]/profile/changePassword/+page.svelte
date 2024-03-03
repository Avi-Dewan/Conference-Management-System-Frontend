<script>
  import { page } from "$app/stores";

  import { onMount } from "svelte";

  import { goto } from "$app/navigation";

  import NavbarUser from "/src/components/navbar_user.svelte";

  let user_id, user = null;;
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

  async function changePassword() {
    const response = await fetch(`${import.meta.env.VITE_BACKEND}/auth/changePassword`, {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify(formData),
    });

    const {message, error} = await response.json();
    
    if(error){
      alert(error);
      return;
    }

    alert(message);

    goto(`/${user_id}/home`);
  }

  let formData = {
    oldPassword: null,
    confirmPassword: null, 
    newPassword: null
  };




  function handleSubmit() {


    if(formData.newPassword != formData.confirmPassword){
      alert("Passwords do not match");
      return;
    }

    formData.email = user.email;

    changePassword();

    // alert(JSON.stringify(formData, null, 2));
  }

</script>

<main>

  {#if user != null && unreadCount != null}

    <NavbarUser myVariable={unreadCount} />

    <h1 style="margin-top: 20px;">Change Password</h1>

    <p  style="margin-top: 20px;"> <b> User: </b> {user.first_name} {user.last_name}</p>
    <p  style="margin-top: 20px; margin-bottom:20px"> <b> Email: </b> {user.email}</p>
   
      
      <div class="form-control">
        <label for="password">Password:</label>
        <input
          type="password"
          id="password"
          bind:value={formData.oldPassword}
        />
      </div>

      <div class="form-control">
        <label for="newPassword">New Password:</label>
        <input
          type="password"
          id="newPassword"
          bind:value={formData.newPassword}
        />
      </div>
      
      <div class="form-control">
        <label for="passwordconfirm">Confirm Password:</label>
        <input
          type="password"
          id="passwordconfirm"
          bind:value={formData.confirmPassword}
        />
      </div>


      <div class="form-control" style="display: block;">
        <button on:click={handleSubmit}>Submit</button>
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
