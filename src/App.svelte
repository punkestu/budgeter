<script>
  // @ts-nocheck

  let list = [];
  let count = 1;
  let meta = "";
  let dumpData = "";

  const add = () => {
    list = [
      ...list,
      {
        id: list[list.length - 1]?.id + 1 || 1,
        name: ``,
        qty: 1,
        harga: null,
      },
    ];
  };

  const remove = (id) => {
    list = list.filter((item) => item.id !== id);
  };

  const clear = () => {
    list = [];
    count = 1;
  };

  const generateMeta = (list) => {
    meta =
      `- Total: Rp. ${total}\n- Orang: ${count}\n- Per Orang: Rp. ${count > 0 ? total / count : 0}\n\n` +
      list.reduce((acc, item) => {
        return item.qty && item.harga
          ? `${acc}- ${item.name || "no name"} : ${item.qty} x ${item.harga} = ${item.qty * item.harga}\n`
          : acc;
      }, "");
  };

  const copyToClipboard = (text) => {
    navigator.clipboard.writeText(text.trim());
    alert("Copied to clipboard");
  };

  const dump = () => {
    if (!dumpData) {
      alert("Please fill the data");
      return;
    }
    const data = dumpData.split("\n\n");
    const metas = data[0]
      .split("\n")
      .map((meta) => {
        const parts = meta.split(":");
        if (parts[0].trim() === "- Orang") {
          return ["count", parseInt(parts[1].trim())];
        }
      })
      .reduce((acc, item) => {
        if (item) {
          acc[item[0]] = item[1];
        }
        return acc;
      }, {});
    if (metas.count) {
      count = metas.count;
    }

    if (!data[1]) {
      return;
    }
    const items = data[1].split("\n").map((item, i) => {
      if (!item.match(/- \w+ \| \d+ x \d+ = \d+/g)) {
        return null;
      }
      const [name, part2] = item.split("|");
      return {
        id: i,
        name: name.trim().split("-")[1].trim(),
        qty: parseInt(part2.split("x")[0].trim()),
        harga: parseInt(part2.split("x")[1].split("=")[1].trim()),
      };
    });
    list = items.filter((item) => item);
  };

  $: total = list.reduce((acc, item) => {
    return item.qty && item.harga ? acc + item.qty * item.harga : acc;
  }, 0);
  $: generateMeta(list);
</script>

<textarea class="w-0 h-0 absolute" bind:value={meta}></textarea>
<main class="bg-slate-900 min-h-screen flex flex-col items-center gap-2 p-4">
  <h1 class="text-center text-white font-bold text-2xl">Budgeter</h1>
  <section class="w-full md:w-2/3 bg-white px-4 py-2 rounded-lg flex flex-col gap-2">
    <h2><span class="font-semibold text-lg">Total: </span>Rp. {total}</h2>
    <div class="w-full">
      <label for="count" class="font-semibold text-lg">Jumlah Orang: </label>
      <input
        type="number"
        id="count"
        bind:value={count}
        class="px-3 py-1 rounded-md border-2 border-slate-400 w-full md:w-auto"
        placeholder="Jumlah Orang"
      />
    </div>
    <h2>
      <span class="font-semibold text-lg">Per Orang: </span>Rp. {count > 0
        ? total / count
        : 0}
    </h2>
    <button
      class="bg-blue-600 text-white px-4 py-1 rounded-lg font-semibold w-full"
      on:click={() => copyToClipboard(meta)}>Save to clipboard</button
    >
  </section>
  <section class="flex flex-col gap-3 w-full md:w-2/3">
    {#each list as item (item.id)}
      <div class="grid grid-cols-12 items-center gap-1">
        <input
          class="px-3 py-1 rounded-md col-span-12 md:col-span-4"
          type="text"
          bind:value={item.name}
          placeholder="Nama Item ..."
        />
        <span class="hidden md:block text-center text-white col-span-1">|</span>
        <input
          class="px-3 py-1 rounded-md col-span-2"
          type="number"
          bind:value={item.qty}
          placeholder="Kuantitas ..."
        />
        <span class="text-center text-white col-span-1">x</span>
        <input
          class="px-3 py-1 rounded-md col-span-7 md:col-span-3"
          type="number"
          bind:value={item.harga}
          placeholder="Harga ..."
        />
        <button class="px-3 py-1 col-span-1" on:click={() => remove(item.id)}
          >❌️</button
        >
      </div>
    {/each}
    <div class="grid grid-cols-2 md:grid-cols-6 gap-2">
      <button
        class="bg-blue-600 text-white px-4 py-1 rounded-lg font-semibold col-span-2 md:col-span-4"
        on:click={add}>Tambah +</button
      >
      <button
        class="bg-blue-600 text-white px-4 py-1 rounded-lg font-semibold"
        popovertarget="dumper">Dump</button
      >
      <button
        class="bg-red-600 text-white px-4 py-1 rounded-lg font-semibold"
        on:click={clear}>Reset</button
      >
    </div>
  </section>
</main>

<div id="dumper" class="w-2/3 rounded-lg border-2 border-slate-400" popover="">
  <textarea
    bind:value={dumpData}
    class="resize-none w-full h-[30vh] md:h-[50vh] rounded-lg p-4"
    placeholder={"- Total: Rp. 60000\n- Orang: 1\n- Per Orang: Rp. 60000"}
  ></textarea>
  <button
    on:click={dump}
    class="bg-blue-600 text-white px-4 py-1 rounded-lg font-semibold w-full"
    popovertarget="dumper"
    popovertargetaction="hide">Dump</button
  >
</div>
