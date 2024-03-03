<script>
  import NavbarChair from "/src/components/navbar_chair.svelte";
  import NavbarUser from "/src/components/navbar_user.svelte";
  import { page } from "$app/stores";
  import "/src/app.css";
  import { onMount } from "svelte";

  let user_id, user;

  let unreadCount = null;

  const features = [
    {
      title: 'Create Conference',
      description: 'Easily create and manage your own conferences.'
    },
    {
      title: 'Submit paper, poster on conference',
      description: 'Submit your research paper, poster for review on a conference.'
    },
    {
      title: 'Review Papers',
      description: 'Review papers submitted by others.'
    },
  ];

  const features2 = [
    {
      title: 'Assign reviewers',
      description: 'Assign reviewers to review papers while maintaining constrains.'
    },
    {
      title: 'Create workshop on a conference',
      description: 'Create workshop and assign instructors on them easily.'
    },
    {
      title: 'Manage deadline and important dates',
      description: 'Manage important dates and deadlines automatically.'
    }
  ]

  onMount(async () => {
    try {
      const tokenResponse = await fetch(
        `${import.meta.env.VITE_BACKEND}/auth`,
        {
          headers: {
            "Content-Type": "application/json",
          },
          credentials: "include",
        }
      );

      let tokenData = await tokenResponse.json();
      user_id = tokenData.user_id;

      let url = `${import.meta.env.VITE_BACKEND}/user/${user_id}`;

      const response = await fetch(url);

      if (!response.ok) {
        throw new Error("Failed to fetch data");
      }

      user = await response.json();

      // let unreadCount = null;

      const unreadNotificationCount = await fetch(
        `${import.meta.env.VITE_BACKEND}/notification/unreadCount/${user_id}`
      );
      unreadCount = await unreadNotificationCount.json();
      unreadCount = unreadCount.unreadCount;

      sessionStorage.setItem("user_type", user.user_type);
    } catch (error) {
      console.error("Error fetching data:", error);
    }
  });
</script>

{#if user != null && unreadCount != null}
  <main>
    {#if user.user_type == "chair"}
      <NavbarChair myVariable={unreadCount} />
    {:else}
      <NavbarUser myVariable={unreadCount} />
    {/if}
    
  
    <h1 style="margin-top: 50px;">Welcome to Conference Management System</h1>

    <h3 style="margin-top: 40px;">Experience seamless conference management like never before! </h3>
      <h3>Our system simplifies the process, saving you time and effort. </h3>
      <h3>Whether you're creating a conference, submitting a paper, or reviewing submissions, we've got you covered.</h3>
      <h3>Enjoy a hassle-free experience with our conference management system.</h3>
        

    <br>

    <h2 style="margin-top: 50px; align: center"> Our Features: </h2>

    <br>
    <br>
    <section class="features">
      
      {#each features as feature}
        <div class="feature">
          <h3>{feature.title}</h3>
          <p>{feature.description}</p>
        </div>
      {/each}
    </section>

    <br>
    <br>

    <section class="features">
      
      {#each features2 as feature}
        <div class="feature">
          <h3>{feature.title}</h3>
          <p>{feature.description}</p>
        </div>
      {/each}
    </section>


  </main>
{/if}

<style>
  main {
    max-width: 90%;
    margin: 2% auto;
  }

  

  h1 {
  }

  .features {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
    gap: 20px;
  }

  .feature {
    border: 1px solid #535151;
    background-color: #beb3b3;
    padding: 20px;
    border-radius: 5px;
    min-height: 150px;
  }
</style>
