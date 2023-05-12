<script>
  import { onMount } from "svelte";
  export let data;

  let l = data[""].label[""];
  // query
  let q = data[""].q;

  const price = (number) => {
    number = Number(number);
    if (number === 0 || isNaN(number)) {
      return "";
    }
    return `${q.currency} ${number.toLocaleString(undefined, {
      minimumFractionDigits: 2,
    })}`;
  };
  const qty = (number) => {
    number = Number(number);
    if (number === 0 || isNaN(number)) {
      return "";
    }
    return number.toLocaleString(undefined, {
      minimumFractionDigits: 0,
    });
  };
  const rate = (rate) => {
    rate = Number(rate) * 100;
    if (!Number.isInteger(rate)) {
      rate = rate.toFixed(2);
    }
    return `${rate} %`;
  };
  const addItem = () => {
    q.itemDesc.push("");
    q.itemPrice.push("");
    q.itemQty.push("");
    q = q;
  };
  const removeItem = () => {
    q.itemDesc.pop();
    q.itemPrice.pop();
    q.itemQty.pop();
    q = q;
  };

  onMount(() => {
    const s = new URLSearchParams(location.search);
    let obj = q;
    Object.keys(q).forEach((key) => {
      const values = s.getAll(key);
      if (values.length > 0) {
        if (Array.isArray(q[key])) {
          obj[key] = values;
          return;
        }
        obj[key] = values[0];
      }
    });
    q = { ...data[q.lang].q, ...obj };
  });

  $: l = {
    ...data[q.lang].label[""],
    ...data[q.lang].label[q.doc],
  };
  $: q.itemAmount = q.itemPrice.map((pr, i) => {
    const num = Number(pr) * Number(q.itemQty[i]);
    return num ? num : "";
  });
  $: q.totalAmount = q.itemAmount.reduce((a, b) => {
    const num = Number(a) + Number(b);
    return num ? num : "";
  }, 0);
  $: q.totalVat = Number(q.totalAmount) * Number(q.vatRate);
  $: q.totalWht = Number(q.totalAmount) * Number(q.whtRate);
  $: q.totalFinal =
    Number(q.totalAmount) +
    Number(q.totalVat) +
    Number(q.totalWht) +
    Number(q.totalAdjust);
</script>

<div class="flex flex-wrap justify-center items-center my-4 print:hidden">
  {#each Object.keys(data) as lng, i (`lang-${i}`)}
    <button
      class="block font-bold duration-300 p-2 {q.lang === lng
        ? 'bg-blue-500 text-gray-100'
        : 'text-gray-900 bg-gray-100 hover:bg-blue-500 focus:bg-blue-500 hover:text-gray-100 focus:text-gray-100'}"
      on:click={() => {
        q.lang = lng;
      }}
    >
      {data[lng]['']}
    </button>
  {/each}
  {#each Object.keys(data[q.lang].label) as dc, i (`doc-${i}`)}
    <button
      class="block font-bold duration-300 p-2 {q.doc === dc
        ? 'bg-blue-500 text-gray-100'
        : 'text-gray-900 bg-gray-100 hover:bg-blue-500 focus:bg-blue-500 hover:text-gray-100 focus:text-gray-100'}"
      on:click={() => {
        q.doc = dc;
      }}
    >
      {data[q.lang].label[dc].title}
    </button>
  {/each}
</div>

<div
  class="font-sans bg-white text-black p-4 max-w-[60rem] mx-auto print:max-w-none print:mx-0"
>
  <div class="grid grid-cols-2 gap-4 mb-3">
    <div class="my-auto">
      <div class="" on:click={() => {}}>
        <img class="mb-3" src={q.vendorLogo} alt="" width="" height="" />
      </div>
      <h3
        class="text-xl mb-1"
        contenteditable="true"
        bind:textContent={q.vendorName}
      />
      <p
        class="ml-3 mb-1"
        contenteditable="true"
        bind:textContent={q.vendorId}
      />
      <p
        class="ml-3 mb-6"
        contenteditable="true"
        bind:textContent={q.vendorAddress}
      />
      <h4 class="mb-1">{l.client}</h4>
      <h3
        class="text-xl mb-1"
        contenteditable="true"
        bind:textContent={q.clientName}
      />
      <p
        class="ml-3 mb-1"
        contenteditable="true"
        bind:textContent={q.clientId}
      />
      <p
        class="ml-3 mb-3"
        contenteditable="true"
        bind:textContent={q.clientAddress}
      />
      <h4 class="">
        <span class="font-bold">{l.subject}</span>
        <span class="" />
        <span
          class="text-lg text-blue-500"
          contenteditable="true"
          bind:textContent={q.subject}
        />
      </h4>
    </div>
    <div class="my-auto">
      <h1 class="text-4xl text-blue-500 mb-3">{l.title}</h1>
      <p class="ml-3 mb-1" contenteditable="true" bind:textContent={q.ref} />
      <p class="ml-3 mb-3" contenteditable="true" bind:textContent={q.date} />
      <div class="bg-gray-50 px-3 py-6 mb-3">
        {#if q.doc !== "receipt"}
          <h4 class="mb-1">{l.duedate}</h4>
          <p
            class="px-2 py-0.5 border bg-white mb-1"
            contenteditable="true"
            bind:textContent={q.duedate}
          />
        {/if}
        <h4 class="mb-1">{l.paymethod}</h4>
        <p
          class="px-2 py-0.5 border bg-white"
          contenteditable="true"
          bind:textContent={q.paymethod}
        />
      </div>
      <h4 class="text-right">
        <span class="font-bold">{l.totalFinal}</span>
        <span class="" />
        <span class="text-lg text-blue-500">
          {price(q.totalFinal)}
        </span>
      </h4>
    </div>
  </div>
  <table class="w-full">
    <thead class="bg-blue-500 text-white">
      <tr class="">
        <th class="p-2 w-px whitespace-nowrap">{l.itemNo}</th>
        <th class="">
          <div class="flex">
            <p class="p-2 flex-grow">{l.itemDesc}</p>
            <button
              class="p-2 focus:bg-blue-700 hover:bg-blue-700 transition duration-300 ease-in-out print:hidden"
              on:click={addItem}
            >
              <!-- heroicons solid duplicate -->
              <svg
                xmlns="http://www.w3.org/2000/svg"
                class="h-6 w-6"
                viewBox="0 0 20 20"
                fill="currentColor"
              >
                <path
                  d="M7 9a2 2 0 012-2h6a2 2 0 012 2v6a2 2 0 01-2 2H9a2 2 0 01-2-2V9z"
                />
                <path d="M5 3a2 2 0 00-2 2v6a2 2 0 002 2V5h8a2 2 0 00-2-2H5z" />
              </svg>
            </button>
            <button
              class="p-2 focus:bg-blue-700 hover:bg-blue-700 transition duration-300 ease-in-out print:hidden"
              on:click={removeItem}
            >
              <!-- heroicons solid trash 	-->
              <svg
                xmlns="http://www.w3.org/2000/svg"
                class="h-6 w-6"
                viewBox="0 0 20 20"
                fill="currentColor"
              >
                <path
                  fill-rule="evenodd"
                  d="M9 2a1 1 0 00-.894.553L7.382 4H4a1 1 0 000 2v10a2 2 0 002 2h8a2 2 0 002-2V6a1 1 0 100-2h-3.382l-.724-1.447A1 1 0 0011 2H9zM7 8a1 1 0 012 0v6a1 1 0 11-2 0V8zm5-1a1 1 0 00-1 1v6a1 1 0 102 0V8a1 1 0 00-1-1z"
                  clip-rule="evenodd"
                />
              </svg>
            </button>
          </div>
        </th>
        <th class="p-2 w-px whitespace-nowrap text-right">{l.itemPrice}</th>
        <th class="p-2 w-px whitespace-nowrap text-right">{l.itemQty}</th>
        <th class="p-2 w-px whitespace-nowrap text-right">{l.itemAmount}</th>
      </tr>
    </thead>
    <tbody class="">
      {#each q.itemDesc as _, i (`item-${i}`)}
        <tr class="border-b">
          <td class="py-1 px-2 whitespace-nowrap" contenteditable="true"
            >{i + 1}</td
          >
          <td
            class="py-1 px-2"
            contenteditable="true"
            bind:textContent={q.itemDesc[i]}
          />
          <td
            class="py-1 px-2 text-right whitespace-nowrap"
            contenteditable="true"
            on:focus={(e) => (e.target.textContent = q.itemPrice[i])}
            on:input={(e) => (q.itemPrice[i] = e.target.textContent)}
            on:blur={(e) => (e.target.textContent = price(q.itemPrice[i]))}
          >
            {price(q.itemPrice[i])}
          </td>
          <td
            class="py-1 px-2 text-right whitespace-nowrap"
            contenteditable="true"
            on:focus={(e) => (e.target.textContent = q.itemQty[i])}
            on:input={(e) => (q.itemQty[i] = e.target.textContent)}
            on:blur={(e) => (e.target.textContent = qty(q.itemQty[i]))}
          >
            {qty(q.itemQty[i])}
          </td>
          <td class="py-1 px-2 text-right whitespace-nowrap">
            {price(q.itemAmount[i])}
          </td>
        </tr>
      {/each}
    </tbody>
    <tfoot class="">
      <tr class="">
        <td class="pr-3" colspan="2" rowspan={q.doc === "receipt" ? 4 : 3}>
          <div class="bg-gray-50 p-3 mb-3">
            <h4 class="mb-1 mt-3">{l.clientSign}</h4>
            <p class="px-2 py-0.5 border bg-white" contenteditable="true">
              <br /><br />
            </p>
            <p
              class="px-2 py-0.5 mb-3 border-b border-r border-l bg-white"
              contenteditable="true"
            />
          </div>
        </td>
        <td class="py-1 px-2 border-b whitespace-nowrap" colspan="2"
          >{l.totalAmount}</td
        >
        <td class="py-1 px-2 border-b text-right whitespace-nowrap">
          {price(q.totalAmount)}
        </td>
      </tr>
      <tr class="">
        <td class="py-1 px-2 border-b whitespace-nowrap" colspan="2">
          <span class="">{l.totalVat}</span>
          <span class="" />
          <span
            class=""
            contenteditable="true"
            on:focus={(e) => (e.target.textContent = q.vatRate)}
            on:input={(e) => (q.vatRate = e.target.textContent)}
            on:blur={(e) => (e.target.textContent = rate(q.vatRate))}
          >
            {rate(q.vatRate)}
          </span>
        </td>
        <td class="py-1 px-2 border-b text-right whitespace-nowrap">
          {price(q.totalVat)}
        </td>
      </tr>
      {#if q.doc === "receipt"}
        <tr class="">
          <td class="py-1 px-2 border-b whitespace-nowrap" colspan="2">
            <span class="">{l.totalWht}</span>
            <span class="" />
            <span
              class=""
              contenteditable="true"
              on:focus={(e) => (e.target.textContent = q.whtRate)}
              on:input={(e) => (q.whtRate = e.target.textContent)}
              on:blur={(e) => (e.target.textContent = rate(q.whtRate))}
            >
              {rate(q.whtRate)}
            </span>
          </td>
          <td class="py-1 px-2 border-b text-right whitespace-nowrap">
            {price(q.totalWht)}
          </td>
        </tr>
      {/if}
      <tr class="">
        <td class="py-1 px-2 border-b whitespace-nowrap" colspan="2"
          >{l.totalAdjust}</td
        >
        <td
          class="py-1 px-2 border-b text-right whitespace-nowrap"
          contenteditable="true"
          on:focus={(e) => (e.target.textContent = q.totalAdjust)}
          on:input={(e) => (q.totalAdjust = e.target.textContent)}
          on:blur={(e) => (e.target.textContent = price(q.totalAdjust))}
        >
          {price(q.totalAdjust)}
        </td>
      </tr>
    </tfoot>
  </table>
  <div class="grid grid-cols-2 gap-4">
    <div class="">
      <h4 class="font-bold mb-1">{l.note}</h4>
      <p class="ml-3 mb-3" contenteditable="true" bind:textContent={q.note} />
      <h3 class="text-xl font-bold text-blue-500">{l.thankMessage}</h3>
    </div>
    <div class="">
      <h4 class="mb-1 mt-3">{l.vendorSign}</h4>
      <p class="px-2 py-0.5 border" contenteditable="true"><br /><br /></p>
      <p
        class="px-2 py-0.5 mb-3 border-b border-r border-l"
        contenteditable="true"
      />
    </div>
  </div>
</div>

<div class="flex flex-wrap justify-center items-center my-4 print:hidden">
  <button
    class="block font-bold duration-300 p-2 text-gray-100 bg-blue-500 hover:bg-gray-100 focus:bg-gray-100 hover:text-gray-900 focus:text-gray-900"
    on:click={() => window.print()}
  >
    Print
  </button>
</div>
