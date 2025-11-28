<script>
  let { icon: Icon, flipped, matched, color, onclick } = $props();
</script>

<!-- svelte-ignore a11y_click_events_have_key_events -->
<!-- svelte-ignore a11y_no_static_element_interactions -->
<div class="card" class:flipped={flipped || matched} {onclick}>
  <div class="inner">
    <div class="front">
      <!-- Face Down -->
    </div>
    <div class="back" style="--glow-color: {color}">
      <!-- Face Up -->
      <Icon size={48} {color} class="icon" />
    </div>
  </div>
</div>

<style>
  .card {
    width: 100px;
    height: 100px;
    perspective: 1000px;
    cursor: pointer;
  }

  .inner {
    position: relative;
    width: 100%;
    height: 100%;
    text-align: center;
    transition: transform 0.3s;
    transform-style: preserve-3d;
    will-change: transform;
  }

  .card.flipped .inner {
    transform: rotateY(180deg);
  }

  .card:hover .inner {
    transform: translateY(-4px);
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.5);
  }

  .front,
  .back {
    position: absolute;
    width: 100%;
    height: 100%;
    -webkit-backface-visibility: hidden;
    backface-visibility: hidden;
    border-radius: 12px;
    display: flex;
    align-items: center;
    justify-content: center;
    border: 1px solid #334155;
  }

  .front {
    background-color: #1e293b;
  }

  .back {
    background-color: #1e293b;
    transform: rotateY(180deg);
    border-color: var(--glow-color);
    box-shadow: 0 0 15px var(--glow-color, rgba(255, 255, 255, 0.1));
  }

  /* Global style for icon glow passed from parent or handled here */
  :global(.icon) {
    filter: drop-shadow(0 0 8px currentColor);
  }
</style>
