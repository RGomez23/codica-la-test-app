<script>
  import { onMount } from "svelte";
  const flags = {
    EUR: "eu",
    USD: "us",
    JPY: "jp",
  };

  let originCurrencyInput = 0;
  let destinationCurrencyOutput = 0;
  let currencies = ["EUR", "USD", "JPY"];
  let selectedOriginCurrency = "EUR";
  let originFlag = "eu";
  let selectedDestinationCurrency = "USD";
  let destinationFlag = "us";
  let isConvertionButtonDisabled = true;
  let conversionHistory = [];
  let rates;

  onMount(async () => {
    await setRates();
  });

  async function setRates() {
    const res = await fetch(
      `https://api.freecurrencyapi.com/v1/latest?apikey=&base_currency=${selectedOriginCurrency}&currencies=EUR,USD,JPY`
    );
    let { data } = await res.json();
    rates = data;
  }

  function beforeConvertionValidation() {
    let canInputBeConverted = /^\d+(\.\d{1,2})?$/.test(originCurrencyInput);

    if (canInputBeConverted) {
      isConvertionButtonDisabled = false;
    }

    if (!canInputBeConverted) {
      isConvertionButtonDisabled = true;
    }
  }

  function setOriginCurrencyFlag() {
    originFlag = flags[selectedOriginCurrency];
  }

  function setDestinationCurrencyFlag() {
    destinationFlag = flags[selectedDestinationCurrency];
  }

  function formatCurrency(currency, amount) {
    return new Intl.NumberFormat("en-US", {
      style: "currency",
      currency: currency,
    }).format(amount);
  }

  function getFormatedDate() {
    const date = new Date();
    return date.toLocaleDateString();
  }

  function executeCurrencyConvertion() {
    destinationCurrencyOutput =
      originCurrencyInput * rates[selectedDestinationCurrency];

    conversionHistory.push({
      originCurrency: selectedOriginCurrency,
      originAmount: formatCurrency(selectedOriginCurrency, originCurrencyInput),
      destinationCurrency: selectedDestinationCurrency,
      destinationAmount: formatCurrency(
        selectedDestinationCurrency,
        destinationCurrencyOutput
      ),
      date: getFormatedDate(),
    });

    conversionHistory = conversionHistory;
  }

  $: selectedOriginCurrency, setOriginCurrencyFlag();
  $: selectedOriginCurrency, setRates();
  $: selectedDestinationCurrency, setDestinationCurrencyFlag();
</script>

<main class="flex flex-col h-dvh px-4">
  <div class="card max-w-md bg-base-100 shadow-xl mx-auto my-auto">
    <div class="card-body grid grid-cols-3">
      <div class="grid col-span-3 content-center">
        {#if rates}
          <p>
            1 {selectedOriginCurrency} es igual a {rates[
              selectedDestinationCurrency
            ]}
            {selectedDestinationCurrency}
          </p>
        {/if}
      </div>
      <div class="grid col-span-2 content-center">
        <p><span class="fi fi-{originFlag}"></span> {selectedOriginCurrency}</p>
      </div>
      <div class="col-span-1">
        <select
          bind:value={selectedOriginCurrency}
          class="select select-ghost w-full max-w-xs"
        >
          {#each currencies as currency}
            <option {currency}>{currency}</option>
          {/each}
        </select>
      </div>
      <div class="col-span-3">
        <input
          type="text"
          class="input input-bordered input-info w-full max-w-xs rounded-full"
          bind:value={originCurrencyInput}
          on:keyup={beforeConvertionValidation}
        />
      </div>
      <div class="col-span-3">
        <button
          class="btn btn-primary w-full rounded-full my-4"
          disabled={isConvertionButtonDisabled}
          on:click={executeCurrencyConvertion}>Convertir</button
        >
      </div>
      <div class="grid col-span-2 content-center">
        <p>
          <span class="fi fi-{destinationFlag}"></span>
          {selectedDestinationCurrency}
        </p>
      </div>
      <div class="col-span-1">
        <select
          class="select select-ghost w-full max-w-xs"
          bind:value={selectedDestinationCurrency}
        >
          {#each currencies as currency}
            <option>{currency}</option>
          {/each}
        </select>
      </div>
      <div class="col-span-3">
        <input
          type="text"
          class="input input-bordered input-info w-full max-w-xs rounded-full"
          bind:value={destinationCurrencyOutput}
        />
      </div>
    </div>
  </div>
  <div class="card max-w-md bg-base-100 shadow-xl mx-auto my-auto">
    <div class="card-body grid grid-cols-3">
      <div class="grid col-span-3">
        <div class="overflow-x-auto">
          <table class="table">
            <!-- head -->
            <thead>
              <tr>
                <th></th>
                <th>Divisa de origen</th>
                <th>Monto de origen</th>
                <th>Divisa de destino</th>
                <th>Monto de destino</th>
                <th>Fecha de conversión</th>
              </tr>
            </thead>
            <tbody>
              {#each conversionHistory as item, i}
                <tr>
                  <th>{i + 1}</th>
                  <td>{item.originCurrency}</td>
                  <td>{item.originAmount}</td>
                  <td>{item.destinationCurrency}</td>
                  <td>{item.destinationAmount}</td>
                  <td>{item.date}</td>
                </tr>
              {/each}
            </tbody>
          </table>
        </div>
      </div>
    </div>
  </div>
</main>
