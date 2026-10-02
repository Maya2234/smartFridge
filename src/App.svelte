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

  let min = 1;
  let max = 6;
  let step = 0.5;

  let topOverlayZIndex = $state(10);
  let timerZIndex = $state(10);
  let settingsZIndex = $state(10);
  let groceriesZIndex = $state(10);
  let countdown = $state(0);
  let dragPositions = $state({
    timer: { x: 0, y: 0 },
    settings: { x: 0, y: 0 },
    decor: { x: 0, y: 0 }
  });
  /** @type {'timer' | 'settings' | 'decor' | null} */
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

  onMount(() => {
    const clock = setInterval(() => {
      currentTime = new Date();
    }, 1000);

    return () => clearInterval(clock);
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

  /** @param {PointerEvent} event @param {'timer' | 'settings' | 'decor'} draggable */
  function startDrag(event, draggable) {
    if (draggable === 'timer' || draggable === 'settings') {
      bringOverlayToFront(draggable);
    }

    const target = event.target;
    if (target instanceof Element && target.closest('button, input, select, textarea, form')) return;

    activeDrag = draggable;
    dragStart = {
      x: event.clientX - dragPositions[draggable].x,
      y: event.clientY - dragPositions[draggable].y
    };

    if (event.currentTarget instanceof HTMLElement) {
      event.currentTarget.setPointerCapture(event.pointerId);
    }
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
  }

  /** @param {string} name */
  function deleteItem(name) {
    groceries = groceries.filter(item => item.name !== name);
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
          </form>
        </div>
      {/if}

      <time class="clock">
        {currentTime.toLocaleTimeString([], {
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
        </li>
        <li>
          {#if showTemperature}
            <div id="temperature-slider">
              <label for="temperature-slider">{sliderValue}°C</label>
              <input
                type="range"
                min={min}
                max={max}
                step={step}
                bind:value={sliderValue}
              />
            </div>
          {/if}
          <button
            class="icon icon list"
            aria-label="Adjust Temperature"
            onclick={() => (showTemperature = !showTemperature)}
          >🌡</button>
        </li>
        <li>
          <button
            class="icon icon list"
            aria-label="Grocery List"
            onclick={() => (showList = !showList)}
          >✎</button>
        </li>
      </ul>

      {#if showTemperature}
        <p class="recommended-note">*recommended temperature: 4°C</p>
      {/if}

      {#if showTimer}
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
              on:countdown={(event) => {
                countdown = event.detail;
              }}
            />
          {/if}
        </div>
      {/if}

      {#if showList}
        <div
          class="grocery-app"
          role="dialog"
          tabindex="-1"
          style:z-index={groceriesZIndex}
          onpointerdown={() => bringOverlayToFront('groceries')}
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



