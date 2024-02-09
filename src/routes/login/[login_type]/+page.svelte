<script>
  import { goto } from "$app/navigation";
  import { page } from "$app/stores";
  import { onMount } from "svelte";


  
  let email, password;
  let login_type, user_id;
  let login_type_array = []



  onMount(async () => {
    try {
      login_type = $page.params.login_type;

      // Split the login_type string into an array
      login_type_array = login_type.split('_');

      console.log(login_type_array[0]);

    } catch (error) {
      console.error("Error fetching data:", error);
    }
  });
  
  async function handleLogIn() {
    const response = await fetch("http://localhost:3000/auth/login", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      credentials: "include",
      body: JSON.stringify({ email: email, password: password }),
    });

    let {message, error} = await response.json();
    
    if(error) {
      alert(error);
      
    } else {
      
      const tokenResponse = await fetch("http://localhost:3000/auth", {
        headers: {
            "Content-Type": "application/json",
          },
        credentials: "include",
      });

      let tokenData = await tokenResponse.json();
      user_id = tokenData.user_id;

      if(login_type == 'normal') {
        goto(`/${user_id}/home`);
      } else {
        alert("special case");
      }

    
    }

    // if (user_id) {
    //   goto(`/${user_id}/home`);
    // } else {
    //   alert("Wrong password");
    // }
  }

  function handleSignUp() {
    goto(`/signup`);
  }
</script>

<main>
  <div >
    <h1>Conference Management System</h1>
    <h3>Manage your conference 
      easily and effectively 
      with our conference management software </h3>
  </div>

  <br>
  <br>
  <br>
  <br>

  <div class="form">
    <div class="form-control">
      <label for="email">Email:</label>
      <input type="text" id="email" bind:value={email} />
    </div>

    <div class="form-control">
      <label for="pasword">Password:</label>

      <input type="password" id="password" bind:value={password} />
    </div>
  </div>

  <div class="form-control" style="display: block;">
    
    <button on:click={handleLogIn}>Log in</button>

    <button on:click={handleSignUp}>Sign Up</button>
  
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
