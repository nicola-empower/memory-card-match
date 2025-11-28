<script>
  import { onMount } from "svelte";
  import Card from "./lib/Card.svelte";
  import { Wifi, Shield, Cpu, Database, Zap, Code } from "lucide-svelte";
  import confetti from "canvas-confetti";

  // Game Configuration
  const ICONS = [
    { component: Wifi, color: "#22d3ee" }, // Cyan
    { component: Shield, color: "#c084fc" }, // Purple
    { component: Cpu, color: "#4ade80" }, // Green
    { component: Database, color: "#f472b6" }, // Pink
    { component: Zap, color: "#fbbf24" }, // Amber/Yellow
    { component: Code, color: "#ef4444" }, // Red
  ];

  let cards = $state([]);
  let flippedCards = $state([]);
  let timeoutId = $state(null); // Track timeout for Fast Play cancellation

  // Stats
  let moves = $state(0);
  let bestScore = $state(null);

  function initGame() {
    // Reset stats
    moves = 0;
    flippedCards = [];
    if (timeoutId) clearTimeout(timeoutId);

    // Create pairs
    const deck = [...ICONS, ...ICONS].map((item, index) => ({
      id: index,
      icon: item.component,
      color: item.color,
      isFlipped: false,
      isMatched: false,
    }));

    // Shuffle
    cards = deck.sort(() => Math.random() - 0.5);
  }

  function handleCardClick(index) {
    // Ignore if already flipped or matched
    if (cards[index].isFlipped || cards[index].isMatched) return;

    // Fast Play: If 2 cards are already flipped (waiting for mismatch timeout), reset them immediately
    if (flippedCards.length === 2) {
      clearTimeout(timeoutId);
      const [first, second] = flippedCards;
      cards[first].isFlipped = false;
      cards[second].isFlipped = false;
      flippedCards = [];
    }

    // Flip the new card
    cards[index].isFlipped = true;
    flippedCards.push(index);

    // Check for match if 2 cards are flipped
    if (flippedCards.length === 2) {
      moves++;
      checkForMatch();
    }
  }

  function checkForMatch() {
    const [firstIndex, secondIndex] = flippedCards;
    const firstCard = cards[firstIndex];
    const secondCard = cards[secondIndex];

    if (firstCard.icon === secondCard.icon) {
      // Match!
      cards[firstIndex].isMatched = true;
      cards[secondIndex].isMatched = true;
      flippedCards = [];

      // Check for Win
      if (cards.every((c) => c.isMatched)) {
        handleWin();
      }
    } else {
      // No Match - wait 1.5s then flip back (unless interrupted by Fast Play)
      timeoutId = setTimeout(() => {
        cards[firstIndex].isFlipped = false;
        cards[secondIndex].isFlipped = false;
        flippedCards = [];
      }, 1500);
    }
  }

  function handleWin() {
    confetti({
      particleCount: 150,
      spread: 70,
      origin: { y: 0.6 },
      colors: ["#22d3ee", "#c084fc", "#4ade80", "#f472b6"],
    });

    if (bestScore === null || moves < bestScore) {
      bestScore = moves;
      localStorage.setItem("memory-game-best-score", moves.toString());
    }
  }

  onMount(() => {
    const savedScore = localStorage.getItem("memory-game-best-score");
    bestScore = savedScore ? parseInt(savedScore) : null;
    initGame();
  });
</script>

<main>
  <div class="layout">
    <!-- Game Section -->
    <div class="game-board">
      <div class="header">
        <h1>MEMORY <span class="highlight">MATCH</span></h1>

        <div class="stats">
          <div class="stat-box">
            <span class="label">MOVES</span>
            <span class="value">{moves}</span>
          </div>
          <div class="stat-box">
            <span class="label">BEST</span>
            <span class="value">{bestScore ?? "-"}</span>
          </div>
        </div>

        <button onclick={initGame}>Reset Game</button>
      </div>

      <div class="grid-container">
        {#each cards as card, index}
          <Card
            icon={card.icon}
            color={card.color}
            flipped={card.isFlipped}
            matched={card.isMatched}
            onclick={() => handleCardClick(index)}
          />
        {/each}
      </div>
    </div>

    <!-- Sidebar Section -->
    <div class="sidebar">
      <div class="instructions">
        <h2>HOW TO PLAY</h2>
        <ul>
          <li>
            <span class="step-number">1</span>
            <p>Click a card to reveal its <strong>Neon Icon</strong>.</p>
          </li>
          <li>
            <span class="step-number">2</span>
            <p>Find the matching icon to keep them <strong>Face Up</strong>.</p>
          </li>
          <li>
            <span class="step-number">3</span>
            <p>
              Match all pairs in the fewest <strong>Moves</strong> possible.
            </p>
          </li>
          <li>
            <span class="step-number">4</span>
            <p>
              Beat your <strong>Best Score</strong> and trigger the confetti!
            </p>
          </li>
        </ul>
      </div>

      <div class="tech-specs">
        <h2>UNDER THE HOOD</h2>
        <div class="spec-item">
          <span class="spec-label">Framework</span>
          <span class="spec-value">Svelte 5</span>
        </div>
        <div class="spec-item">
          <span class="spec-label">State</span>
          <span class="spec-value">Local Reactivity</span>
        </div>
        <div class="spec-item">
          <span class="spec-label">Icons</span>
          <span class="spec-value">Lucide SVG</span>
        </div>
        <div class="spec-item">
          <span class="spec-label">Hosting</span>
          <span class="spec-value">Vercel</span>
        </div>
      </div>
    </div>
  </div>

  <footer>
    <p>
      Created by <a href="https://nicolaberry.dev" target="_blank"
        >Nicola Berry</a
      >
    </p>
  </footer>
</main>

<style>
  main {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: space-between; /* Pushes footer to bottom */
    min-height: 100vh;
    padding: 2rem;
  }

  .layout {
    display: flex;
    gap: 4rem;
    align-items: flex-start;
    max-width: 1200px;
    width: 100%;
    margin: auto 0; /* Centers layout vertically in remaining space */
  }

  .game-board {
    flex: 1;
    display: flex;
    flex-direction: column;
    align-items: center;
  }

  .sidebar {
    width: 300px;
    padding-top: 2rem;
    animation: slideIn 0.5s ease-out;
  }

  .instructions {
    background: rgba(30, 41, 59, 0.5);
    border: 1px solid #334155;
    border-radius: 16px;
    padding: 2rem;
    backdrop-filter: blur(10px);
  }

  h2 {
    color: #22d3ee;
    font-size: 1.5rem;
    margin: 0 0 1.5rem 0;
    letter-spacing: 0.1em;
    text-shadow: 0 0 10px rgba(34, 211, 238, 0.3);
  }

  ul {
    list-style: none;
    padding: 0;
    margin: 0;
    display: flex;
    flex-direction: column;
    gap: 1.5rem;
  }

  li {
    display: flex;
    gap: 1rem;
    align-items: flex-start;
  }

  .step-number {
    background: #334155;
    color: #f8fafc;
    width: 24px;
    height: 24px;
    border-radius: 50%;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 0.875rem;
    font-weight: 700;
    flex-shrink: 0;
    margin-top: 2px;
  }

  p {
    margin: 0;
    color: #94a3b8;
    line-height: 1.5;
    font-size: 0.95rem;
  }

  strong {
    color: #e2e8f0;
  }

  .header {
    margin-bottom: 2rem;
    text-align: center;
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 1.5rem;
  }

  h1 {
    font-size: 3rem;
    font-weight: 800;
    letter-spacing: -0.05em;
    margin: 0;
    color: #e2e8f0;
    text-shadow: 0 0 20px rgba(255, 255, 255, 0.1);
  }

  .highlight {
    color: #22d3ee;
    text-shadow: 0 0 10px #22d3ee;
  }

  .stats {
    display: flex;
    gap: 2rem;
  }

  .stat-box {
    display: flex;
    flex-direction: column;
    align-items: center;
  }

  .label {
    font-size: 0.75rem;
    text-transform: uppercase;
    letter-spacing: 0.1em;
    color: #94a3b8;
    margin-bottom: 0.25rem;
  }

  .value {
    font-size: 1.5rem;
    font-weight: 700;
    color: #f8fafc;
    font-variant-numeric: tabular-nums;
  }

  button {
    background: transparent;
    border: 1px solid #334155;
    color: #94a3b8;
    padding: 0.5rem 1.5rem;
    border-radius: 999px;
    font-weight: 600;
    transition: all 0.2s;
  }

  button:hover {
    border-color: #22d3ee;
    color: #22d3ee;
    box-shadow: 0 0 15px rgba(34, 211, 238, 0.2);
  }

  .grid-container {
    display: grid;
    grid-template-columns: repeat(4, 1fr);
    gap: 1.5rem;
    max-width: 600px;
    margin: 0 auto;
  }

  .tech-specs {
    background: rgba(30, 41, 59, 0.5);
    border: 1px solid #334155;
    border-radius: 16px;
    padding: 2rem;
    backdrop-filter: blur(10px);
    margin-top: 1.5rem;
  }

  .spec-item {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 0.75rem 0;
    border-bottom: 1px solid rgba(51, 65, 85, 0.5);
  }

  .spec-item:last-child {
    border-bottom: none;
  }

  .spec-label {
    color: #94a3b8;
    font-size: 0.9rem;
  }

  .spec-value {
    color: #f8fafc;
    font-weight: 600;
    font-family: monospace;
    background: rgba(34, 211, 238, 0.1);
    padding: 0.2rem 0.5rem;
    border-radius: 4px;
    color: #22d3ee;
    font-size: 0.85rem;
  }

  footer {
    margin-top: 3rem;
    text-align: center;
    color: #64748b;
    font-size: 0.9rem;
  }

  footer a {
    color: #94a3b8;
    text-decoration: none;
    transition: color 0.2s;
  }

  footer a:hover {
    color: #22d3ee;
  }

  @keyframes slideIn {
    from {
      opacity: 0;
      transform: translateX(20px);
    }
    to {
      opacity: 1;
      transform: translateX(0);
    }
  }

  @media (max-width: 900px) {
    .layout {
      flex-direction: column;
      align-items: center;
      gap: 3rem;
    }

    .sidebar {
      width: 100%;
      max-width: 500px;
      padding-top: 0;
    }
  }

  @media (max-width: 500px) {
    .grid-container {
      grid-template-columns: repeat(3, 1fr);
      gap: 1rem;
    }

    h1 {
      font-size: 2rem;
    }

    main {
      padding: 1rem;
    }
  }
</style>
