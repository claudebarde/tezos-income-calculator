<script>
  import { afterUpdate } from "svelte";
  import moment from "moment";
  import { createEventDispatcher } from "svelte";

  export let transactions, address;

  const dispatch = createEventDispatcher();
  let localAddress = "tz1SjrNeUE4zyPGSZpogDZd5tvryixNDsD2v";
  let bakerAddress = "";
  let checkStakingRewards = false;
  let totalIncome = 0;

  const validateAddress = event => {
    if (event.which == 13 || event.keyCode == 13) {
      // user pressed Enter
      dispatch("fetchTxs", localAddress);
    }
  };

  const formatHash = hash => hash.slice(0, 5) + "..." + hash.slice(-5);

  const checkBakerAddress = event => {
    const value = event.target.value;
    // check incomes coming from baker
    if (event.which == 13 || event.keyCode == 13) {
      checkStakingRewards = true;
      const filteredTxs = transactions.filter(tx => tx.sender === value);
      if (filteredTxs.length > 0) {
        totalIncome = filteredTxs.reduce((a, b) => ({
          amount: a.amount + b.amount
        }));
        // unfold returned object
        totalIncome = totalIncome.amount / 1000000;
        bakerAddress = value;
      } else {
        bakerAddress = "";
        checkStakingRewards = false;
      }
    }
  };

  afterUpdate(() => {
    if (transactions && transactions.length > 0 && !checkStakingRewards) {
      checkStakingRewards = false;
      totalIncome = transactions.reduce((a, b) => ({
        amount: a.amount + b.amount
      }));
      // unfold returned object
      totalIncome = totalIncome.amount / 1000000;
    }
  });
</script>

<style>
  .baker-address {
    max-width: 500px;
    margin: 0 auto;
    padding-bottom: 15px;
  }

  table td a,
  table td span {
    vertical-align: -webkit-baseline-middle;
  }
</style>

<div class="hero-body">
  <div class="container has-text-centered">
    <div class="columns is-centered">
      {#if transactions === undefined}
        <div class="column is-two-thirds">
          <div class="field">
            <label for="address-input">
              <strong class="is-size-4">Enter an address below:</strong>
            </label>
            <br />
            <br />
            <p class="control has-icons-left">
              <input
                id="address-input"
                class="input is-rounded is-large"
                type="text"
                placeholder="Enter an address on mainnet"
                bind:value={localAddress}
                on:keydown={validateAddress} />
              <span class="icon is-left">
                <i class="fa fa-search" />
              </span>
            </p>
          </div>
        </div>
      {:else if transactions.length === 0}
        <div class="column is-two-thirds">
          <article class="message is-info">
            <div class="message-body">
              <p>
                <strong>No transaction found</strong>
              </p>
              <br />
              <p>
                There are no transaction associated with this address for the
                current month.
              </p>
            </div>
          </article>
        </div>
      {:else}
        <div class="column is-two-thirds">
          <h1 class="title is-4 is-hidden-mobile">
            Total income {checkStakingRewards ? `from ${formatHash(bakerAddress)}` : ''}
            for this month: ꜩ {totalIncome}
          </h1>
          <h1 class="title is-4 is-hidden-desktop">
            Total income
            {@html checkStakingRewards ? `<br />from ${formatHash(bakerAddress)}` : ''}
            <br />
            for this month:
            <br />
            ꜩ {totalIncome}
          </h1>
          <div class="field baker-address">
            <p class="control has-icons-left">
              <input
                id="baker-address"
                class="input is-rounded is-small"
                type="text"
                placeholder={"Enter your baker's address to calculate staking rewards"}
                value={bakerAddress}
                on:keyup={checkBakerAddress} />
              <span class="icon is-left">
                <i class="fa fa-birthday-cake" />
              </span>
            </p>
          </div>
          <h3 class="subtitle">Address: {formatHash(address)}</h3>
          <div class="table-container">
            <!-- DESKTOP VIEW -->
            <table
              class="table is-striped is-narrow is-fullwidth is-hidden-mobile">
              <thead>
                <tr>
                  <td />
                  <td>Sender</td>
                  <td>Amount</td>
                  <td>Transaction Hash</td>
                  <td>Date</td>
                </tr>
              </thead>
              <tbody>
                {#if checkStakingRewards}
                  {#each transactions.filter(tx => tx.sender === bakerAddress) as tx}
                    <tr>
                      <td>
                        <img
                          src={`https://services.tzkt.io/v1/avatars/${tx.sender}`}
                          alt="tzkitty"
                          class="image is-32x32" />
                      </td>
                      <td>
                        <a
                          href={`https://tzkt.io/${tx.sender}`}
                          target="_blank"
                          rel="noreferrer noopener">
                          {formatHash(tx.sender)}
                        </a>
                      </td>
                      <td>
                        <span>ꜩ {tx.amount / 1000000}</span>
                      </td>
                      <td>
                        <a
                          href={`https://tzkt.io/${tx.hash}`}
                          target="_blank"
                          rel="noreferrer noopener">
                          {formatHash(tx.hash)}
                        </a>
                      </td>
                      <td>
                        <span>
                          {moment(tx.timestamp).format('ddd, MMM Do YYYY, h:mm a')}
                        </span>
                      </td>
                    </tr>
                  {/each}
                {:else}
                  {#each transactions as tx}
                    <tr class="is-hidden-mobile">
                      <td>
                        <img
                          src={`https://services.tzkt.io/v1/avatars/${tx.sender}`}
                          alt="tzkitty"
                          class="image is-32x32" />
                      </td>
                      <td>
                        <a
                          href={`https://tzkt.io/${tx.sender}`}
                          target="_blank"
                          rel="noreferrer noopener">
                          {formatHash(tx.sender)}
                        </a>
                      </td>
                      <td>
                        <span>ꜩ {tx.amount / 1000000}</span>
                      </td>
                      <td>
                        <a
                          href={`https://tzkt.io/${tx.hash}`}
                          target="_blank"
                          rel="noreferrer noopener">
                          {formatHash(tx.hash)}
                        </a>
                      </td>
                      <td>
                        <span>
                          {moment(tx.timestamp).format('ddd, MMM Do YYYY, h:mm a')}
                        </span>
                      </td>
                    </tr>
                  {/each}
                {/if}
              </tbody>
            </table>
            <!-- MOBILE VIEW -->
            <div class="table-container">
              <table class="table is-narrow is-striped is-fullwidth">
                <tbody>
                  {#if checkStakingRewards}
                    {#each transactions.filter(tx => tx.sender === bakerAddress) as tx}
                      <tr class="is-hidden-desktop">
                        <td class="has-text-centered">
                          <img
                            src={`https://services.tzkt.io/v1/avatars/${tx.sender}`}
                            alt="tzkitty"
                            class="image is-32x32"
                            style="display:inline-block;vertical-align:middle;" />
                          &nbsp;
                          <a
                            href={`https://tzkt.io/${tx.sender}`}
                            target="_blank"
                            rel="noreferrer noopener">
                            {formatHash(tx.sender)}
                          </a>
                          &nbsp;
                          <span>ꜩ {tx.amount / 1000000}</span>
                        </td>
                      </tr>
                      <tr class="is-hidden-desktop">
                        <td class="has-text-centered">
                          <span>
                            {moment(tx.timestamp).format('ddd, MMM Do YYYY, h:mm a')}
                          </span>
                        </td>
                      </tr>
                    {/each}
                  {:else}
                    {#each transactions as tx}
                      <tr class="is-hidden-desktop">
                        <td class="has-text-centered">
                          <img
                            src={`https://services.tzkt.io/v1/avatars/${tx.sender}`}
                            alt="tzkitty"
                            class="image is-32x32"
                            style="display:inline-block;vertical-align:middle;" />
                          &nbsp;
                          <a
                            href={`https://tzkt.io/${tx.sender}`}
                            target="_blank"
                            rel="noreferrer noopener">
                            {formatHash(tx.sender)}
                          </a>
                          &nbsp;
                          <span>ꜩ {tx.amount / 1000000}</span>
                        </td>
                      </tr>
                      <tr class="is-hidden-desktop">
                        <td class="has-text-centered">
                          <span>
                            {moment(tx.timestamp).format('ddd, MMM Do YYYY, h:mm a')}
                          </span>
                        </td>
                      </tr>
                    {/each}
                  {/if}
                </tbody>
              </table>
            </div>
          </div>
        </div>
      {/if}
    </div>
  </div>
</div>
