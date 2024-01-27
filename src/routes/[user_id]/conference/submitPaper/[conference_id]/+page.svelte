<script>
  import { page } from "$app/stores";

  const conference_id = $page.params.conference_id;

  import Navbar from "/src/components/navbar.svelte";

  import { createClient } from "@supabase/supabase-js";

  let user_id;

  $: user_id = $page.params.user_id;

  // Create Supabase client
  const supabase = createClient(
    "https://xrhpvghzaashivkemzfs.supabase.co",
    "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6InhyaHB2Z2h6YWFzaGl2a2VtemZzIiwicm9sZSI6InNlcnZpY2Vfcm9sZSIsImlhdCI6MTcwNjExNjk0NywiZXhwIjoyMDIxNjkyOTQ3fQ.StJztxPxeUe1LjwqLjuQTWzcc1sQ7g85LO0Pykd11T0"
  );

  let file_path = "";
  let files;
  // Upload file using standard upload
  async function uploadFile() {
    console.log(files[0]);
    alert(file_path);
    const { data, error } = await supabase.storage
      .from("test2")
      .upload("okok/haha.txt", files[0]);
    if (error) {
      // Handle error
    } else {
      // Handle success
    }
  }
  async function getPath() {
    alert(file_path);
    console.log(files[0]);
  }
  async function downloadFile() {
    const { data } = supabase.storage.from("test2").getPublicUrl("test/bodox");

    console.log(data);
  }
</script>

<form>
  <label for="file">File</label>
  <input id="file" type="file" bind:value={file_path} bind:files />
  <button on:click={uploadFile}>Upload</button>
</form>

<button on:click={downloadFile}></button>
