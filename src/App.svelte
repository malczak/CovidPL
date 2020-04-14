<script>
  import { onMount } from "svelte";
  import Map from "./Map.svelte";
  import createEmptyStates from "./states";

  let promise = null;
  let days = [];
  let index = 0;
  let stats = { min: 0, max: 0, sick: 0, dead: 0 };
  $: {
    mapDay(days[index]);
  }

  const MinDate = new Date(2020, 2, 12);
  const NowDate = new Date();
  const DaysDiff = Math.ceil(
    (NowDate.getTime() - MinDate.getTime()) / 86400000
  );
  const Dates = new Array(DaysDiff).fill(0).map((_, index) => {
    let date = new Date(MinDate);
    date.setDate(MinDate.getDate() + index);
    return date;
  });

  const Palette = [
    "#ffffff",
    "#fff7dd",
    "#ffedc6",
    "#ffe3b1",
    "#ffd99e",
    "#ffce8a",
    "#ffc477",
    "#ffb964",
    "#ffad51",
    "#ffa13c",
    "#fc9629",
    "#f48e28",
    "#ec8626",
    "#e47e25",
    "#dc7523",
    "#d56d21",
    "#ce6420",
    "#c85a1d",
    "#c3501b",
    "#d4200e"
  ];

  const migrateData = data => {
    if (!data.records.some(item => item.hasOwnProperty("Liczba"))) {
      return data;
    }

    let records = createEmptyStates();
    data.records.forEach(item => {
      const name = item["Województwo"].toLowerCase();
      const sick = parseInt(item["Liczba"]) || 0;
      const dead = parseInt(item["Liczba zgonów"]) || 0;
      const state = records.find(itemAt => itemAt.state === name);
      if (state) {
        state.count += sick;
        state.dead += dead;
      }
    });

    return { date: data.date, records };
  };

  const formatDate = date =>
    [
      date.getFullYear(),
      `00${date.getMonth() + 1}`.substr(-2),
      `00${date.getDate()}`.substr(-2)
    ].join("");

  const urlForDate = date =>
    `${DATA_ENDPOINT}`.replace("%DATE%", formatDate(date));

  const mapDay = data => {
    [
      ...document.querySelectorAll(".area > path"),
      ...document.querySelectorAll(".area > polygon")
    ].forEach(node => {
      node.style.fill = Palette[0];
    });

    if (!data) {
      return;
    }

    data = migrateData(data);

    const records = data.records;
    stats = records.reduce(
      (accum, item) => {
        const v = item.count;
        item.value = v;
        accum.sick += v;
        accum.dead += item.dead;
        accum.max = Math.max(accum.max, v);
        accum.min = Math.min(accum.min, v);
        return accum;
      },
      {
        sick: 0,
        dead: 0,
        max: Number.NEGATIVE_INFINITY,
        min: Number.POSITIVE_INFINITY
      }
    );

    const offset = 1;
    records.forEach(item => {
      const v = item.count;
      const unified = (v - stats.min) / (stats.max - stats.min);
      const scaledValue =
        offset + Math.round(unified * (Palette.length - offset));
      const color = Palette[Math.min(scaledValue, Palette.length - 1)];
      const stateEl = document.querySelector(`#${item.id}`);
      if (stateEl) {
        const counterEl = stateEl.querySelector("text tspan");
        if (counterEl) {
          counterEl.innerHTML = v;
        }
        let pathEl = stateEl.children[0];
        pathEl.style.fill = color;
      }
    });
  };

  onMount(() => {
    promise = Promise.all([
      ...Dates.map(date =>
        fetch(urlForDate(date))
          .then(data => data.json())
          .then(records => ({ date, records }))
      )
    ]).then(result => {
      days = days.concat(result);
      setInterval(() => {
        index = (index + 1) % days.length;
      }, 1000);
    });
  });
</script>

<style>
  main {
    text-align: center;
    padding: 1em;
    margin: 0 auto;
  }

  .stats {
    position: absolute;
    top: 10px;
    left: 10px;
    min-width: 150px;
    padding: 2px;
    font-size: 1.2em;
    text-align: left;
  }

  .stats > div {
    margin-bottom: 4px;
  }

  .stats .s1,
  .stats .s2 {
    margin: 4px;
    font-weight: bold;
  }

  .stats .s1 {
    color: green;
  }

  .stats .s2 {
    color: #ff3e00;
  }

  .stats .pal {
    font-size: 0.7em;
  }

  .stats .pal .min {
    float: left;
  }

  .stats .pal .max {
    float: right;
  }
</style>

<main>

  {#if promise}
    {#await promise}
      <span>Loading virus 🦠 🦠 🦠</span>
    {:then _}
      <div class="stats">
        <div>
          {new Intl.DateTimeFormat('pl-PL', {
            month: 'numeric',
            day: 'numeric',
            year: 'numeric'
          }).format(days[index].date)}
        </div>
        <div>
          Confirmed:
          <span class="s1">{stats.sick}</span>
        </div>
        <div>
          Deaths:
          <span class="s2">{stats.dead}</span>
        </div>
        <div class="pal">
          <svg
            viewBox="0 0 {Palette.length * 10} 20"
            preserveAspectRatio="none">
            {#each Palette as color, index}
              <rect x={index * 10} y="0" width="10" height="20" fill={color} />
            {/each}
          </svg>
          <span class="min">0</span>
          <span class="max">{stats.max}</span>
        </div>
      </div>

    {:catch _}
      <span>Ups</span>
    {/await}
  {/if}

  <Map />

</main>
