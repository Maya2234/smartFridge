<script>
  import { onMount } from 'svelte'
  import svelteLogo from './assets/svelte.svg'
  import viteLogo from './assets/vite.svg'
  import heroImg from './assets/hero.png'
  import Counter from './lib/Counter.svelte'

  let currentTime = $state(new Date())

  onMount(() => {
    const clock = setInterval(() => {
      currentTime = new Date()
    }, 1000)

    return () => clearInterval(clock)
  })

  let sliderValue = $state(4);
  let showTemperature = $state(false);
	let min = 1;
	let max = 6;
	let step = .5;

  import Timer from './timer.svelte';
  import Keypad from './keypad.svelte';
  let showTimer = $state(false);
  let countdown = $state(0);
  let timerPosition = $state({ x: 0, y: 0 });
  let isDragging = $state(false);
  let dragStart = { x: 0, y: 0 };

  /** @param {PointerEvent} event */
  function startTimerDrag(event) {
    const target = event.target;
    if (target instanceof Element && target.closest('button, input, form')) return;

    isDragging = true;
    dragStart = {
      x: event.clientX - timerPosition.x,
      y: event.clientY - timerPosition.y
    };
    if (event.currentTarget instanceof HTMLElement) {
      event.currentTarget.setPointerCapture(event.pointerId);
    }
  }

  /** @param {PointerEvent} event */
  function moveTimer(event) {
    if (!isDragging) return;

    timerPosition = {
      x: event.clientX - dragStart.x,
      y: event.clientY - dragStart.y
    };
  }

  function stopTimerDrag() {
    isDragging = false;
  }
</script>

<section id="center">
  <div class="hero">
  </div>
  <div>
    <h1>Project 1: Smart Fridge</h1>
    <div id="info-row">
    <p>Maya Tomarchio</p>  <a id="readme-link" href="https://github.com/Maya2234/smartFridge/blob/main/README.md" target="_blank">Read Me</a>
    </div>
  </div>
</section>

<div class="ticks"></div>

<section id="prototype">
  <div id="docs">
    <svg class="icon" role="presentation" aria-hidden="true">
    </svg>
    <p></p>
<section id="fridge_UI">
  <time class="clock">
    {currentTime.toLocaleTimeString([], {
      hour: 'numeric',
      minute: '2-digit'
    })}
  </time>
  <button class="settings-icon" aria-label="Settings">⚙</button>
  <h3 class="decor">Maya</h3>
  <ul style="position: absolute; top: 40%; right: 2rem;   transform: translateY(-50%);">
    <li><button class="icon icon list" aria-label="Set Timer" onclick={() => showTimer = !showTimer}>⏲</button></li>
    <li>
        {#if showTemperature}
        <div  id="temperature-slider" >
        <label for="temperature-slider">{sliderValue}°C</label>
        <input 
          type="range"  
          min = {min} 
          max = {max} 
          step = {step} 
          bind:value={sliderValue} 
        />
        </div>
        {/if}
      <button class="icon icon list" aria-label="Adjust Temperature" onclick={() => showTemperature = !showTemperature}>🌡</button></li>
    
      <li><button class="icon icon list" aria-label="Grocery List">✎</button></li>
  </ul>

  {#if showTemperature}
    <p class="recommended-note">*recommended temperature: 4°C</p>
  {/if}

  {#if showTimer}
    <draggable
      id="timer"
      role="dialog"
      aria-label="Timer"
      tabindex="-1"
      class:dragging={isDragging}
      style={`--drag-x: ${timerPosition.x}px; --drag-y: ${timerPosition.y}px`}
      onpointerdown={startTimerDrag}
      onpointermove={moveTimer}
      onpointerup={stopTimerDrag}
      onpointercancel={stopTimerDrag}
    >
      {#if countdown}
        <Timer
          on:new={() => {
            countdown = 0;
          }}
          {countdown}
        />
      {:else}
        <Keypad
          on:countdown={(e) => {
            countdown = e.detail;
          }}
        />
      {/if}
    </draggable>
  {/if}

</section>
  </div>

</section>

<div class="ticks"></div>
<section id="spacer"></section>


