<script>
  export addfield = "";

  let temp = "";

  function handleAdd() {
    addfield = temp;
  }
</script>

<div class="container">
  <div class="column">
    <input type="text" bind:value={temp} />
  </div>

  <div class="column">
    <button on:click={handleAdd}>Add</button>
  </div>
</div>

<style>
  .block {
    display: block;
  }
  .column input {
    padding: 8px;
    border: 1px solid #ccc;
    border-radius: 4px;
    font-size: 16px;
    width: 110%;
    box-sizing: border-box;
  }
  .container {
    margin-left: 1%;
    margin-top: 40%;
    display: flex;
    clear: both;
  }
  .container button {
    margin-top: -5%;
    margin-left: 110%;

    padding: 8px 20px;
    border: none;
    border-radius: 4px;
    background-color: #007bff;
    color: #fff;
    font-size: 16px;
    cursor: pointer;
  }
</style>
