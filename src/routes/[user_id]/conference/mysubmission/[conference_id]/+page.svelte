<script>
  import { page } from "$app/stores";
  import { DatePicker } from "@svelte-plugins/datepicker";

  const conference_id = $page.params.conference_id;

  import NavbarChair from "/src/components/navbar_chair.svelte";
  import NavbarUser from "/src/components/navbar_user.svelte";

  import { onMount } from "svelte";

  import { goto } from "$app/navigation";

  let user_id, user_type;

  user_id = $page.params.user_id;

  let conf_data = null;

  let confurl = `http://localhost:3000/conference/${conference_id}`;
  let myPaperInConferenceURL = `http://localhost:3000/paper/mysubmission/${conference_id}/${user_id}`;

  const currentDate = new Date();

  let paper_data = null;
  let review_data = null;
  onMount(async () => {
    try {
      user_type = sessionStorage.getItem("user_type");

      console.log(user_type);

      const response = await fetch(confurl);
      if (!response.ok) {
        throw new Error("Failed to fetch data");
      }

      conf_data = await response.json();
    } catch (error) {
      console.error("Error fetching data:", error);
    }
  });

  onMount(async () => {
    try {
      user_type = sessionStorage.getItem("user_type");

      console.log(user_type);

      const response = await fetch(myPaperInConferenceURL);
      if (!response.ok) {
        throw new Error("Failed to fetch data");
      }

      paper_data = await response.json();

      console.log(paper_data);

      // let myPaperReviewURL = `http://localhost:3000/paper/review/${conference_id}/${user_id}`;
      // const review_response = await fetch(myPaperReviewURL);

      // if(!review_response.ok){
      //   throw new Error("failed to fetch data");
      // }

      // review_data = await review_response.json();
      // console.log(review_data);
    } catch (error) {
      console.error("Error fetching data:", error);
    }
  });

  async function handleDeleteSubmission(paper_id) {
    const thisPage = window.location.pathname;
    goto(`/${user_id}/home`).then(() => goto(thisPage));
    let response = await fetch(
      "http://localhost:3000/paper/delete_submission",
      {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify({ paper_id: paper_id }),
      }
    );
  }

  let unreadCount = null;

  onMount(async () => {
    try {
      user_type = sessionStorage.getItem("user_type");

      const unreadNotificationCount = await fetch(
        `http://localhost:3000/notification/unreadCount/${user_id}`
      );

      unreadCount = await unreadNotificationCount.json();
      unreadCount = unreadCount.unreadCount;
    } catch (error) {
      console.error("Error fetching data:", error);
    }
  });
</script>

<main>
  {#if unreadCount != null}
    {#if user_type == "chair"}
      <NavbarChair myVariable={unreadCount} />
    {:else}
      <NavbarUser myVariable={unreadCount} />
    {/if}

    {#if conf_data != null && paper_data != null}
      <h2>Conference Title: {conf_data.conference_title}</h2>

      {#each paper_data as item}
        <div>
          <h3>Paper title: {item.paper_title}</h3>

          <p>
            <b>Authors: </b>
            {#each item.authors as author, index (author)}
              {author.full_name}{index < item.authors.length - 1 ? ", " : ""}
            {/each}
          </p>
          <p><b>Abstract:</b>{item.abstract}</p>
          <p></p>
          <p><b>Research Area:</b>{item.related_fields}</p>

          <button on:click={() => goto(item.pdf_link)}>View pdf</button>

          <h3>Status: {item.status}</h3>

          {#if item.status == "pending"}
            <div style="margin-top: 20px;">
              <button
                style="background-color: red;"
                on:click={() => {
                  handleDeleteSubmission(item.paper_id);
                }}>Delete submission</button
              >
            </div>
          {:else}
            {#each item.reviews as review, idx}
              <p>
                {#if review.review != null}
                  Reviewer {idx + 1} said: {review.review}
                {/if}
                <br />
                {#if review.rating != null}
                  Reviewer {idx + 1} rating: {review.rating}
                {/if}
              </p>
            {/each}
          {/if}
        </div>
      {/each}
    {/if}
  {/if}
</main>

<style>
  main {
    max-width: 90%;
    margin: 2% auto;
  }

  h1 {
  }

  button {
    margin-top: 0%;
    padding: 8px 20px;
    border: none;
    border-radius: 4px;
    background-color: #007bff;
    color: #fff;
    font-size: 16px;
    cursor: pointer;
  }
</style>
