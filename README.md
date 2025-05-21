<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>High School Basketball Manager</title>
<style>
  body { font-family: Arial, sans-serif; margin: 20px; background: #f0f0f0; }
  h1, h2 { color: #222; }
  table { border-collapse: collapse; width: 100%; margin-bottom: 20px; background: white; }
  th, td { border: 1px solid #ccc; padding: 8px; text-align: center; }
  th { background: #555; color: white; }
  button { padding: 6px 12px; margin: 4px 0; cursor: pointer; }
  button:disabled { background: #999; cursor: default; }
  .injured { color: red; font-weight: bold; }
  #log { background: #222; color: #eee; padding: 10px; height: 180px; overflow-y: auto; font-family: monospace; font-size: 14px; white-space: pre-wrap; }
  #controls { margin-bottom: 10px; }
  .starter { font-weight: bold; background: #dff0d8; }
</style>
</head>
<body>

<h1>High School Basketball Team Manager</h1>

<div id="controls">
  <button id="simulateGameBtn">Simulate Game</button>
  <button id="nextSeasonBtn" disabled>Next Season</button>
  <span style="margin-left:20px;">Season: <strong id="seasonNumber">1</strong></span>
  <span style="margin-left:20px;">Record: <strong id="record">0 - 0</strong></span>
  <span style="margin-left:20px;">Game: <strong id="gameNumber">0</strong> / 10</span>
</div>

<h2>Varsity Roster</h2>
<table id="rosterTable">
  <thead>
    <tr>
      <th>Starter</th>
      <th>Name</th>
      <th>Position</th>
      <th>Grade</th>
      <th>Rating</th>
      <th>Potential</th>
      <th>PPG</th>
      <th>RPG</th>
      <th>APG</th>
      <th>GP</th>
      <th>Status</th>
    </tr>
  </thead>
  <tbody id="rosterTableBody">
  </tbody>
</table>

<h2>Recruits</h2>
<table id="recruitTable">
  <thead>
    <tr>
      <th>Name</th>
      <th>Position</th>
      <th>Grade</th>
      <th>Rating</th>
      <th>Potential</th>
      <th>Action</th>
    </tr>
  </thead>
  <tbody id="recruitTableBody">
  </tbody>
</table>

<h2>Game Log</h2>
<div id="log"></div>

<script>
  // --- Globals ---
  let varsityRoster = [];
  let recruits = [];
  let seasonNumber = 1;
  let teamWins = 0;
  let teamLosses = 0;
  let gamesPlayed = 0;
  const seasonLength = 10;

  // Positions
  const positions = ['PG', 'SG', 'SF', 'PF', 'C'];

  // DOM Elements
  const rosterTableBody = document.getElementById('rosterTableBody');
  const recruitTableBody = document.getElementById('recruitTableBody');
  const simulateGameBtn = document.getElementById('simulateGameBtn');
  const nextSeasonBtn = document.getElementById('nextSeasonBtn');
  const seasonNumberSpan = document.getElementById('seasonNumber');
  const recordSpan = document.getElementById('record');
  const logDiv = document.getElementById('log');
  const gameNumberSpan = document.getElementById('gameNumber');

  // --- Utility functions ---
  function randInt(min, max) {
    return Math.floor(Math.random() * (max - min + 1)) + min;
  }

  function toLetterGrade(score) {
    if (score >= 90) return 'A';
    else if (score >= 80) return 'B';
    else if (score >= 70) return 'C';
    else if (score >= 60) return 'D';
    else return 'F';
  }

  function gradeToText(grade) {
    switch(grade) {
      case 1: return 'Freshman';
      case 2: return 'Sophomore';
      case 3: return 'Junior';
      case 4: return 'Senior';
      default: return 'Unknown';
    }
  }

  const firstNames = ["Alex", "Jordan", "Taylor", "Morgan", "Casey", "Drew", "Cameron", "Ryan", "Jamie", "Skyler"];
  const lastNames = ["Smith", "Johnson", "Williams", "Brown", "Jones", "Davis", "Miller", "Wilson", "Moore", "Taylor"];

  function generateRandomName() {
    const first = firstNames[randInt(0, firstNames.length - 1)];
    const last = lastNames[randInt(0, lastNames.length - 1)];
    return first + " " + last;
  }

  function randomPosition() {
    return positions[randInt(0, positions.length - 1)];
  }

  function createPlayer(name, rating, potential, grade=1, position=null) {
    return {
      name,
      rating,
      potential,
      grade,
      position: position || randomPosition(),
      injured: false,
      dev: 1 + Math.random() * 0.5,
      stats: { ppg: 0, rpg: 0, apg: 0 },
      gamesPlayed: 0,
      seasonsPlayed: 0,
      ratingStartOfSeason: rating,
    };
  }

  function logMessage(msg) {
    const time = new Date().toLocaleTimeString();
    logDiv.innerHTML = `[${time}] ${msg}\n` + logDiv.innerHTML;
  }

  // --- Initialize roster ---
  function initializeRoster() {
    varsityRoster = [];
    for(let i = 0; i < 5; i++) {
      let grade = randInt(1, 3);
      let rating = randInt(60, 90);
      let potential = Math.min(100, rating + randInt(5, 20));
      let player = createPlayer(generateRandomName(), rating, potential, grade, positions[i]);
      varsityRoster.push(player);
    }
    for(let i = 5; i < 10; i++) {
      let grade = randInt(1, 3);
      let rating = randInt(60, 90);
      let potential = Math.min(100, rating + randInt(5, 20));
      let player = createPlayer(generateRandomName(), rating, potential, grade);
      varsityRoster.push(player);
    }
  }

  // --- Generate recruits ---
  function generateRecruits() {
    recruits = [];
    const numRecruits = randInt(5, 8);
    for(let i = 0; i < numRecruits; i++) {
      let rating = randInt(50, 85);
      let potential = Math.min(100, rating + randInt(5, 20));
      let grade = randInt(1, 2);
      recruits.push(createPlayer(generateRandomName(), rating, potential, grade));
    }
    updateRecruitsUI();
    logMessage(`Generated ${numRecruits} new recruits.`);
  }

  // --- Update Roster UI ---
  function updateRosterUI() {
    rosterTableBody.innerHTML = '';
    varsityRoster.forEach((p, idx) => {
      const tr = document.createElement('tr');

      let status = p.injured ? 'Injured' : 'Active';
      let starterCell = idx < 5 ? positions[idx] : '';

      tr.className = idx < 5 ? 'starter' : '';

      tr.innerHTML = `
        <td>${starterCell}</td>
        <td>${p.name}</td>
        <td>${p.position}</td>
        <td>${gradeToText(p.grade)}</td>
        <td>${toLetterGrade(p.rating)}</td>
        <td>${toLetterGrade(p.potential)}</td>
        <td>${p.stats.ppg.toFixed(1)}</td>
        <td>${p.stats.rpg.toFixed(1)}</td>
        <td>${p.stats.apg.toFixed(1)}</td>
        <td>${p.gamesPlayed}</td>
        <td class="${p.injured ? 'injured' : ''}">${status}</td>
      `;
      rosterTableBody.appendChild(tr);
    });

    recordSpan.textContent = `${teamWins} - ${teamLosses}`;
    seasonNumberSpan.textContent = seasonNumber;
    gameNumberSpan.textContent = gamesPlayed;
  }

  // --- Update Recruits UI ---
  function updateRecruitsUI() {
    recruitTableBody.innerHTML = '';
    recruits.forEach((p, idx) => {
      const tr = document.createElement('tr');
      tr.innerHTML = `
        <td>${p.name}</td>
        <td>${p.position}</td>
        <td>${gradeToText(p.grade)}</td>
        <td>${toLetterGrade(p.rating)}</td>
        <td>${toLetterGrade(p.potential)}</td>
        <td><button data-index="${idx}">Recruit</button></td>
      `;
      recruitTableBody.appendChild(tr);
    });
    // Add recruit button listeners
    document.querySelectorAll('#recruitTableBody button').forEach(button => {
      button.onclick = e => {
        const idx = parseInt(e.target.getAttribute('data-index'));
        recruitPlayer(idx);
      };
    });
  }

  // --- Recruit a player ---
  function recruitPlayer(idx) {
    if (idx < 0 || idx >= recruits.length) return;
    const player = recruits.splice(idx, 1)[0];
    player.grade = 1;  // Freshman on team
    varsityRoster.push(player);
    logMessage(`Recruited ${player.name} (${player.position})`);
    updateRosterUI();
    updateRecruitsUI();
  }

  // --- Simulate a Game ---
  function simulateGame() {
    simulateGameBtn.disabled = true;

    let teamScore = 0;
    let opponentScore = randInt(40, 80);
    let standoutPerformances = [];

    // Player contribution based on rating, position, and randomness
    varsityRoster.forEach(player => {
      if(player.injured) return; // Injured players don't play

      // Base stats weighted by position and rating
      let basePPG = 0, baseRPG = 0, baseAPG = 0;

      switch(player.position) {
        case 'PG':
          basePPG = player.rating * 0.35;
          baseRPG = player.rating * 0.10;
          baseAPG = player.rating * 0.40;
          break;
        case 'SG':
          basePPG = player.rating * 0.40;
          baseRPG = player.rating * 0.15;
          baseAPG = player.rating * 0.20;
          break;
        case 'SF':
          basePPG = player.rating * 0.35;
          baseRPG = player.rating * 0.25;
          baseAPG = player.rating * 0.15;
          break;
        case 'PF':
          basePPG = player.rating * 0.30;
          baseRPG = player.rating * 0.35;
          baseAPG = player.rating * 0.10;
          break;
        case 'C':
          basePPG = player.rating * 0.25;
          baseRPG = player.rating * 0.45;
          baseAPG = player.rating * 0.05;
          break;
      }

      // Random variation (±15%)
      const variance = 0.15;
      let ppg = basePPG * (1 + (Math.random() * 2 - 1) * variance);
      let rpg = baseRPG * (1 + (Math.random() * 2 - 1) * variance);
      let apg = baseAPG * (1 + (Math.random() * 2 - 1) * variance);

      // Clamp to zero minimum
      ppg = Math.max(0, ppg);
      rpg = Math.max(0, rpg);
      apg = Math.max(0, apg);

      // Convert to per game averages over the season including this game
      player.stats.ppg = (player.stats.ppg * player.gamesPlayed + ppg) / (player.gamesPlayed + 1);
      player.stats.rpg = (player.stats.rpg * player.gamesPlayed + rpg) / (player.gamesPlayed + 1);
      player.stats.apg = (player.stats.apg * player.gamesPlayed + apg) / (player.gamesPlayed + 1);

      player.gamesPlayed++;
      teamScore += ppg;

      // Check for standout performance
      if(ppg >= 25) standoutPerformances.push(`${player.name} scored an impressive ${ppg.toFixed(1)} points!`);
      if(rpg >= 15) standoutPerformances.push(`${player.name} dominated the boards with ${rpg.toFixed(1)} rebounds!`);
      if(apg >= 10) standoutPerformances.push(`${player.name} dazzled with ${apg.toFixed(1)} assists!`);

      // Small chance player gets injured (3%)
      if (!player.injured && Math.random() < 0.03) {
        player.injured = true;
        logMessage(`Oh no! ${player.name} got injured during the game.`);
      }
    });

    teamScore = Math.round(teamScore);

    logMessage(`Game Result: Your Team ${teamScore} - Opponent ${opponentScore}`);

    standoutPerformances.forEach(p => logMessage(p));

    if (teamScore > opponentScore) {
      teamWins++;
      logMessage("You won the game! 🏆");
    } else {
      teamLosses++;
      logMessage("You lost the game. Keep practicing!");
    }

    gamesPlayed++;
    updateRosterUI();

    // Enable next season button after season ends
    if (gamesPlayed >= seasonLength) {
      simulateGameBtn.disabled = true;
      nextSeasonBtn.disabled = false;
      logMessage("Season complete! Click 'Next Season' to progress.");

      // Show end-of-season awards
      displaySeasonAwards();
    } else {
      simulateGameBtn.disabled = false;
    }
  }

  // --- Display End-of-Season Awards ---
  function displaySeasonAwards() {
    if (varsityRoster.length === 0) return;

    // Filter players with games played
    let eligiblePlayers = varsityRoster.filter(p => p.gamesPlayed > 0);

    // MVP (highest PPG)
    let mvp = eligiblePlayers.reduce((a,b) => a.stats.ppg > b.stats.ppg ? a : b);

    // Best Rebounder (highest RPG)
    let rebounder = eligiblePlayers.reduce((a,b) => a.stats.rpg > b.stats.rpg ? a : b);

    // Best Playmaker (highest APG)
    let playmaker = eligiblePlayers.reduce((a,b) => a.stats.apg > b.stats.apg ? a : b);

    // Most Improved (biggest rating increase from start of season)
    let mostImproved = eligiblePlayers.reduce((a,b) => 
      (b.rating - b.ratingStartOfSeason) > (a.rating - a.ratingStartOfSeason) ? b : a
    );

    logMessage("\n🏅 End of Season Awards 🏅");
    logMessage(`MVP: ${mvp.name} (${toLetterGrade(mvp.rating)}) - ${mvp.stats.ppg.toFixed(1)} PPG`);
    logMessage(`Best Rebounder: ${rebounder.name} - ${rebounder.stats.rpg.toFixed(1)} RPG`);
    logMessage(`Best Playmaker: ${playmaker.name} - ${playmaker.stats.apg.toFixed(1)} APG`);
    logMessage(`Most Improved Player: ${mostImproved.name} (+${(mostImproved.rating - mostImproved.ratingStartOfSeason).toFixed(1)} rating points)`);
  }

  // --- Next Season ---
  function nextSeason() {
    seasonNumber++;
    logMessage(`\nSeason ${seasonNumber} starts!`);

    // Progress players and graduate seniors
    varsityRoster.forEach(player => {
      player.seasonsPlayed++;
      player.grade++;
    });

    // Remove graduated players
    varsityRoster = varsityRoster.filter(p => p.grade <= 4);

    // Recover injured players randomly (50%)
    varsityRoster.forEach(p => {
      if (p.injured && Math.random() < 0.5) {
        p.injured = false;
        logMessage(`${p.name} recovered from injury.`);
      }
    });

    // Improve ratings for remaining players
    varsityRoster.forEach(p => {
      let growth = (p.potential - p.rating) * 0.1 * p.dev;
      p.rating = Math.min(100, p.rating + growth);
      p.ratingStartOfSeason = p.rating; // reset for tracking improvements next season
    });

    // Generate new recruits for this season
    generateRecruits();

    // Reset season stats & counters
    varsityRoster.forEach(p => {
      p.stats = { ppg: 0, rpg: 0, apg: 0 };
      p.gamesPlayed = 0;
    });
    teamWins = 0;
    teamLosses = 0;
    gamesPlayed = 0;

    simulateGameBtn.disabled = false;
    nextSeasonBtn.disabled = true;

    updateRosterUI();
  }

  // --- Initialize game ---
  initializeRoster();
  generateRecruits();
  updateRosterUI();

  // --- Event listeners ---
  simulateGameBtn.addEventListener('click', simulateGame);
  nextSeasonBtn.addEventListener('click', nextSeason);

</script>

</body>
</html>
