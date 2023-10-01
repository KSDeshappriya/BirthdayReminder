<script lang="ts">
  import { onMount, onDestroy } from "svelte";
  import { tick } from "svelte";
  import Papa from "papaparse";

  let csvDataU: any[] = [];
  let upcomingBirthdays: any[] = [];
  let countArr: any[] = [];
  let countdown: string = "";
  let interval: NodeJS.Timeout;

  onMount(async () => {
    const response = await fetch("data.csv");
    const text = await response.text();
    const result = Papa.parse(text, { header: true }).data;
    // console.log(result)

    const today = new Date();
    upcomingBirthdays = result.filter((row: any) => {
      const birthdate = new Date(
        `${row.Month} ${row.Day}, ${today.getFullYear()}`
      );
      return birthdate >= today;
    });
    // console.log(upcomingBirthdays)

    csvDataU = result.map((row: any) => ({
      Name: row.Name,
      Date: `${row.Month} ${row.Day}, ${row.Year}`,
    }));

    // Initialize the countdown
    updateCountdown();
    // Update the countdown every second
    interval = setInterval(updateCountdown, 1000);
  });

  onDestroy(() => {
    // Cleanup the interval when the component is unmounted
    clearInterval(interval);
  });

  async function updateCountdown() {
    const currentDate: any = new Date();
    const firstBirthdayDate: any = new Date(
      `${upcomingBirthdays[0].Month} ${upcomingBirthdays[0].Day}`
    );
    firstBirthdayDate.setFullYear(currentDate.getFullYear());

    const timeRemaining = firstBirthdayDate - currentDate;
    const days = Math.floor(timeRemaining / (1000 * 60 * 60 * 24));
    const hours = Math.floor(
      (timeRemaining % (1000 * 60 * 60 * 24)) / (1000 * 60 * 60)
    );
    const minutes = Math.floor(
      (timeRemaining % (1000 * 60 * 60)) / (1000 * 60)
    );
    const seconds = Math.floor((timeRemaining % (1000 * 60)) / 1000);

    countdown = `${days} days ${hours} hours ${minutes} minutes ${seconds} seconds`;
    countArr = [`${days}`, `${hours}`, `${minutes}`, `${seconds}`];

    // Use tick to trigger a Svelte update
    await tick();
  }
</script>

<main>
  <!-- countdown -->
  {#if upcomingBirthdays.length > 0}
    <div class="countdown">
      <h1 id="headline">Countdown to next birthday</h1>
      <p>
        {upcomingBirthdays[0].Name}'s birthday on {upcomingBirthdays[0].Month}
        {upcomingBirthdays[0].Day}
      </p>
      <div id="countdown">
        <ul>
          <li><span id="days">{countArr[0]}</span>days</li>
          <li><span id="hours">{countArr[1]}</span>Hours</li>
          <li><span id="minutes">{countArr[2]}</span>Minutes</li>
          <li><span id="seconds">{countArr[3]}</span>Seconds</li>
        </ul>
      </div>
    </div>
  {:else}
    <p>No upcoming birthdays found.</p>
  {/if}

  <!-- reminder -->
  {#if upcomingBirthdays.length > 0}
    <h1>Birthday Reminders</h1>
    <ul>
      {#each upcomingBirthdays as row, index}
        <li data-id={index}>
          {row.Name}'s birthday is on {row.Month}
          {row.Day}
        </li>
      {/each}
    </ul>
  {/if}

  <!-- data table -->
  {#if csvDataU.length > 0}
    <h1>CSV Data</h1>
    <table>
      <thead>
        <tr>
          <th>Name</th>
          <th>Date</th>
        </tr>
      </thead>
      <tbody>
        {#each csvDataU as row, rowIndex}
          <tr data-id={rowIndex}>
            <td>{row.Name}</td>
            <td>{row.Date}</td>
          </tr>
        {/each}
      </tbody>
    </table>
  {:else}
    <p>No CSV data available.</p>
  {/if}
</main>

<!-- <script lang="ts">
  import { onMount } from "svelte";
  import Papa from "papaparse";

  let csvData: any[] = [];
  let csvDataU: any[] = [];
  let upcomingBirthdays: any[] = [];

  onMount(async () => {
    // Load and parse the CSV data
    const response = await fetch("data.csv"); // Replace with the path to your CSV file
    const text = await response.text();
    Papa.parse(text, {
      header: true, // Set this to true if your CSV has headers
      complete: (result) => {
        csvData = result.data;
        csvDataU = csvData.map((row) => ({
          Name: row.Name,
          Date: `${row.Month} ${row.Day}, ${row.Year}`,
        }));
        // Calculate upcoming birthdays
        const today = new Date();
        upcomingBirthdays = csvData.filter((row) => {
          const birthdate = new Date(`${row.Month} ${row.Day}, ${row.Year}`);
          birthdate.setFullYear(today.getFullYear()); // Set the year to the current year
          return birthdate >= today;
        });
      },
    });
  });
</script>

<main>
  <h1>CSV Data</h1>
  <table>
    {#if csvDataU.length > 0}
      <thead>
        <tr>
          <th>Name</th>
          <th>Date</th>
        </tr>
      </thead>
      <tbody>
        {#each csvDataU as row, rowIndex}
          <tr key={rowIndex}>
            <td>{row.Name}</td>
            <td>{row.Date}</td>
          </tr>
        {/each}
      </tbody>
    {:else}
      <p>No CSV data available.</p>
    {/if}
  </table>

  <h1>Birthday Reminders</h1>
  <ul>
    {#each upcomingBirthdays as raw, index}
      <li key={index}>
        {raw.Name}'s birthday is on {raw.Month} {raw.Day}
      </li>
    {/each}
  </ul>
</main>

<style>
  /* Add your CSS styles here */
  table {
    border-collapse: collapse;
    width: 50%;
  }
  th, td {
    border: 1px solid #ddd;
    padding: 8px;
    text-align: left;
  }
  th {
    background-color: #f2f2f2;
  }
  tr:nth-child(even) {
    background-color: #f2f2f2;
  }
</style> -->

<!-- <script lang="ts">
  import { onMount } from "svelte";
  import Papa from "papaparse";

  let csvData: any[] = [];

  onMount(async () => {
    // Load and parse the CSV data
    const response = await fetch("data.csv"); // Replace with the path to your CSV file
    const text = await response.text();
    Papa.parse(text, {
      header: true, // Set this to true if your CSV has headers
      complete: (result) => {
        csvData = result.data;
      },
    });
  });
</script>

<main>
  <h1>CSV Data</h1>
  <table>
    {#if csvData.length > 0}
      <thead>
        <tr>
          {#each Object.keys(csvData[0]) as header}
            <th>{header}</th>
          {/each}
        </tr>
      </thead>
      <tbody>
        {#each csvData as row, rowIndex}
          <tr key={rowIndex}>
            {#each Object.values(row) as value, columnIndex}
              <td key={columnIndex}>{value}</td>
            {/each}
          </tr>
        {/each}
      </tbody>
    {:else}
      <p>No CSV data available.</p>
    {/if}
  </table>
</main>

<style>
  /* Add your CSS styles here */
  table {
    border-collapse: collapse;
    width: 100%;
  }
  th,
  td {
    border: 1px solid #ddd;
    padding: 8px;
    text-align: left;
  }
  th {
    background-color: #f2f2f2;
  }
  tr:nth-child(even) {
    background-color: #f2f2f2;
  }
</style> -->

<style>
  /* Your CSS styles here */
  table {
    border-collapse: collapse;
    width: 50%;
  }
  th,
  td {
    border: 1px solid #ddd;
    padding: 8px;
    text-align: left;
  }
  th {
    background-color: #f2f2f2;
  }
  tr:nth-child(even) {
    background-color: #f2f2f2;
  }

  /* Countdown styles */

  .countdown {
    color: #333;
    margin: 0 auto;
    text-align: center;
  }

  .countdown h1 {
    font-weight: normal;
    letter-spacing: 0.125rem;
    text-transform: uppercase;
  }

  .countdown li {
    display: inline-block;
    font-size: 1.5em;
    list-style-type: none;
    padding: 1em;
    text-transform: uppercase;
  }

  .countdown li span {
    display: block;
    font-size: 4.5rem;
  }

  .countdown .emoji span {
    font-size: 4rem;
    padding: 0 0.5rem;
  }

  @media all and (max-width: 768px) {
    .countdown h1 {
      font-size: calc(1.5rem * var(--smaller));
    }

    .countdown li {
      font-size: calc(1.125rem * var(--smaller));
    }

    .countdown li span {
      font-size: calc(3.375rem * var(--smaller));
    }
  }
</style>
