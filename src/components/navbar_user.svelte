<script>
  import { page } from "$app/stores";

  let user_id;

  user_id = $page.params.user_id;

  export let myVariable;
</script>

<header>
  <div class="container">
    <h1 class="logo"></h1>

    <nav>
      <ul>
        <li><a href="/{user_id}/home">Home</a></li>
        <li>
          <a href="/{user_id}/conference/conference_list/all">Conferences</a>
        </li>
        <li><a href="/{user_id}/Request">My Request</a></li>
        <!-- <li><a href="/{user_id}/workshop_request">Workshop Request</a></li>
        <li><a href="/{user_id}/author_request">co_authors Request</a></li> -->
        <li><a href="/{user_id}/review">My Review</a></li>

        {#if myVariable > 0}
          <li>
            <a href="/{user_id}/notification" style="color: red;"
              >My notification [{myVariable}]</a
            >
          </li>
        {:else}
          <li><a href="/{user_id}/notification">My notification</a></li>
        {/if}
        <li><a href="/{user_id}/dashboardUser">Dashboard</a></li>
        <li><a href="/{user_id}/profile/myProfile">My Profile</a></li>
        <li><a href="/">Log out</a></li>
      </ul>
    </nav>
  </div>
</header>

<style>
  @import url("https://fonts.googleapis.com/css?family=Work+Sans:400,600");
  body {
    margin: 0;
    background: #e4ebe1;
    font-family: "Work Sans", sans-serif;
    font-weight: 800;
  }

  .container {
    width: 90%;
    margin: 0 auto;
  }

  header {
    background: #e4ebe1;
  }
  header::after {
    content: "";
    display: table;
    clear: both;
  }

  .logo {
    float: left;
    padding: 10px 0;
  }

  nav {
    justify-content: space-between;
  }

  nav ul {
    margin: 0;
    padding: 0;
    list-style: none;
  }

  nav li {
    display: inline-block;
    margin-left: 2%;
    margin-right: 7%;
    padding-top: 2%;

    position: relative;
  }

  nav a {
    color: #444;
    text-decoration: none;
    text-transform: uppercase;
    font-size: 14px;
  }

  nav a:hover {
    color: #000;
  }

  nav a::before {
    content: "";
    display: block;
    height: 5px;
    background-color: #444;

    position: absolute;
    top: 0;
    width: 0%;

    transition: all ease-in-out 250ms;
  }

  nav a:hover::before {
    width: 100%;
  }
</style>
