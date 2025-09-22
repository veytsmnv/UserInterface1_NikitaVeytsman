<script>
  import { onMount } from "svelte";
  import Chart from "chart.js/auto";


  let { labels = [], balances = [] } = $props();

  let canvas;
  let chart;

  onMount(() => {
    chart = new Chart(canvas, {
      type: "bar",
      data: {
        labels: labels,
        datasets: [
          {
            label: "Amount Won/Lost",
            data: balances,
            backgroundColor: balances.map(v => v >= 0 ? "rgba(34,197,94,0.7)" : "rgba(239,68,68,0.7)"),
          }
        ]
      },
      options: {
        responsive: true,
        maintainAspectRatio: false,
        plugins: {
          legend: { display: false }
        },
        scales: {
          y: { beginAtZero: true }
        }
      }
    });
  });

  $effect(() => {
    if (chart) {
      chart.data.labels = labels;
      chart.data.datasets[0].data = balances;
      chart.update();
    }
  });
</script>

<style>
  .chart-box {
    width: 100%;
    height: 300px;
    background: #111;
    border-radius: 6px;
    margin-top: 1rem;
  }

  canvas {
    width: 100% !important;
    height: 100% !important;
  }
</style>

<div class="chart-box">
  <canvas bind:this={canvas}></canvas>
</div>
