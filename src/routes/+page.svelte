<script>
  import { goto } from "$app/navigation";

  import { onMount } from "svelte";

  function handleSignUp() {
    goto(`/signup`);
  }

  function handleLogIn() {
    goto(`/login/normal`);
  }
</script>

<main>
  <h1>Conference Management System</h1>

  <br>
  <br>
  <br>
  
  <div class="container">

    <div class="text">
      <h2 style="text-align: center;">Manage Your Conference <br>
        Easily and Effectively with <br>
        
        Our Conference Management Software </h2>

        <div class="form-control" style="display: block; text-align: center;">
          <button on:click={handleSignUp}>Sign Up</button>
          <button on:click={handleLogIn}>Login</button>
        </div>
    </div>

    <div class="image">
      <img src="/src/images/conference.jpg" alt="Conference" style="width: 1200px; height: 800px;">
    </div>
    
  </div>

  <br>
  <br>
  <br>
  <br>

 

 

</main>

<style>

  main {
    max-width: 90%;
    margin: 2% auto;
  }

  

  .container {
    display: flex;
    align-items: center;
  }
  .text {
    flex: 1;
  }
  .image {
    flex: 1;
  }

  .image img {
    border-radius: 20px; /* Adjust as needed */
  }
 


  .form-control {
    margin-bottom: 20px;
    min-height: 20px;
    min-width: 50px;
    clear: both;
  }

 

  .form-control button {
    margin-top: 5%;
    margin-left: 10px;
    margin-right: 10px;
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

</style>
