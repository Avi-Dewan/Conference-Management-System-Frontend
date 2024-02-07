<script>
  import { page } from "$app/stores";
  import { onMount } from "svelte";
  import NavbarChair from "/src/components/navbar_chair.svelte";
  import NavbarUser from "/src/components/navbar_user.svelte";

  const conference_id = $page.params.conference_id;
  let user_id;

  user_id = $page.params.user_id;
  let url = `http://localhost:3000/conference/${conference_id}/viewUnassaignedPapers`;
  let conference_url = `http://localhost:3000/conference/${conference_id}`;
  let papers = null;
  let conf_data = null;

  onMount(async () => {
    try {
      const response = await fetch(url);
      if (!response.ok) {
        throw new Error("Failed to fetch data");
      }

      papers = await response.json();

      console.log(papers);
    } catch (error) {
      console.error("Error fetching data:", error);
    }
  });

  onMount(async () => {
    try {
      const response = await fetch(conference_url);
      if (!response.ok) {
        throw new Error("Failed to fetch data");
      }

      conf_data = await response.json();
    } catch (error) {
      console.error("Error fetching data:", error);
    }
  });
</script>

<main>
  {#if papers != null && conf_data != null}
    <NavbarChair />
    <div>
      <h1>{conf_data.conference_title}</h1>
      {#each papers as item}
        <hr />
        <div style="margin-top: 70px;">
          <h2>Paper Title: {item.paper_title}</h2>
          <p><b>Author:</b> {item.authors}</p>
          <p><b>Track:</b> {item.related_fields}</p>
          <p><b>Abstract:</b> {item.abstract}</p>
          <p><b>Requested Reviewer:</b> {item.requestedReviewers}</p>
          <p><b> <u> Assigned Reviewers and reviews: </u></b></p>

          {#each item.reviews as rev}
            <div>
              <b>{rev.full_name}</b>
              {#if rev.rating != null}
                <p>rating: {rev.rating}</p>
                <p>review: {rev.review}</p>
              {:else}
                <p style="color:red">Not given review yet</p>
                <a href="#"> notify "{rev.full_name}"" to write review</a>
                <br />
                <br />
              {/if}
              <br />
            </div>
          {/each}
          <a href={item.pdf_link}>View file</a>
          <a href="/{user_id}/conference/assignReviewer/{item.paper_id}">
            Assign
          </a>
        </div>
      {/each}
    </div>
  {/if}
</main>

<style>
  main {
    max-width: 90%;
    margin: 2% auto;
  }

  h1 {
  }
</style>
