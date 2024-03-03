<script>
  import NavbarChair from "/src/components/navbar_chair.svelte";
  import NavbarUser from "/src/components/navbar_user.svelte";
  import { page } from "$app/stores";
  import "/src/app.css";
  import { onMount } from "svelte";

  let user_id, user;

  let unreadCount = null;

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

    <button class="btn btn-accent">Accent</button>
    <h1
      class="mb-4 text-4xl font-extrabold leading-none tracking-tight text-gray-900 md:text-5xl lg:text-6xl dark:text-white"
    >
      Regain <mark class="px-2 text-white bg-blue-600 rounded dark:bg-blue-500"
        >control</mark
      > over your days
    </h1>
    <h1>Hello world</h1>
    <h1>Welcome to Conference Management System</h1>
    <p>
      Lorem ipsum dolor, sit amet consectetur adipisicing elit. Unde ipsum,
      numquam aspernatur earum dolore, libero pariatur voluptatibus itaque qui
      illo alias, tempore ullam? Perferendis mollitia obcaecati pariatur
      molestias exercitationem. Nam?
    </p>
  </main>
{/if}

<style>
  main {
    max-width: 90%;
    margin: 2% auto;
  }

  h1 {
  }
</style>
