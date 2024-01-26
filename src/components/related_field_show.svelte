<script>
</script>

<div class="container">
  <div class="column">
    <slot />
  </div>
  <div class="column">
    <button>Delete</button>
  </div>
</div>

<style>
  .block {
    display: block;
  }
  .container {
    display: flex;
    justify-content: space-evenly;
    clear: both;
  }
  .container button {
    margin-top: 20%;
    margin-left: 250%;

    padding: 8px 20px;
    border: none;
    border-radius: 4px;
    background-color: #007bff;
    color: #fff;
    font-size: 16px;
    cursor: pointer;
  }
</style>
