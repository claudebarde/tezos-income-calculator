<script>
  import { createEventDispatcher } from "svelte";

  const dispatch = createEventDispatcher();
  let address = "";

  const validateAddress = event => {
    if (event.which == 13 || event.keyCode == 13) {
      // user pressed Enter
      dispatch("fetchTxs", address);
      address = "";
    }
  };
</script>

<style>
  .navbar-end {
    width: 450px !important;
  }

  .navbar-end .navbar-item,
  .navbar-end .field,
  .navbar-end input {
    width: 100%;
  }

  input {
    font-family: "Montserrat", sans-serif;
  }
</style>

<div class="hero-head has-background-primary">
  <header class="navbar is-spaced">
    <div class="container">
      <div class="navbar-brand">
        <p class="navbar-item has-text-weight-bold is-size-5 has-text-white">
          Tezos Income Calculator
        </p>
      </div>
      <div class="navbar-menu">
        <div class="navbar-end">
          <div class="navbar-item">
            <div class="field">
              <p class="control has-icons-left">
                <input
                  class="input is-rounded"
                  type="text"
                  placeholder="Enter an address on mainnet"
                  bind:value={address}
                  on:keydown={validateAddress} />
                <span class="icon is-left">
                  <i class="fa fa-search" />
                </span>
              </p>
            </div>
          </div>
        </div>
      </div>
    </div>
  </header>
</div>
