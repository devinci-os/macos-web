<script lang="ts">
  import { onMount } from 'svelte';
  import { colors } from '__/configs/theme/colors.config';
  import { wallpapersConfig } from '__/configs/wallpapers/wallpaper.config';
  import { prefersReducedMotion } from '__/stores/prefers-motion.store';
  import { theme } from '__/stores/theme.store';
  import { wallpaper } from '__/stores/wallpaper.store';
  import { WebView } from '__/stores/window.store';
  import DarkMode from '~icons/gg/dark-mode';
  import CheckedIcon from '~icons/ic/outline-check';
  import TransitionMaskedIcon from '~icons/mdi/transition-masked';
  import ActionCenterSurface from './ActionCenterSurface.svelte';
  import ActionCenterTile from './ActionCenterTile.svelte';
  export let isThemeWarningDialogOpen: boolean;

  let containerEl: HTMLElement;

  function toggleTheme() {
    if (wallpapersConfig[$wallpaper.id].type === 'dynamic' && $wallpaper.canControlTheme) {
      isThemeWarningDialogOpen = true;
      return;
    }

    $theme.scheme = $theme.scheme === 'light' ? 'dark' : 'light';
  }

  function toggleMotionPreference() {
    $prefersReducedMotion = !$prefersReducedMotion;
  }

  function openWallpapersApp() {
    new WebView({
      title: 'Wallpaper',
      appID: 'wallpapers',
      loadComponent: async () =>
        (await import('@ui/components/apps/WallpaperApp/WallpaperSelectorApp.svelte')).default,
    }).open();
  }

  onMount(() => containerEl?.focus());
</script>

<section
  class="container"
  class:dark={$theme.scheme === 'dark'}
  tabindex={-1}
  bind:this={containerEl}
>
  <!-- Main controls: Wifi, Bluetooth, Airdrop -->
  <ActionCenterSurface
    grid={[
      [1, 6],
      [1, 2],
    ]}
  >
    <ActionCenterTile grid={[1, 1]}>
      <button class="toggle" class:filled={$theme.scheme === 'dark'} on:click={toggleTheme}>
        <DarkMode />
      </button>
      Dark mode
    </ActionCenterTile>
  </ActionCenterSurface>

  <ActionCenterSurface
    grid={[
      [7, 6],
      [1, 2],
    ]}
  >
    <ActionCenterTile grid={[1, 1]}>
      <button
        class="toggle"
        class:filled={!$prefersReducedMotion}
        on:click={toggleMotionPreference}
      >
        <TransitionMaskedIcon />
      </button>
      Animations
    </ActionCenterTile>
  </ActionCenterSurface>

  <ActionCenterSurface
    grid={[
      [1, 12],
      [3, 2],
    ]}
  >
    <ActionCenterTile grid={[1, 1]}>
      <div class="color-picker">
        <p>System Color</p>
        <div class="color-palette">
          {#each Object.keys(colors) as colorID}
            <button
              style="--color-hsl: {colors[colorID][$theme.scheme]
                .hsl}; --color-contrast-hsl: {colors[colorID][$theme.scheme].contrastHsl}"
              on:click={() => ($theme.primaryColor = colorID)}
            >
              {#if $theme.primaryColor === colorID}
                <CheckedIcon />
              {/if}
            </button>
          {/each}
        </div>
      </div>
    </ActionCenterTile>
  </ActionCenterSurface>

  <ActionCenterSurface
    grid={[
      [1, 12],
      [5, 3],
    ]}
  >
    <ActionCenterTile
      focusable={true}
      grid={[1, 1]}
      on:click={openWallpapersApp}
      on:keyup={(e) => ['Enter', 'Space Bar'].includes(e.key) && openWallpapersApp()}
    >
      <div class="wallpaper-tile">
        <img
          class="wallpaper-thumbnail"
          src="/assets/wallpapers/{wallpapersConfig[$wallpaper.id].thumbnail}.jpg"
          alt="Current wallpaper"
        />

        <div class="wallpaper-info">
          <h3>{wallpapersConfig[$wallpaper.id].name}</h3>
          <p>{wallpapersConfig[$wallpaper.id].type} wallpaper</p>
        </div>
      </div>
    </ActionCenterTile>
  </ActionCenterSurface>
</section>

<style lang="scss">
  .container {
    --border-size: 0;

    display: grid;
    grid-template-columns: repeat(12, 1fr);
    grid-auto-rows: minmax(1.55rem, auto);
    gap: 0.75rem;

    width: 19.5rem;

    padding: 0.75rem;

    position: relative;

    user-select: none;

    background-color: hsla(var(--system-color-light-hsl), 0.3);
    backdrop-filter: blur(12px);

    border-radius: 1rem;

    box-shadow: hsla(0, 0%, 0%, 0.3) 0px 0px 11px 0px,
      inset 0 0 0 var(--border-size) hsla(var(--system-color-dark-hsl), 0.3),
      0 0 0 var(--border-size) hsla(var(--system-color-light-hsl), 0.3);

    &.dark {
      --border-size: 0.5px;
    }
  }

  .toggle {
    --size: 1.7rem;

    --bgcolor: var(--system-color-dark-hsl);
    --bgalpha: 0.1;

    --svgcolor: var(--system-color-light-contrast-hsl);
    --svgalpha: 0.9;

    height: var(--size) !important;
    width: var(--size);

    padding: 0;

    display: flex;
    place-items: center;

    border-radius: 50%;

    background-color: hsla(var(--bgcolor), var(--bgalpha));

    transition: box-shadow 100ms ease, background-color 150ms ease;

    :global(svg) {
      color: hsla(var(--svgcolor), var(--svgalpha));
    }

    &:focus-visible {
      box-shadow: 0 0 0 0.25rem hsla(var(--bgcolor), 0.4);
    }

    &.filled {
      --bgcolor: var(--system-color-primary-hsl);
      --bgalpha: 1;

      --svgcolor: var(--system-color-primary-contrast-hsl);
      --svgalpha: 1;
    }
  }

  .wallpaper-tile {
    height: 100%;
    width: 100%;

    display: grid;
    grid-template-columns: auto 1fr;
    gap: 1rem;
    align-items: center;

    padding: 0.25rem 0 0.25rem 0.25rem;

    img {
      aspect-ratio: 1 / 1;
      height: 4.5rem;

      object-fit: cover;

      border-radius: 0.5rem;
    }

    h3 {
      width: 100%;

      font-size: 1rem;
      line-height: 1.618;
    }

    p {
      text-transform: capitalize;
      font-size: 0.8rem;
      font-weight: 400;
    }
  }

  .color-picker {
    height: max-content;
    width: 100%;

    display: grid;
    gap: 0.5rem;

    padding: 0.25rem;

    .color-palette {
      margin-top: 0.5rem;

      display: flex;
      justify-content: space-between;

      width: 100%;

      button {
        height: 1.4rem;
        width: 1.4rem;

        color: hsl(var(--color-contrast-hsl));

        border-radius: 50%;

        background-color: hsl(var(--color-hsl));

        transition: box-shadow 200ms ease-in;

        &:hover,
        &:focus-visible {
          box-shadow: 0 0 0 0.2rem hsla(var(--color-hsl), 0.25);
        }
      }
    }
  }
</style>
