---
layout: page
title: Extra
description: Extra Material.
---


## Pos - Time Graph Widget

```html
<script>
document.addEventListener('DOMContentLoaded', function () {
  // ====== Simple Position->Velocity Simulation (JavaScript) ======

  // Canvas setup
  const posCanvas = document.getElementById('posCanvas');
  const velCanvas = document.getElementById('velCanvas');
  const posCtx = posCanvas.getContext('2d');
  const velCtx = velCanvas.getContext('2d');

  const computeBtn = document.getElementById('computeBtn');
  const resetBtn = document.getElementById('resetBtn');
  const statusMsg = document.getElementById('statusMsg');

  let drawing = false;
  let posPoints = []; // {x: , y: } in canvas coordinates

  // ---- Utility: draw axes on a canvas ----
  function drawAxes(ctx, width, height, xLabel, yLabel) {
    ctx.clearRect(0, 0, width, height);
    ctx.save();
    ctx.strokeStyle = '#aaa';
    ctx.lineWidth = 1;

    // t-axis (x-axis) along bottom
    ctx.beginPath();
    ctx.moveTo(40, height - 30);
    ctx.lineTo(width - 10, height - 30);
    ctx.stroke();

    // y-axis on left
    ctx.beginPath();
    ctx.moveTo(40, height - 30);
    ctx.lineTo(40, 10);
    ctx.stroke();

    // Labels
    ctx.fillStyle = '#555';
    ctx.font = '14px sans-serif';
    ctx.fillText(xLabel, width - 60, height - 10);
    ctx.save();
    ctx.translate(15, 20);
    ctx.rotate(-Math.PI / 2);
    ctx.fillText(yLabel, 0, 0);
    ctx.restore();

    ctx.restore();
  }

  // Initial axes
  drawAxes(posCtx, posCanvas.width, posCanvas.height, 'time', 'position');
  drawAxes(velCtx, velCanvas.width, velCanvas.height, 'time', 'velocity');

  // ---- Drawing on position canvas ----
  function getCanvasPos(event, canvas) {
    const rect = canvas.getBoundingClientRect();
    const clientX = event.touches ? event.touches[0].clientX : event.clientX;
    const clientY = event.touches ? event.touches[0].clientY : event.clientY;
    return {
      x: clientX - rect.left,
      y: clientY - rect.top
    };
  }

  function startDraw(e) {
    drawing = true;
    posPoints = []; // start a new curve
    statusMsg.textContent = '';
    const p = getCanvasPos(e, posCanvas);
    posPoints.push(p);

    // redraw axes and start path
    drawAxes(posCtx, posCanvas.width, posCanvas.height, 'time', 'position');
    posCtx.strokeStyle = '#0074D9';
    posCtx.lineWidth = 2;
    posCtx.beginPath();
    posCtx.moveTo(p.x, p.y);
  }

  function moveDraw(e) {
    if (!drawing) return;
    e.preventDefault(); // avoid scrolling on touch
    const p = getCanvasPos(e, posCanvas);
    posPoints.push(p);
    posCtx.lineTo(p.x, p.y);
    posCtx.stroke();
  }

  function endDraw() {
    if (!drawing) return;
    drawing = false;
    posCtx.closePath();
  }

  // Mouse events
  posCanvas.addEventListener('mousedown', startDraw);
  posCanvas.addEventListener('mousemove', moveDraw);
  posCanvas.addEventListener('mouseup', endDraw);
  posCanvas.addEventListener('mouseleave', endDraw);

  // Touch events (for tablets/phones)
  posCanvas.addEventListener('touchstart', startDraw);
  posCanvas.addEventListener('touchmove', moveDraw);
  posCanvas.addEventListener('touchend', endDraw);

  // ---- Compute velocity from drawn curve ----
  function computeVelocity() {
    if (posPoints.length < 3) {
      statusMsg.textContent = 'Please draw a curve first (at least a few points).';
      return;
    }

    // Sort points by x (time) to ensure increasing time
    const pts = [...posPoints].sort((a, b) => a.x - b.x);

    // Map canvas x,y to normalized time and position
    // We'll treat x from 40 to (width-10) as time range [0, 1]
    // y from (height-30) at bottom to 10 at top as position range [0, 1]
    const w = posCanvas.width;
    const h = posCanvas.height;
    const xMin = 40, xMax = w - 10;
    const yBottom = h - 30, yTop = 10;

    const normPoints = pts
      .filter(p => p.x >= xMin && p.x <= xMax && p.y <= yBottom && p.y >= yTop)
      .map(p => {
        const t = (p.x - xMin) / (xMax - xMin); // 0..1
        const xPos = 1 - (p.y - yTop) / (yBottom - yTop); // 0..1, higher is up
        return { t, xPos };
      });

    if (normPoints.length < 3) {
      statusMsg.textContent = 'Curve too small or outside axes area.';
      return;
    }

    // Compute numerical derivative using central differences
    const velPoints = [];
    for (let i = 1; i < normPoints.length - 1; i++) {
      const pPrev = normPoints[i - 1];
      const pNext = normPoints[i + 1];
      const dt = pNext.t - pPrev.t;
      if (dt === 0) continue;
      const dx = pNext.xPos - pPrev.xPos;
      const v = dx / dt;
      velPoints.push({ t: normPoints[i].t, v });
    }

    if (velPoints.length < 2) {
      statusMsg.textContent = 'Not enough data to compute velocity.';
      return;
    }

    // Determine velocity range for plotting
    let vMin = Infinity, vMax = -Infinity;
    velPoints.forEach(p => {
      if (p.v < vMin) vMin = p.v;
      if (p.v > vMax) vMax = p.v;
    });
    if (vMin === vMax) {
      // Expand a bit so it's visible
      vMin -= 1;
      vMax += 1;
    }

    // Draw velocity axes
    drawAxes(velCtx, velCanvas.width, velCanvas.height, 'time', 'velocity');

    // Plot velocity curve on velocity canvas
    const vw = velCanvas.width;
    const vh = velCanvas.height;
    const vxMin = 40, vxMax = vw - 10;
    const vyBottom = vh - 30, vyTop = 10;

    velCtx.strokeStyle = '#FF4136';
    velCtx.lineWidth = 2;
    velCtx.beginPath();
    velPoints.forEach((p, index) => {
      const cx = vxMin + p.t * (vxMax - vxMin);
      const normV = (p.v - vMin) / (vMax - vMin); // 0..1
      const cy = vyBottom - normV * (vyBottom - vyTop);
      if (index === 0) {
        velCtx.moveTo(cx, cy);
      } else {
        velCtx.lineTo(cx, cy);
      }
    });
    velCtx.stroke();

    statusMsg.textContent = 'Velocity computed from the slope of your position curve.';
  }

  // ---- Reset everything ----
  function resetAll() {
    posPoints = [];
    drawAxes(posCtx, posCanvas.width, posCanvas.height, 'time', 'position');
    drawAxes(velCtx, velCanvas.width, velCanvas.height, 'time', 'velocity');
    statusMsg.textContent = '';
  }

  computeBtn.addEventListener('click', computeVelocity);
  resetBtn.addEventListener('click', resetAll);
});
</script>


