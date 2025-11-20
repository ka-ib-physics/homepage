---
layout: page
title: Extra
description: Extra Material.
---

## Pos - Time Graph Widget

<div class="position-velocity-sim">
  <h2>Position–Time to Velocity–Time Simulation</h2>
  <p>Draw a position–time graph in the top box (time on the x-axis, position on the y-axis). Then click <strong>Compute velocity</strong> to see the velocity–time graph below.</p>

  <div style="display:flex; flex-direction:column; gap:1rem; max-width:800px;">
    <!-- Position canvas -->
    <div>
      <h3>Position vs Time</h3>
      <canvas id="posCanvas" width="800" height="300" style="border:1px solid #ccc; touch-action:none;"></canvas>
      <p style="font-size:0.9rem; color:#555;">
        Hint: time increases to the right, position increases upward. Click and drag to draw.
      </p>
    </div>

    <!-- Controls -->
    <div style="display:flex; gap:0.5rem; align-items:center;">
      <button id="computeBtn">Compute velocity</button>
      <button id="resetBtn">Reset</button>
      <span id="statusMsg" style="font-size:0.9rem; color:#555;"></span>
    </div>

    <!-- Velocity canvas -->
    <div>
      <h3>Velocity vs Time</h3>
      <canvas id="velCanvas" width="800" height="300" style="border:1px solid #ccc;"></canvas>
      <p style="font-size:0.9rem; color:#555;">
        The velocity graph is computed as the slope (derivative) of your position graph.
      </p>
    </div>
  </div>
</div>

<script src="{{ '/assets/js/posvel.js' | relative_url }}"></script>
