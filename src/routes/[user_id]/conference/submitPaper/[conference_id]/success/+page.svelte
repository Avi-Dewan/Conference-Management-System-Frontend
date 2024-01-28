<script>
</script>

<main>
  <div>
    <button>Show Success</button>

    <div class="success-dialog">
      <p>Success! Your action was completed.</p>
    </div>
  </div>
</main>

<style>
  .success-dialog {
    position: fixed;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    background-color: #4caf50;
    color: white;
    padding: 20px;
    border-radius: 10px;
    box-shadow: 0 4px 8px 0 rgba(0, 0, 0, 0.2);
    z-index: 9999;
  }
</style>
