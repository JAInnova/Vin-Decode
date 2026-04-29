[vin-scanner-widget index.html](https://github.com/user-attachments/files/27205773/vin-scanner-widget.html)
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no">
<title>VIN Barcode Scanner</title>

<!-- Jotform Custom Widget API -->
<script src="//js.jotform.com/JotFormCustomWidget.min.js"></script>

<!-- ZXing barcode reader (UMD bundle exposes window.ZXing) -->
<script src="https://unpkg.com/@zxing/library@0.21.3/umd/index.min.js"></script>

<style>
  *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

  :root {
    --orange: #ff5b00;
    --orange-dark: #c44400;
    --ink: #1a1d24;
    --ink-soft: #4a5160;
    --paper: #f5f6f8;
    --paper-dark: #e8eaef;
    --line: #d6d9e0;
    --green: #16a34a;
    --green-bg: #ecfdf5;
    --red: #dc2626;
    --red-bg: #fef2f2;
    --warn: #ca8a04;
    --warn-bg: #fefce8;
  }

  body {
    font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", "Helvetica Neue", sans-serif;
    color: var(--ink);
    background: transparent;
    padding: 0;
    line-height: 1.4;
    -webkit-font-smoothing: antialiased;
  }

  .widget {
    width: 100%;
    max-width: 520px;
    margin: 0 auto;
  }

  /* Scanner viewport */
  .scanner {
    position: relative;
    width: 100%;
    aspect-ratio: 4 / 3;
    background: #000;
    border-radius: 10px;
    overflow: hidden;
    display: none;
    box-shadow: 0 2px 8px rgba(0,0,0,0.08);
  }
  .scanner.active { display: block; }
  .scanner video {
    width: 100%;
    height: 100%;
    object-fit: cover;
    display: block;
  }

  /* Reticle overlay */
  .reticle {
    position: absolute;
    inset: 0;
    pointer-events: none;
  }
  .reticle::before {
    content: "";
    position: absolute;
    top: 50%; left: 50%;
    transform: translate(-50%, -50%);
    width: 86%;
    height: 28%;
    border: 2px solid var(--orange);
    border-radius: 4px;
    box-shadow: 0 0 0 9999px rgba(0,0,0,0.4);
  }
  .reticle-hint {
    position: absolute;
    bottom: 10px; left: 0; right: 0;
    text-align: center;
    color: #fff;
    font-size: 12px;
    letter-spacing: 0.3px;
    text-shadow: 0 1px 3px rgba(0,0,0,0.7);
  }
  .scan-line {
    position: absolute;
    top: 50%; left: 7%; right: 7%;
    height: 2px;
    background: var(--orange);
    box-shadow: 0 0 8px var(--orange);
    transform: translateY(-50%);
    animation: scan 1.6s ease-in-out infinite;
  }
  @keyframes scan {
    0%, 100% { transform: translateY(-90%); opacity: 0.4; }
    50% { transform: translateY(90%); opacity: 1; }
  }

  /* Buttons */
  .controls {
    display: flex;
    gap: 8px;
    margin-top: 10px;
  }
  button {
    flex: 1;
    padding: 14px 16px;
    border: 0;
    border-radius: 8px;
    font: inherit;
    font-size: 15px;
    font-weight: 600;
    cursor: pointer;
    transition: transform 0.05s, background 0.15s, opacity 0.15s;
    -webkit-tap-highlight-color: transparent;
  }
  button:active { transform: translateY(1px); }
  button:disabled { opacity: 0.4; cursor: not-allowed; }

  .btn-primary {
    background: var(--orange);
    color: #fff;
  }
  .btn-primary:hover { background: var(--orange-dark); }

  .btn-secondary {
    background: var(--paper-dark);
    color: var(--ink);
  }
  .btn-secondary:hover { background: var(--line); }

  .btn-icon {
    flex: 0 0 auto;
    width: 48px;
    padding: 14px 0;
  }

  /* Result card */
  .result {
    margin-top: 12px;
    padding: 12px 14px;
    background: var(--paper);
    border-radius: 8px;
    border-left: 4px solid var(--line);
    display: none;
  }
  .result.visible { display: block; }
  .result.success {
    background: var(--green-bg);
    border-left-color: var(--green);
  }
  .result.error {
    background: var(--red-bg);
    border-left-color: var(--red);
  }
  .result-row {
    display: flex;
    justify-content: space-between;
    align-items: baseline;
    gap: 12px;
  }
  .result-label {
    font-size: 10px;
    text-transform: uppercase;
    letter-spacing: 0.8px;
    color: var(--ink-soft);
    font-weight: 600;
  }
  .result-status {
    font-size: 11px;
    font-weight: 600;
  }
  .result.success .result-status { color: var(--green); }
  .result.error .result-status { color: var(--red); }
  .result-value {
    font-family: ui-monospace, "SF Mono", "Cascadia Mono", Menlo, Consolas, monospace;
    font-size: 17px;
    font-weight: 600;
    letter-spacing: 0.5px;
    word-break: break-all;
    margin-top: 4px;
  }

  /* Manual entry */
  .manual {
    margin-top: 14px;
  }
  .manual-label {
    display: block;
    font-size: 12px;
    color: var(--ink-soft);
    margin-bottom: 6px;
    font-weight: 500;
  }
  .manual input {
    width: 100%;
    padding: 12px 14px;
    font-family: ui-monospace, "SF Mono", Menlo, monospace;
    font-size: 16px;
    font-weight: 600;
    letter-spacing: 1px;
    border: 1.5px solid var(--line);
    border-radius: 8px;
    text-transform: uppercase;
    background: #fff;
    color: var(--ink);
    transition: border-color 0.15s, box-shadow 0.15s;
  }
  .manual input:focus {
    outline: none;
    border-color: var(--orange);
    box-shadow: 0 0 0 3px rgba(255, 91, 0, 0.15);
  }

  /* Status line */
  .status {
    font-size: 13px;
    color: var(--ink-soft);
    margin-top: 10px;
    min-height: 18px;
    display: flex;
    align-items: center;
    gap: 6px;
  }
  .status.error { color: var(--red); }
  .status.success { color: var(--green); }
  .status.warn { color: var(--warn); }

  .icon {
    width: 18px;
    height: 18px;
    flex-shrink: 0;
  }

  /* Initial state - no scanner shown */
  .placeholder {
    width: 100%;
    aspect-ratio: 4 / 3;
    background: var(--paper);
    border: 2px dashed var(--line);
    border-radius: 10px;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    color: var(--ink-soft);
    gap: 10px;
    padding: 20px;
    text-align: center;
  }
  .placeholder.hidden { display: none; }
  .placeholder-icon {
    width: 48px;
    height: 48px;
    opacity: 0.5;
  }
  .placeholder-text {
    font-size: 14px;
    max-width: 280px;
  }
</style>
</head>
<body>
<div class="widget">

  <!-- Pre-scan placeholder -->
  <div class="placeholder" id="placeholder">
    <svg class="placeholder-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5">
      <rect x="3" y="6" width="18" height="12" rx="1"/>
      <path d="M7 6v12M10 6v12M13 6v12M16 6v12M19 6v12" stroke-width="1"/>
    </svg>
    <div class="placeholder-text">Tap <strong>Scan VIN</strong> to use your camera, or type the VIN manually below.</div>
  </div>

  <!-- Live scanner -->
  <div class="scanner" id="scanner">
    <video id="video" playsinline muted></video>
    <div class="reticle">
      <div class="scan-line"></div>
      <div class="reticle-hint">Center the VIN barcode in the box</div>
    </div>
  </div>

  <!-- Controls -->
  <div class="controls">
    <button class="btn-primary" id="startBtn" type="button">
      <span style="display:inline-flex;align-items:center;gap:6px;">
        <svg class="icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
          <path d="M23 19a2 2 0 0 1-2 2H3a2 2 0 0 1-2-2V8a2 2 0 0 1 2-2h4l2-3h6l2 3h4a2 2 0 0 1 2 2z"/>
          <circle cx="12" cy="13" r="4"/>
        </svg>
        Scan VIN
      </span>
    </button>
    <button class="btn-secondary" id="stopBtn" type="button" style="display:none">Cancel</button>
    <button class="btn-secondary btn-icon" id="torchBtn" type="button" style="display:none" title="Toggle flashlight">
      <svg class="icon" style="margin:0 auto" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
        <path d="M18 2L6 14h6l-2 8 12-12h-6l2-8z"/>
      </svg>
    </button>
  </div>

  <!-- Result -->
  <div class="result" id="result">
    <div class="result-row">
      <span class="result-label">Captured VIN</span>
      <span class="result-status" id="resultStatus"></span>
    </div>
    <div class="result-value" id="resultValue"></div>
  </div>

  <!-- Manual entry -->
  <div class="manual">
    <label class="manual-label" for="manualInput">Or enter VIN manually (17 characters)</label>
    <input
      type="text"
      id="manualInput"
      maxlength="17"
      autocapitalize="characters"
      autocomplete="off"
      autocorrect="off"
      spellcheck="false"
      placeholder="1HGBH41JXMN109186"
      inputmode="text"
    >
  </div>

  <!-- Status -->
  <div class="status" id="status"></div>
</div>

<script>
(function () {
  'use strict';

  // VIN spec: 17 chars, alphanumeric, but never I, O, or Q
  var VIN_REGEX = /^[A-HJ-NPR-Z0-9]{17}$/;
  var VIN_EXTRACT = /[A-HJ-NPR-Z0-9]{17}/i;

  // Elements
  var els = {
    placeholder: document.getElementById('placeholder'),
    scanner:     document.getElementById('scanner'),
    video:       document.getElementById('video'),
    startBtn:    document.getElementById('startBtn'),
    stopBtn:     document.getElementById('stopBtn'),
    torchBtn:    document.getElementById('torchBtn'),
    result:      document.getElementById('result'),
    resultValue: document.getElementById('resultValue'),
    resultStatus:document.getElementById('resultStatus'),
    manualInput: document.getElementById('manualInput'),
    status:      document.getElementById('status')
  };

  // State
  var state = {
    value: '',
    codeReader: null,
    track: null,
    torchOn: false,
    scanning: false
  };

  // ---------- Helpers ----------
  function setStatus(msg, kind) {
    els.status.textContent = msg || '';
    els.status.className = 'status' + (kind ? ' ' + kind : '');
  }

  function setValue(val, fromScan) {
    val = (val || '').toUpperCase().trim();
    state.value = val;

    if (val) {
      els.result.classList.add('visible');
      els.resultValue.textContent = val;
      var valid = VIN_REGEX.test(val);
      if (valid) {
        els.result.className = 'result visible success';
        els.resultStatus.textContent = '✓ VALID';
        setStatus('VIN captured', 'success');
      } else {
        els.result.className = 'result visible error';
        els.resultStatus.textContent = val.length === 17 ? 'INVALID CHARACTERS' : (val.length + '/17 CHARS');
        if (fromScan) {
          setStatus("Scanned but doesn't match a 17-char VIN. Try again.", 'warn');
        } else {
          setStatus('VIN must be 17 characters with no I, O, or Q.', 'warn');
        }
      }
    } else {
      els.result.classList.remove('visible');
      setStatus('');
    }

    if (els.manualInput.value !== val) els.manualInput.value = val;
  }

  // ---------- Scanner ----------
  function startScanner() {
    if (typeof ZXing === 'undefined' || !ZXing.BrowserMultiFormatReader) {
      setStatus('Scanner library failed to load — please use manual entry.', 'error');
      return;
    }

    if (!navigator.mediaDevices || !navigator.mediaDevices.getUserMedia) {
      setStatus('Camera not supported in this browser. Use manual entry.', 'error');
      return;
    }

    els.placeholder.classList.add('hidden');
    els.scanner.classList.add('active');
    els.startBtn.style.display = 'none';
    els.stopBtn.style.display = 'block';
    setStatus('Starting camera…');

    // Configure formats - VIN labels typically use Code 39 or Data Matrix.
    // Including Code 128 and QR for flexibility.
    var hints = new Map();
    hints.set(ZXing.DecodeHintType.POSSIBLE_FORMATS, [
      ZXing.BarcodeFormat.CODE_39,
      ZXing.BarcodeFormat.CODE_128,
      ZXing.BarcodeFormat.DATA_MATRIX,
      ZXing.BarcodeFormat.QR_CODE,
      ZXing.BarcodeFormat.PDF_417
    ]);
    hints.set(ZXing.DecodeHintType.TRY_HARDER, true);

    state.codeReader = new ZXing.BrowserMultiFormatReader(hints, 300);
    state.scanning = true;

    // Prefer rear camera
    var constraints = {
      video: {
        facingMode: { ideal: 'environment' },
        width: { ideal: 1280 },
        height: { ideal: 720 }
      }
    };

    state.codeReader.decodeFromConstraints(constraints, els.video, function (result, err) {
      if (!state.scanning) return;
      if (result) {
        var text = result.getText();
        var match = text.match(VIN_EXTRACT);
        var captured = match ? match[0].toUpperCase() : text.toUpperCase();
        setValue(captured, true);
        if (VIN_REGEX.test(captured)) {
          // Brief flash, then stop
          setTimeout(stopScanner, 300);
        }
      }
      // Ignore NotFoundException - that's "no barcode in frame yet"
    }).then(function () {
      // Camera started - check for torch capability
      if (els.video.srcObject) {
        var tracks = els.video.srcObject.getVideoTracks();
        if (tracks && tracks[0]) {
          state.track = tracks[0];
          var capabilities = state.track.getCapabilities ? state.track.getCapabilities() : {};
          if (capabilities.torch) {
            els.torchBtn.style.display = 'block';
          }
        }
      }
      setStatus('Point camera at VIN barcode');
    }).catch(function (e) {
      console.error('Camera error:', e);
      var msg = 'Camera unavailable';
      if (e && e.name === 'NotAllowedError') {
        msg = 'Camera permission denied. Check your browser settings.';
      } else if (e && e.name === 'NotFoundError') {
        msg = 'No camera found on this device. Use manual entry.';
      } else if (e && e.message) {
        msg = 'Camera error: ' + e.message;
      }
      setStatus(msg, 'error');
      resetScannerUI();
    });
  }

  function stopScanner() {
    state.scanning = false;
    if (state.codeReader) {
      try { state.codeReader.reset(); } catch (e) {}
      state.codeReader = null;
    }
    state.track = null;
    state.torchOn = false;
    resetScannerUI();
  }

  function resetScannerUI() {
    els.scanner.classList.remove('active');
    els.placeholder.classList.remove('hidden');
    els.startBtn.style.display = 'block';
    els.stopBtn.style.display = 'none';
    els.torchBtn.style.display = 'none';
  }

  function toggleTorch() {
    if (!state.track || !state.track.applyConstraints) return;
    state.torchOn = !state.torchOn;
    state.track.applyConstraints({
      advanced: [{ torch: state.torchOn }]
    }).catch(function (e) {
      console.warn('Torch toggle failed', e);
      state.torchOn = false;
    });
  }

  // ---------- Wire up ----------
  els.startBtn.addEventListener('click', startScanner);
  els.stopBtn.addEventListener('click', function () {
    stopScanner();
    setStatus('');
  });
  els.torchBtn.addEventListener('click', toggleTorch);

  els.manualInput.addEventListener('input', function (e) {
    setValue(e.target.value, false);
  });

  // Stop camera if user navigates away
  window.addEventListener('pagehide', stopScanner);

  // ---------- Jotform integration ----------
  if (typeof JFCustomWidget !== 'undefined') {
    JFCustomWidget.subscribe('ready', function () {
      try {
        // Preload existing value if editing a submission
        var existing = JFCustomWidget.getWidgetSetting('defaultValue');
        if (existing) setValue(existing, false);

        // Optional: read a "required" widget setting
        var req = JFCustomWidget.getWidgetSetting('required');
        if (req === 'Yes' || req === true || req === 'true') {
          els.manualInput.required = true;
        }
      } catch (e) {
        console.warn('Widget settings read failed', e);
      }
    });

    JFCustomWidget.subscribe('submit', function () {
      var val = state.value;
      var valid = VIN_REGEX.test(val);
      JFCustomWidget.sendSubmit({
        valid: valid,
        value: val
      });
    });
  }
})();
</script>
</body>
</html>
