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
