<script>
  import moment from "moment";
  import Header from "./components/Header.svelte";
  import Body from "./components/Body.svelte";
  import Footer from "./components/Footer.svelte";

  let address = undefined;
  let transactions = undefined;
  let loading = false;
  let error = false;

  const fetchAndSortTxs = async event => {
    loading = true;
    error = false;
    // gets address
    address = event.detail;
    // creates "to" param
    const dateNow = new Date();
    const to = dateNow.toISOString();
    // creates "from" param
    const from = moment()
      .startOf("month")
      .toISOString();
    try {
      const response = await fetch(
        `https://staging.api.tzkt.io/v1/accounts/${address}/operations?from=${from}&to=${to}`
      );
      let txs = await response.json();
      if (txs.length > 0) {
        // if transactions were found
        // looks for incoming transactions
        txs = txs
          .filter(tx => tx.sender.address !== address)
          .map(tx => ({
            amount: tx.amount,
            hash: tx.hash,
            sender: tx.sender.address,
            timestamp: tx.timestamp
          }));
        transactions = [...txs];
      } else {
        transactions = [];
      }
    } catch (err) {
      console.log(err);
      error = true;
    } finally {
      loading = false;
    }
  };
</script>

<style>
  .buttons {
    justify-content: center !important;
  }
</style>

<main class="hero is-fullheight">
  <Header on:fetchTxs={fetchAndSortTxs} />
  {#if loading}
    <div class="hero-body">
      <div class="container has-text-centered">
        <div class="columns is-centered">
          <div class="column is-two-thirds">
            <div class="buttons">
              <button class="button is-white is-loading is-large" disabled />
              <button class="button is-white is-large" disabled>
                Loading...
              </button>
            </div>
          </div>
        </div>
      </div>
    </div>
  {:else if error}
    <div class="hero-body">
      <div class="columns is-centered">
        <div class="column is-two-thirds">
          <div class="message is-danger">
            <div class="message-body">
              <p>
                <strong>Error</strong>
              </p>
              <br />
              <p>
                An error occured when fetching the transactions for this
                address.
              </p>
              <p>Please try again later.</p>
            </div>
          </div>
        </div>
      </div>
    </div>
  {:else}
    <Body {transactions} {address} on:fetchTxs={fetchAndSortTxs} />
  {/if}
  <Footer />
</main>
