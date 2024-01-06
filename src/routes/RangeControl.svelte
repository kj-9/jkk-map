<script>
  import RangeSlider from 'svelte-range-slider-pips';
  import ReturnIcon from './ReturnIcon.svelte';

  export let rent;
  export let year;

  const titleRent = '家賃';
  const titleYear = '築年数';

  const conf = {
    range: true,
    pushy: true,
    float: true,
    springValues: { stiffness: 1, damping: 1 }
  };
</script>

<div class="container">
  <div class="return-icon"><a href="/projects"><ReturnIcon /></a></div>
  <div><h2>{titleRent}</h2></div>
  <div class="slider">
    <RangeSlider
      bind:values={rent}
      formatter={(v) => v / 10_000}
      suffix="万円"
      min={30000}
      max={200000}
      step={5000}
      {...conf}
    />
  </div>
  <div><h2>{titleYear}</h2></div>
  <div class="slider">
    <RangeSlider bind:values={year} suffix="年" min={0} max={65} step={5} {...conf} />
  </div>
</div>

<style>
  h2 {
    font-size: 1rem;
    font-weight: 300;
  }
  .container {
    display: grid;
    grid-template-columns: 1fr 1fr 2.5fr;
    align-items: center;
    min-width: 250px;
    background-color: #e7e6e6b7;
    border-radius: 20px;
    padding: 1rem;
    --range-handle: #159a9c;
    --range-handle-focus: #159a9c;
    --range-handle-inactive: #159a9c; /* show handle even if inactive */
  }

  .return-icon {
    grid-row-start: 1;
    grid-column-start: 1;
    grid-row-end: 3;
    grid-column-end: 2;
  }

  /* setting for RangeSlider from svelte-range-slider-pips */
  :global(.slider .rangeBar) {
    /* set slider background */
    background: linear-gradient(270deg, #019597 -1.35%, #ddefef 106.73%);
  }

  :global(.slider .rangeSlider .rangeHandle) {
    /* smaller handle */
    height: 1em;
    width: 1em;
  }

  :global(.slider .rangeSlider .rangeNub) {
    border: solid 1px white; /* set white border for handle */
    top: -1px; /* offset to 1px border to center handle */
  }

  :global(.slider .rangeSlider :is(.rangeHandle, .rangeHandle.active) .rangeFloat) {
    font-size: 0.8em;
    opacity: 1; /* always show float */
    transform: translate(0%, -100%); /* except first, Float is right aligned */
  }

  :global(.slider .rangeSlider .rangeHandle:nth-child(1) .rangeFloat) {
    transform: translate(-100%, -100%); /* first Float is left aligned */
  }
</style>
