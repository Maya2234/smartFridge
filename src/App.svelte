<script>
  import { onMount } from 'svelte';
  import Keypad from './keypad.svelte';
  import Timer from './timer.svelte';

  let currentTime = $state(new Date());
  let sliderValue = $state(4);
  let showTemperature = $state(false);
  let showTimer = $state(false);
  let showSettings = $state(false);
  let showList = $state(false);
  let settingsView = $state('display');   // display or schedule
  let toBuy = $state(false);

  //temprature slider values
  let min = 1;
  let max = 6;
  let step = 0.5;

  let timerOffset = $state(1);
  let timerRunning = $state(false);

  let topOverlayZIndex = $state(10);
  let timerZIndex = $state(10);
  let settingsZIndex = $state(10);
  let groceriesZIndex = $state(10);
  let countdown = $state(0);
  let dragPositions = $state({
    timer: { x: 0, y: 0 },
    settings: { x: 0, y: 0 },
    decor: { x: 0, y: 0 },
    groceries: { x: 0, y: 0 }
  });
  /** @type {'timer' | 'settings' | 'decor' | 'groceries' | null} */
  let activeDrag = $state(null);
  let dragStart = { x: 0, y: 0 };

  const colors = [
    { name: 'Cream', value: '#fdf6ec' },
    { name: 'Lavender', value: '#e6dcf5' },
    { name: 'Sky', value: '#d6ecfa' },
    { name: 'Mint', value: '#dcf2e4' },
    { name: 'Charcoal', value: '#2b2b33' },
    { name: 'white', value: '#ffffff' }
  ];

  const textColors = [
    { name: 'Cream', value: 'black' },
    { name: 'Hot Pink', value: 'rgb(255, 17, 160)' },
    { name: 'Sky', value: '#d6ecfa' },
    { name: 'Mint', value: '#dcf2e4' },
    { name: 'Charcoal', value: '#2b2b33' },
    { name: 'white', value: '#ffffff' }
  ];

  let selectedColor = $state(colors[0].value);
  let decorText = $state('Maya');
  let decorTextColor = $state(textColors[0].value);

  let templates = $state([
    { name: 'Default', color: colors[0].value, text: 'Maya', textColor: textColors[0].value },
    { name: 'Halftone', color: colors[1].value, text: 'Hello', textColor: textColors[1].value },
    { name: 'Hotel Lobby', color: colors[2].value, text: 'Welcome', textColor: textColors[2].value },
    { name: 'Garden', color: colors[3].value, text: 'Fresh', textColor: textColors[4].value },
    { name: 'Charcoal', color: colors[4].value, text: 'XD', textColor: textColors[3].value },
    { name: 'Juicy', color: colors[5].value, text: 'Bright!', textColor: textColors[1].value }
  ]);
  let selectedTemplate = $state('');
  let newTemplateName = $state('');
  /** @type {{name: string, bought: boolean}[]} */
  let groceries = $state([]);
  let newItemName = $state('');
  let itemsLeft = $derived(groceries.filter(item => !item.bought).length);

  /** @type {{id:number, template: string, time: string}[]} */
  let schedules = $state([]);          // { id, template, time }
  let scheduleTemplate = $state('');
  let scheduleTime = $state('');       // "HH:MM" from <input type="time">
  let lastCheckedTime = '';            // plain variable, doesn't need $state

  let simulating = $state(false);
  let simMinutes = $state(0);          // 0 to 1439 = minutes since midnight

  const SIM_SECONDS = 120;             // how long a full day takes
  const TICK_MS = 100;                 // how often the sim clock updates
  const MINUTES_PER_TICK = 1440 / ((SIM_SECONDS * 1000) / TICK_MS);   // 1.2
  let simElapsedSeconds = $derived(Math.floor((simMinutes / 1440) * SIM_SECONDS));

/** @type {ReturnType<typeof setInterval> | undefined} */
  let simTimer;
/** @type {{ color: string, text: string, textColor: string } | null} */
  let savedLook = null;

  /** @param {Date} d */
function localTime(d) {
  return `${String(d.getHours()).padStart(2, '0')}:${String(d.getMinutes()).padStart(2, '0')}`;
}

function addSchedule() {
  if (!scheduleTemplate || !scheduleTime) return;

  schedules = [
    ...schedules,
    { id: Date.now(), template: scheduleTemplate, time: scheduleTime }
  ].sort((a, b) => a.time.localeCompare(b.time));
  scheduleTime = '';
}

/** @param {number} id */
function deleteSchedule(id) {
  schedules = schedules.filter(s => s.id !== id);
}

/** @param {Date} now */
function checkSchedules(now) {
  const time = localTime(now);
  if (time === lastCheckedTime) return;   // still the same minute, already handled
  lastCheckedTime = time;

  for (const s of schedules) {
    if (s.time === time) {
      applyTemplate(s.template);
      selectedTemplate = s.template;
    }
  }
}
  let displayTime = $derived.by(() => {
  if (!simulating) return currentTime;
  const d = new Date();
  d.setHours(0, 0, 0, 0);
  d.setMinutes(Math.floor(simMinutes));
  return d;
});


/** @param {number} m */
function minutesToTime(m) {
  return `${String(Math.floor(m / 60)).padStart(2, '0')}:${String(m % 60).padStart(2, '0')}`;
}

/** @param {string} time */
function runSchedulesFor(time) {
  for (const s of schedules) {
    if (s.time === time) {
      applyTemplate(s.template);
      selectedTemplate = s.template;
    }
  }
}

function startSimulation() {
  if (simulating) return;

  savedLook = { color: selectedColor, text: decorText, textColor: decorTextColor };
  simMinutes = 0;
  simulating = true;
  runSchedulesFor('00:00');

  simTimer = setInterval(() => {
    const prev = Math.floor(simMinutes);
    const next = simMinutes + MINUTES_PER_TICK;

    // fire every minute we just passed over, not just the one we landed on
    const last = Math.min(Math.floor(next), 1439);
    for (let m = prev + 1; m <= last; m++) runSchedulesFor(minutesToTime(m));

    if (next >= 1440) {
      stopSimulation();
    } else {
      simMinutes = next;
    }
  }, TICK_MS);
}

function stopSimulation() {
  clearInterval(simTimer);
  simulating = false;
  if (savedLook) {
    selectedColor = savedLook.color;
    decorText = savedLook.text;
    decorTextColor = savedLook.textColor;
    savedLook = null;
  }
}

onMount(() => {
  const clock = setInterval(() => {
    currentTime = new Date();
    if (!simulating) checkSchedules(currentTime);   // real schedules pause during a sim
  }, 1000);

  return () => {
    clearInterval(clock);
    clearInterval(simTimer);                        // don't leave the sim running if the page unmounts
  };
});

  /** @param {'timer' | 'settings' | 'groceries'} overlay */
  function bringOverlayToFront(overlay) {
    topOverlayZIndex += 1;
    if (overlay === 'timer') timerZIndex = topOverlayZIndex;
    if (overlay === 'settings') settingsZIndex = topOverlayZIndex;
    if (overlay === 'groceries') groceriesZIndex = topOverlayZIndex;
  }

  /** @param {PointerEvent} event */
  function raiseGroceriesWhenOpened(event) {
    const target = event.target;
    if (target instanceof Element && target.closest('[aria-label="Grocery List"]') && !showList) {
      bringOverlayToFront('groceries');
    }
  }

  /** @param {PointerEvent} event @param {'timer' | 'settings' | 'decor' | 'groceries'} draggable */
  function startDrag(event, draggable) {
    if (draggable !== 'decor') {
      bringOverlayToFront(draggable);
    }

    const target = event.target;
    if (target instanceof Element && target.closest('button, input, select, textarea, form')) return;

    activeDrag = draggable;
    dragStart = {
      x: event.clientX - dragPositions[draggable].x,
      y: event.clientY - dragPositions[draggable].y
    };
  }

  /** @param {PointerEvent} event */
  function moveDrag(event) {
    if (!activeDrag) return;

    dragPositions[activeDrag] = {
      x: event.clientX - dragStart.x,
      y: event.clientY - dragStart.y
    };
  }

  function stopDrag() {
    activeDrag = null;
  }

  function addItem() {
    if (newItemName.trim() === '') return;

    groceries = [...groceries, { name: newItemName.trim(), bought: false }];
    newItemName = '';

    toBuy = true;

  }

  /** @param {string} name */
  function deleteItem(name) {
    groceries = groceries.filter(item => item.name !== name);
    if (groceries.length === 0) toBuy = false;
  }

  /** @param {string} name */
  function applyTemplate(name) {
    const template = templates.find(item => item.name === name);
    if (!template) return;

    selectedColor = template.color;
    decorText = template.text;
    decorTextColor = template.textColor;
  }

  function addTemplate() {
    const name = newTemplateName.trim();
    if (name === '') return;

    const existingTemplate = templates.find(template => template.name === name);
    if (existingTemplate) {
      existingTemplate.color = selectedColor;
      existingTemplate.text = decorText;
      existingTemplate.textColor = decorTextColor;
    } else {
      templates = [
        ...templates,
        { name, color: selectedColor, text: decorText, textColor: decorTextColor }
      ];
    }

    selectedTemplate = name;
    newTemplateName = '';
  }
</script>

<section id="center">
  <div class="hero"></div>
  <div>
    <h1>Project 1: Smart Fridge</h1>
    <div id="info-row">
      <p>Maya Tomarchio</p>
      <a
        id="readme-link"
        href="https://github.com/Maya2234/smartFridge/blob/main/README.md"
        target="_blank"
      >Read Me</a>
    </div>
  </div>
</section>

<section id="prototype">
  <div id="docs">
    <svg class="icon" role="presentation" aria-hidden="true">
    </svg>
    <p></p>
    <b style="text-align:center; display: block; margin:0;">SIMULATE</b>
    {#if !simulating}
    <div style="text-align: center;">
    <button class="sim-buttons" style="width:3rem;height:3rem; border-radius: 50%;" onclick={startSimulation}>▶</button>
    </div>
    {:else}
    <div class="sim-controls">
      <button class="sim-buttons" style="width:3rem;height:3rem; border-radius: 50%;" onclick={stopSimulation}>■</button>
      <div class="sim-progress" aria-hidden="true">
        <div class="sim-progress-fill" style:width={`${(simMinutes / 1440) * 100}%`}></div>
      </div>
      <span class="sim-time">
        {String(Math.floor(simElapsedSeconds / 60)).padStart(2, '0')}:{String(simElapsedSeconds % 60).padStart(2, '0')} / 02:00
      </span>
    </div>
    {/if}
    <br>
    <section id="fridge_UI" style:background-color={selectedColor}>

      <!-- Settings overlay -->
      {#if showSettings}
        <div
          id="settings"
          role="dialog"
          aria-label="Settings"
          tabindex="-1"
          class:dragging={activeDrag === 'settings'}
          style={`--drag-x: ${dragPositions.settings.x}px; --drag-y: ${dragPositions.settings.y}px; z-index: ${settingsZIndex}`}
          onpointerdown={(event) => startDrag(event, 'settings')}
          onpointermove={moveDrag}
          onpointerup={stopDrag}
          onpointercancel={stopDrag}
        >
          <h1>Settings</h1>
          <button id="close" onclick={() => (showSettings = !showSettings)}>X</button>


           {#if settingsView === 'display'}
            <!-- background color, decor text, decor text color, template dropdown, save form -->

          <div class="setting-row">
            <label for="theme-color">Background Color:</label>
            <div class="bar" role="group" aria-label="Background color">
              {#each colors as color (color.value)}
                <button
                  class="swatch"
                  class:active={selectedColor === color.value}
                  style:background-color={color.value}
                  aria-label={color.name}
                  aria-pressed={selectedColor === color.value}
                  onclick={() => (selectedColor = color.value)}
                ></button>
              {/each}
            </div>
          </div>

          <div class="setting-row">
            <label style="padding-right: 1.5rem;">
              Decor Text:
              <input
                style="flex: 0 0 12rem; margin-left: 3.3rem;"
                type="text"
                id="decor-text"
                bind:value={decorText}
                placeholder="decor text...."
              />
            </label>
          </div>

          <div class="setting-row">
            <label for="theme-color">Decor Text Color:</label>
            <div class="bar" role="group" aria-label="Decor text color">
              {#each textColors as color (color.value)}
                <button
                  class="text-swatch"
                  class:active={decorTextColor === color.value}
                  style:background-color={color.value}
                  aria-label={color.name}
                  aria-pressed={decorTextColor === color.value}
                  onclick={() => (decorTextColor = color.value)}
                ></button>
              {/each}
            </div>
          </div>

          <div class="setting-row">
            <label for="template-select">Template:</label>
            <select
              id="template-select"
              bind:value={selectedTemplate}
              onchange={(event) => applyTemplate(event.currentTarget.value)}
            >
              <option value="" disabled>Choose a template…</option>
              {#each templates as template (template.name)}
                <option value={template.name}>{template.name}</option>
              {/each}
            </select>
          </div>

          <form
            id="setting-row"
            onsubmit={(event) => {
              event.preventDefault();
              addTemplate();
            }}
          >
            <input
              style="max-width: 50%;"
              type="text"
              bind:value={newTemplateName}
              placeholder="template name..."
            />
            <button type="submit">Save current as template</button>
            
          </form><div>
            <button class="setting-row schedule" type="submit" onclick={() => (settingsView = 'schedule')}>Template Schedule</button></div>
          {:else}
            <!-- SCHEDULE TEMPLTES-->
            <h2 style="margin-right: auto;">Template Schedule</h2>
            <form class="setting-row" onsubmit={(event) => {event.preventDefault(); addSchedule(); }}>
              <select bind:value={scheduleTemplate}>
                <option value="" disabled>Template…</option>
                {#each templates as template (template.name)}
                  <option value={template.name}>{template.name}</option>
                {/each}
              </select>
              <input type="time" bind:value={scheduleTime} />
              <button type="submit">Add</button>
            </form>
        <div class="schedule-list">
            {#each schedules as s (s.id)}
              <div class="schedule-item">
                <span>{s.time}: {s.template}</span>
                <button onclick={() => deleteSchedule(s.id)}>&times;</button>
              </div>
            {/each}           
          </div>
            <button style="  margin-top: auto;  align-self: flex-start;" onclick={() => (settingsView = 'display')}>← Back</button>
          {/if}
        </div>
      {/if}

      <time class="clock">
        {displayTime.toLocaleTimeString([], {
          hour: 'numeric',
          minute: '2-digit'
        })}
      </time>

      <button
        class="settings-icon"
        aria-label="Settings"
        onclick={() => {
          if (!showSettings) bringOverlayToFront('settings');
          showSettings = !showSettings;
        }}
      >⚙</button>

      <div class="decor-clip" aria-hidden="true">
        <h3
          class="decor"
          class:dragging={activeDrag === 'decor'}
          style:color={decorTextColor}
          style={`--drag-x: ${dragPositions.decor.x}px; --drag-y: ${dragPositions.decor.y}px`}
          onpointerdown={(event) => startDrag(event, 'decor')}
          onpointermove={moveDrag}
          onpointerup={stopDrag}
          onpointercancel={stopDrag}
        >
          {decorText}
        </h3>
      </div>

      <ul
        style="position: absolute; top: 40%; right: 2rem; transform: translateY(-50%);"
        onpointerdown={raiseGroceriesWhenOpened}
      >
        <li>
          <button
            class="icon icon list"
            aria-label="Set Timer"
            onclick={() => {
              if (!showTimer) bringOverlayToFront('timer');
              showTimer = !showTimer;
            }}
          >⏲</button>
          {#if timerRunning}
            <svg class="timer-ring" viewBox="-50 -50 100 100" aria-hidden="true">
              <circle r="46" fill="none" stroke="rgba(0,0,0,0.15)" stroke-width="5" />
              <path
                d="M 0 -46 a 46 46 0 0 0 0 92 46 46 0 0 0 0 -92"
                fill="none"
                stroke="hsl(208, 100%, 50%)"
                stroke-width="5"
                stroke-linecap="round"
                pathLength="1"
                stroke-dasharray="1"
                stroke-dashoffset={timerOffset}
              />
            </svg>
          {/if}
        </li>
        <li>
          {#if showTemperature}
            <div id="temperature-slider">
              <label
                id="temperature-label"
                for="temperature-slider"
                class:recommended={sliderValue === 4}>

                {sliderValue}°C
              </label>
              <input
                type="range"
                min={min}
                max={max}
                step={step}
                bind:value={sliderValue}
                aria-valuetext={sliderValue === 4 ? 'recommended temperature' : undefined}
              />
              {#if sliderValue === 4}
                <span id="temperature-label" class="recommended">*Recommended</span>
              {/if}            </div>
          {/if}
          <button
            class="icon icon list"
            aria-label="Adjust Temperature"
            onclick={() => (showTemperature = !showTemperature)}
          >🌡</button>
        </li>
        <li>
          <button
            class="icon icon list grocery-icon"
            aria-label="Grocery List"
            onclick={() => (showList = !showList)}
          >
            ✎
            {#if toBuy}
              <span class="grocery-badge" aria-hidden="true">{itemsLeft}</span>
            {/if}
          </button>
        </li>
      </ul>

        <div
          id="timer"
          role="dialog"
          aria-label="Timer"
          tabindex="-1"
          class:dragging={activeDrag === 'timer'}
          style={`--drag-x: ${dragPositions.timer.x}px; --drag-y: ${dragPositions.timer.y}px; z-index: ${timerZIndex}`}
          onpointerdown={(event) => startDrag(event, 'timer')}
          onpointermove={moveDrag}
          onpointerup={stopDrag}
          onpointercancel={stopDrag}
          style:display={showTimer ? undefined : 'none'}
        >
          {#if countdown}
          <Timer
            on:new={() => {
              timerRunning = false;
              countdown = 0;
            }}
            on:runningChange={(event) => (timerRunning = event.detail)}
            {countdown}
            bind:progress={timerOffset}
          />
          {:else}
          <Keypad
            on:countdown={(event) => {
              timerOffset = 1;          // start with an empty ring
              countdown = event.detail;
              timerRunning = true;
            }}
          />
          {/if}
        </div>
      {#if showList}
        <div
          class="grocery-app"
          class:dragging={activeDrag === 'groceries'}
          style={`--drag-x: ${dragPositions.groceries.x}px; --drag-y: ${dragPositions.groceries.y}px; z-index: ${groceriesZIndex}`}
          onpointerdown={(event) => startDrag(event, 'groceries')}
          onpointermove={moveDrag}
          onpointerup={stopDrag}
          onpointercancel={stopDrag}
          role="dialog"
          tabindex="-1"
          style:z-index={groceriesZIndex}
        >
          <h2>Groceries</h2>

          <form
            onsubmit={(event) => {
              event.preventDefault();
              addItem();
            }}
          >
            <input type="text" bind:value={newItemName} placeholder="new item..." />
            <button type="submit">+</button>
          </form>

          {#if groceries.length === 0}
            <p class="empty-msg"></p>
          {:else}
            <p class="summary">{itemsLeft} item(s) to buy</p>

            <ul>
              {#each groceries as item (item.name)}
                <li class:completed={item.bought}>
                  <label>
                    <input type="checkbox" bind:checked={item.bought} />
                    <span class="item-name">{item.name}</span>
                  </label>
                  <button class="delete-btn" onclick={() => deleteItem(item.name)}>
                    &times;
                  </button>
                </li>
              {/each}
            </ul>
          {/if}
        </div>
      {/if}
    </section>

    
  </div>
</section>



