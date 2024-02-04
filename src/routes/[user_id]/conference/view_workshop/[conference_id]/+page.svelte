<script>
  import { page } from "$app/stores";
  import { DatePicker } from "@svelte-plugins/datepicker";

  const conference_id = $page.params.conference_id;

  import NavbarChair from "/src/components/navbar_chair.svelte";
  import NavbarUser from "/src/components/navbar_user.svelte";

  import { onMount } from "svelte";

  let user_id, user_type;

  user_id = $page.params.user_id;
</script>

<main>
  <h1>{conference_id}</h1>
</main>

<style>
</style>
