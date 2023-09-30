<script lang="ts">
  import { onMount, onDestroy } from "svelte";
  import { tick } from "svelte";
  import Papa from "papaparse";

  let csvDataU: any[] = [];
  let upcomingBirthdays: any[] = [];
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
    const currentDate = new Date();
    const firstBirthdayDate = new Date(`${upcomingBirthdays[0].Month} ${upcomingBirthdays[0].Day}`);
    firstBirthdayDate.setFullYear(currentDate.getFullYear());

    const timeRemaining = firstBirthdayDate - currentDate;
    const days = Math.floor(timeRemaining / (1000 * 60 * 60 * 24));
    const hours = Math.floor((timeRemaining % (1000 * 60 * 60 * 24)) / (1000 * 60 * 60));
    const minutes = Math.floor((timeRemaining % (1000 * 60 * 60)) / (1000 * 60));
    const seconds = Math.floor((timeRemaining % (1000 * 60)) / 1000);

    countdown = `${days} days ${hours} hours ${minutes} minutes ${seconds} seconds`;

    // Use tick to trigger a Svelte update
    await tick();
  }
</script>

<main>
  {#if upcomingBirthdays.length > 0}
    <h1>Countdown to Next Birthday</h1>
    <p>Next birthday: {upcomingBirthdays[0].Name}'s birthday on {upcomingBirthdays[0].Month} {upcomingBirthdays[0].Day}</p>
    <p>Time remaining: {countdown}</p>
  {:else}
    <p>No upcoming birthdays found.</p>
  {/if}

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
</style>
