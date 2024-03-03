<script>
  import { createClient } from "@supabase/supabase-js";
  import NavbarChair from "/src/components/navbar_chair.svelte";
  import NavbarUser from "/src/components/navbar_user.svelte";

  import "/src/app.css";

  import { page } from "$app/stores";

  import { onMount } from "svelte";

  import { goto } from "$app/navigation";

  let user_id = $page.params.user_id,
    user_type;
  let conference_id = $page.params.conference_id;

  let chair_id;

  console.log(conference_id);

  $: filterText = "";

  let addfield = "";
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

  let co_authors_wihtout_account = [];

  let form_co_author_without_account = {
    first_name: "",
    last_name: "",
    email: "",
    affiliation: "",
  };

  function handleAdd_CoAuthor() {
    // console.log("hello");
    if (
      form_co_author_without_account.first_name != "" &&
      form_co_author_without_account.last_name != "" &&
      form_co_author_without_account.email != "" &&
      form_co_author_without_account.affiliation != ""
    ) {
      co_authors_wihtout_account = [
        ...co_authors_wihtout_account,
        form_co_author_without_account,
      ];

      form_co_author_without_account = {
        first_name: "",
        last_name: "",
        email: "",
        affiliation: "",
      };
    }
  }
  function removeCoAuthor(index) {
    co_authors_wihtout_account = co_authors_wihtout_account.filter(
      (_, i) => i != index
    );
  }

  async function submitPaper() {
    const req = await fetch(`${import.meta.env.VITE_BACKEND}/paper/submit`, {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify(formData),
    });

    let { co_authors_wihtout_account_id, paper_id } = await req.json();

    console.log("co authors without account id", co_authors_wihtout_account_id);
    console.log("paper id", paper_id);

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

    let fullnameurl = `${import.meta.env.VITE_BACKEND}/user/getFullName/${formData.main_author_id}`;

    const response_fullname = await fetch(fullnameurl);

    if (!response_fullname.ok) {
      throw new Error("Failed to fetch data");
    }

    let main_author_fullname = await response_fullname.json();

    let author_notification_body = `An user named ${main_author_fullname} has requested you to be a co-author on a paper titled ${formData.paper_title} on the conference ${conf_data.conference_title}`;

    let author_notification_json = {
      type: "notify_coauthor_paper",
      conference_id: conference_id,
      paper_id: paper_id,
    };

    // console.log(reviewer_id);

    for (let i = 0; i < formData.co_authors.length; i++) {
      console.log("printing co authors");
      console.log(formData.co_authors[i]);

      response = await fetch(
        `${import.meta.env.VITE_BACKEND}/notification/send`,
        {
          method: "POST",
          headers: {
            "Content-Type": "application/json",
          },
          body: JSON.stringify({
            user_id: formData.co_authors[i].user_id,
            notification_body: author_notification_body,
            notification_json: author_notification_json,
          }),
        }
      );

      data = await response.json();
    }

    for (let i = 0; i < co_authors_wihtout_account_id.length; i++) {
      console.log("printing co authors without account");
      console.log(co_authors_wihtout_account_id[i]);

      response = await fetch(
        `${import.meta.env.VITE_BACKEND}/notification/send`,
        {
          method: "POST",
          headers: {
            "Content-Type": "application/json",
          },
          body: JSON.stringify({
            user_id: co_authors_wihtout_account_id[i],
            notification_body: author_notification_body,
            notification_json: author_notification_json,
          }),
        }
      );

      data = await response.json();
    }
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
      console.log("whole data printing");
      console.log(wholeData);

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
    formData.co_authors = authors;

    formData.file = files[0];

    formData.main_author_id = user_id;
    formData.co_authors_without_account = co_authors_wihtout_account;

    formData.conference_id = conference_id;
    // console.log(conference_id);

    console.log(formData);

    uploadFile();

    submitPaper();

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

    {#if data != null && conf_data != null}
      <h1 class="mt-5 mb-5">Submit a Paper</h1>

      <div class="form">
        <div class="form-control">
          <h2 class="mb-4">Paper Title</h2>
          <input
            type="text"
            id="paper_title"
            bind:value={formData.paper_title}
          />
        </div>

        <div class="form-control">
          <h2 class="mb-5">Co-Authors</h2>

          {#if show == false}
            <div class="container">
              {#each authors as item, index (item)}
                <div class="column">
                  <div class="card w-96 bg-gray-200 mt-10">
                    <div class="card-body">
                      <h3>Name: {item.full_name}</h3>
                      <p>Affiliation: {item.current_institution}</p>
                      <p>Expertise: {item.expertise}</p>

                      <div class="card-actions justify-end mt-4">
                        <button
                          class="btn"
                          on:click={() => removeItem(index)}
                          style="background-color:red;"
                        >
                          Remove</button
                        >
                      </div>
                    </div>
                  </div>
                  <!-- <div>
                    <label style="width: 1px;"><h4>{item.full_name}</h4></label>
                  </div>
                  <div>
                    <button
                      class="btn"
                      on:click={() => removeItem(index)}
                      style="background-color:red;"
                    >
                      Remove</button
                    >
                  </div> -->
                </div>
              {/each}
            </div>
          {/if}

          {#if show == true}
            <div class="scrollable-window">
              <input
                type="text"
                placeholder="Filter by name"
                bind:value={filterText}
                style="max-width: 80%;margin-left:5%"
                on:input={() => {
                  data = wholeData.filter(
                    (item) =>
                      item.full_name
                        .toLowerCase()
                        .toLowerCase()
                        .indexOf(filterText.toLowerCase()) !== -1
                  );

                  if (filterText == "") {
                    data = wholeData;
                  }
                }}
              />
              {#each data as item (item.user_id)}
                <div class="card text-primary-content">
                  <div class="card-body">
                    <h3>Name: {item.full_name}</h3>
                    <h4>Affliation: {item.current_institution}</h4>
                    <h4>Expertise: {item.expertise}</h4>
                    <div class="card-actions justify-end mt-4">
                      <button
                        class="btn"
                        style="background-color:green;color:white"
                        on:click={handleAdd({
                          full_name: item.full_name,
                          current_institution: item.current_institution,
                          expertise: item.expertise,
                          user_id: item.user_id,
                        })}>Add as Co-author</button
                      >
                    </div>
                  </div>
                </div>
                <!-- <div>
                  <div class="card">
                    <h3>Name: {item.full_name}</h3>
                    <h4>Affliation: {item.current_institution}</h4>
                    <h4>Expertise: {item.expertise}</h4>
                    <button
                      on:click={handleAdd({
                        full_name: item.full_name,
                        user_id: item.user_id,
                      })}>Add</button
                    >
                  </div>
                </div> -->
              {/each}
            </div>
          {/if}
          <button
            class="mt-4 mb-3 btn"
            style="background-color: gray;color:white"
            on:click={authorSelect}>Choose Co-authors</button
          >
        </div>

        <div class="form-control">
          <h2 class="mb-5">
            Insert co-author info if they do not have account on the system
          </h2>
          <div>
            {#each co_authors_wihtout_account as item, index (item)}
              <div class="column">
                <div class="card w-96 bg-gray-200">
                  <div class="card-body">
                    <h4><b>Name: {item.first_name} {item.last_name} </b></h4>
                    <p>Email:{item.email}</p>
                    <p>Affliation:{item.affiliation}</p>

                    <div class="card-actions justify-end mt-4">
                      <button
                        class="btn"
                        on:click={() => removeCoAuthor(index)}
                        style="background-color:red;"
                      >
                        Remove</button
                      >
                    </div>
                  </div>
                </div>
                <!-- <div>
                    <label style="width: 1px;"><h4>{item.full_name}</h4></label>
                  </div>
                  <div>
                    <button
                      class="btn"
                      on:click={() => removeItem(index)}
                      style="background-color:red;"
                    >
                      Remove</button
                    >
                  </div> -->
              </div>
              <!-- <div class="two-column">
                <div>
                  <h4>{item.first_name} {item.last_name}</h4>
                  <h5>{item.email}</h5>
                  <h5>{item.affiliation}</h5>
                </div>
                <div>
                  <button
                    on:click={() => removeCoAuthor(index)}
                    style="background-color:red;"
                  >
                    Remove</button
                  >
                </div>
              </div> -->
            {/each}
          </div>
        </div>
        <br />
        <br />

        <div class="form-control">
          <div>
            <label for="first_name">First Name:</label>
            <input
              type="text"
              id="first_name"
              bind:value={form_co_author_without_account.first_name}
            />
          </div>

          <div>
            <label for="last_name"> Last Name:</label>
            <input
              type="text"
              id="last_name"
              bind:value={form_co_author_without_account.last_name}
            />
          </div>
        </div>

        <div class="form-control">
          <label for="email">Email:</label>
          <input
            type="text"
            id="email"
            bind:value={form_co_author_without_account.email}
          />
        </div>

        <div class="form-control">
          <label for="affiliation">Affiliation:</label>
          <input
            type="text"
            id="affiliation"
            bind:value={form_co_author_without_account.affiliation}
          />
        </div>

        <div class="btn btn-success">
          <button on:click={handleAdd_CoAuthor} style="height:40px;">
            Add co-author
          </button>
        </div>

        <div class="form-control">
          <h3 style="margin-top: 8%;">Research Track</h3>

          <div>
            <select bind:value={selectedTrack}>
              {#each conf_track as item}
                <option value={item}>{item}</option>
              {/each}
            </select>
          </div>
        </div>

        <div style="margin-top: 5%;">
          <h3>Abstract</h3>
          <textarea
            style="height: 150px; width:700px;border-radius:1em"
            bind:value={formData.abstract}
          ></textarea>
        </div>

        <form class="form-control">
          <h3 class="mt-4 mb-4">Attach Paper</h3>
          <input id="file" type="file" bind:value={file_path} bind:files />
        </form>
        <div>
          <button
            style="background-color: green;color:white"
            class="btn mt-4"
            on:click={handleSubmit}>Submit</button
          >
        </div>
      </div>
    {/if}
  {/if}
</main>

<style>
  /* .card button {
    position: relative;
    left: 40%;
    bottom: 10px;
  } */

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

  /* .form-control button {
    margin-top: 5%;
    padding: 8px 20px;
    border: none;
    border-radius: 4px;
    background-color: #007bff;
    color: #fff;
    font-size: 16px;
    cursor: pointer;
  } */

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
  /* .column button {
    margin-left: 18%;
    margin-top: 1%;
    background-color: green;
    height: 40px;
  } */

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
