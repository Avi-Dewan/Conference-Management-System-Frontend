<script>
  import { createClient } from "@supabase/supabase-js";
  import NavbarChair from "/src/components/navbar_chair.svelte";
  import NavbarUser from "/src/components/navbar_user.svelte";

  import { page } from "$app/stores";

  import { onMount } from "svelte";

  import { goto } from "$app/navigation";
  import "/src/app.css";

  let user_id = $page.params.user_id,
    user_type;
  let conference_id = $page.params.conference_id;

  let paper_id = $page.params.paper_id;
  let chair_id;

  console.log(conference_id);

  $: filterText = "";

  let addfield = "";

  let paper_detail = null;

  let formData = {
    paper_id: "",
    paper_title: "",
    abstract: "",
    pdf_link: "",
    related_fields: null,
    file: null,
    co_authors: null,
  };

  let url = `${import.meta.env.VITE_BACKEND}/user/all`;
  let conf_url = `${import.meta.env.VITE_BACKEND}/conference/${conference_id}`;

  let data = null;

  let conf_data = null;

  let conf_track = [];

  let wholeData = null;

  let file_path;
  let files;

  let authors = [];

  let researchFields = [];

  let paper_abstract = "";

  let selectedTrack = "";

  let show = false;

  let submitted = false;

  onMount(async () => {
    try {
      paper_detail = await fetch(
        `${import.meta.env.VITE_BACKEND}/paper/${paper_id}`
      );

      let paper_author = await fetch(
        `${import.meta.env.VITE_BACKEND}/paper/${paper_id}/author`
      );

      paper_author = await paper_author.json();
      authors = paper_author;

      paper_detail = await paper_detail.json();

      paper_detail = paper_detail[0];

      formData.paper_title = paper_detail.paper_title;
      formData.abstract = paper_detail.abstract;
      selectedTrack = paper_detail.related_fields;

      console.log(paper_detail);
    } catch (error) {
      console.error("Error fetching data:", error);
    }
  });

  async function editPaper() {
    const req = await fetch(
      `${import.meta.env.VITE_BACKEND}/paper/edit_paper/${paper_id}`,
      {
        method: "PUT",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify(formData),
      }
    );

    //let data = await req.json();

    const response_chair = await fetch(
      `${import.meta.env.VITE_BACKEND}/conference/conference_chair/${conference_id}`
    );

    if (!response_chair.ok) {
      throw new Error("Failed to fetch data");
    }

    chair_id = await response_chair.json();

    chair_id = chair_id.user_id;

    console.log("chair id hocche...");

    console.log(chair_id);

    let notification_body = `A paper titled ${formData.paper_title} has been submitted in the conference ${conf_data.conference_title}`;

    let notification_json = {
      type: "notify_chair_paper",
      conference_id: conference_id,
    };

    // console.log(reviewer_id);

    let response = await fetch(
      `${import.meta.env.VITE_BACKEND}/notification/send`,
      {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify({
          user_id: chair_id,
          notification_body: notification_body,
          notification_json: notification_json,
        }),
      }
    );

    data = await response.json();

    // let fullnameurl = `${import.meta.env.VITE_BACKEND}/user/getFullName/${formData.main_author_id}`;

    // const response_fullname = await fetch(fullnameurl);

    // if (!response_fullname.ok) {
    //   throw new Error("Failed to fetch data");
    // }

    // let main_author_fullname = await response_fullname.json();

    // let author_notification_body = `An user named ${main_author_fullname} has requested you to be a co-author on a paper titled ${formData.paper_title} on the conference ${conf_data.conference_title}`;

    // let author_notification_json = {
    //   type: "notify_coauthor_paper",
    //   conference_id: conference_id,
    //   paper_id: paper_id,
    // };

    // console.log(reviewer_id);

    // for (let i = 0; i < formData.co_authors.length; i++) {
    //   console.log("printing co authors");
    //   console.log(formData.co_authors[i]);

    //   response = await fetch(`${import.meta.env.VITE_BACKEND}/notification/send", {
    //     method: "POST",
    //     headers: {
    //       "Content-Type": "application/json",
    //     },
    //     body: JSON.stringify({
    //       user_id: formData.co_authors[i].user_id,
    //       notification_body: author_notification_body,
    //       notification_json: author_notification_json,
    //     }),
    //   });

    //   data = await response.json();
    // }
  }

  onMount(async () => {
    try {
      user_type = sessionStorage.getItem("user_type");

      const response = await fetch(url);
      if (!response.ok) {
        throw new Error("Failed to fetch data");
      }

      data = await response.json();
      wholeData = data;

      for (let i = 0; i < wholeData.length; i++) {
        if (wholeData[i]["user_id"] === user_id) {
          wholeData.splice(i, 1);
          i--; // decrement i to adjust for the removed element
        }
      }
    } catch (error) {
      console.error("Error fetching data:", error);
    }
  });

  onMount(async () => {
    try {
      const response = await fetch(conf_url);
      if (!response.ok) {
        throw new Error("Failed to fetch data");
      }

      conf_data = await response.json();
      conf_track = conf_data.related_fields;
    } catch (error) {
      console.error("Error fetching data:", error);
    }
  });

  function authorSelect() {
    if (data != null) {
      show = !show;
    }
  }

  function handleAdd(name) {
    // console.log("hello");

    authors = [...authors, name];
    // console.log("eta handle add er vitore")
    // console.log(data)
    // console.log("data sesh")
    // console.log(authors)
    // console.log("author sesh")

    data = data.filter((item1) => {
      return !authors.some((item2) => item2.user_id === item1.user_id);
    });
    show = false;
  }

  function removeItem(index) {
    authors = authors.filter((_, i) => i != index);
  }

  function handleAddReserach() {
    // console.log("hello");
    if (addfield != "") {
      researchFields = [...researchFields, String(addfield)];
      addfield = "";
    }
  }
  function removeItemResearch(index) {
    researchFields = researchFields.filter((_, i) => i != index);
  }

  // Create Supabase client
  const supabase = createClient(
    "https://xrhpvghzaashivkemzfs.supabase.co",
    "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6InhyaHB2Z2h6YWFzaGl2a2VtemZzIiwicm9sZSI6InNlcnZpY2Vfcm9sZSIsImlhdCI6MTcwNjExNjk0NywiZXhwIjoyMDIxNjkyOTQ3fQ.StJztxPxeUe1LjwqLjuQTWzcc1sQ7g85LO0Pykd11T0"
  );

  // Upload file using standard upload
  async function uploadFile() {
    file_path = file_path.split(".").pop();

    console.log(file_path);

    file_path = conference_id + "/" + Date.now() + "." + file_path;

    formData.pdf_link = supabase.storage
      .from("submission")
      .getPublicUrl(file_path).data.publicUrl;

    const { data, error } = await supabase.storage
      .from("submission")
      .upload(file_path, files[0]);
    if (error) {
      // Handle error
    } else {
      // Handle success
    }
  }

  function handleSubmit() {
    formData.related_fields = selectedTrack;

    formData.file = files[0];

    formData.main_author_id = user_id;

    formData.conference_id = conference_id;
    console.log(conference_id);

    uploadFile();

    editPaper();

    submitted = true;

    goto(`/${user_id}/conference/submitPaper/${conference_id}/success`);

    // alert(JSON.stringify(formData, null, 2));
  }
  let unreadCount = null;
  onMount(async () => {
    try {
      const unreadNotificationCount = await fetch(
        `${import.meta.env.VITE_BACKEND}/notification/unreadCount/${user_id}`
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
    <NavbarUser />

    {#if data != null && conf_data != null && paper_detail != null}
      <h1 class="mt-5 mb-5">Submit a Paper</h1>

      <div class="form">
        <div class="form-control">
          <h2>Paper Title</h2>
          <input
            class="mt-5"
            type="text"
            id="paper_title"
            bind:value={formData.paper_title}
          />
        </div>

        <div class="mt-5">
          <h2>Paper Authors:</h2>
          {#each authors as item}
            <div class="mt-5 ml-1">
              <b>{item.full_name}</b>
            </div>
          {/each}
        </div>

        <div class="form-control">
          <h3 style="margin-top: 2%;">Research Track</h3>

          <div>
            <select class="mt-5" bind:value={selectedTrack}>
              {#each conf_track as item}
                <option value={item}>{item}</option>
              {/each}
            </select>
          </div>
        </div>

        <div style="margin-top: 5%;">
          <h3>Abstract</h3>
          <textarea
            class="mt-4"
            style="height: 150px; width:700px;border-radius:1em"
            bind:value={formData.abstract}
          ></textarea>
        </div>

        <form class="form-control">
          <h3 class="mt-5">Attach Paper</h3>
          <input
            class="mt-5"
            id="file"
            type="file"
            bind:value={file_path}
            bind:files
          />
        </form>
        <div class="form-control" style="display: block;">
          <button on:click={handleSubmit}>Submit</button>
        </div>
      </div>
    {/if}
  {/if}
</main>

<style>
  .card button {
    position: relative;
    left: 40%;
    bottom: 10px;
  }
  .two-column {
    margin-top: -1%;
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
  .card {
    margin-left: 5%;
    margin-top: 2%;
    border: 1px solid grey;
    max-width: 80%;
    text-align: center;
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
    margin-bottom: 10px;
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

  .scrollable-window {
    height: 300px; /* Adjust height as needed */
    overflow-y: auto;
    border: 1px solid #ccc;
    padding: 10px;
  }
</style>
