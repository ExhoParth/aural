<script>
  import { onMount, onDestroy } from 'svelte';
  import { Mic, MicOff, Play, Square, SkipBack, SkipForward, X, Volume2, Sliders } from 'lucide-svelte';

  export let data;

  let isListening = false;
  let transcript = '';
  let recognition;
  let audioVisualization = null;
  let audioContext;
  let analyser;
  let dataArray;
  let canvasContext;
  let animationFrame;
  let recordingTime = 0;
  let recordingInterval;

  onMount(() => {
    setupSpeechRecognition();
    setupAudioVisualization();
  });

  onDestroy(() => {
    if (recognition) {
      recognition.stop();
    }
    if (audioContext) {
      audioContext.close();
    }
    if (animationFrame) {
      cancelAnimationFrame(animationFrame);
    }
    if (recordingInterval) {
      clearInterval(recordingInterval);
    }
  });

  function setupSpeechRecognition() {
    if ('webkitSpeechRecognition' in window) {
      recognition = new webkitSpeechRecognition();
      recognition.continuous = true;
      recognition.interimResults = true;

      recognition.onresult = (event) => {
        const current = event.resultIndex;
        const result = event.results[current];
        const transcriptResult = result[0].transcript;
        
        if (result.isFinal) {
          transcript += transcriptResult + ' ';
        }
      };

      recognition.onerror = (event) => {
        console.error('Speech recognition error', event.error);
        isListening = false;
        stopRecordingTimer();
      };
    } else {
      console.error('Speech recognition not supported');
    }
  }

  function setupAudioVisualization() {
    const canvas = document.getElementById('audio-visualization');
    if (canvas) {
      canvasContext = canvas.getContext('2d');
      
      try {
        audioContext = new (window.AudioContext || window.webkitAudioContext)();
        analyser = audioContext.createAnalyser();
        analyser.fftSize = 256;
        const bufferLength = analyser.frequencyBinCount;
        dataArray = new Uint8Array(bufferLength);
        
        // For demo purposes, we'll simulate audio data
        if (!isListening) {
          simulateAudioVisualization();
        }
      } catch (e) {
        console.error('Web Audio API is not supported in this browser', e);
      }
    }
  }

  function simulateAudioVisualization() {
    if (!canvasContext || !dataArray) return;
    
    const draw = () => {
      if (!isListening) {
        // Generate random data for visualization when not recording
        for (let i = 0; i < dataArray.length; i++) {
          dataArray[i] = Math.random() * 10; // Very low values when not recording
        }
      } else {
        // Generate more active data when recording
        for (let i = 0; i < dataArray.length; i++) {
          dataArray[i] = Math.random() * 100 + 50;
        }
      }
      
      drawVisualization();
      animationFrame = requestAnimationFrame(draw);
    };
    
    draw();
  }

  function drawVisualization() {
    if (!canvasContext || !dataArray) return;
    
    const canvas = canvasContext.canvas;
    const width = canvas.width;
    const height = canvas.height;
    
    canvasContext.clearRect(0, 0, width, height);
    
    const barWidth = width / dataArray.length;
    let x = 0;
    
    for (let i = 0; i < dataArray.length; i++) {
      const barHeight = isListening ? (dataArray[i] / 255) * height : (dataArray[i] / 255) * (height / 5);
      
      canvasContext.fillStyle = isListening ? '#4285F4' : '#D0D0D0';
      canvasContext.fillRect(x, height - barHeight, barWidth - 1, barHeight);
      
      x += barWidth;
    }
  }

  function toggleListening() {
    if (isListening) {
      recognition.stop();
      stopRecordingTimer();
    } else {
      transcript = '';
      recognition.start();
      startRecordingTimer();
    }
    isListening = !isListening;
  }

  function startRecordingTimer() {
    recordingTime = 0;
    recordingInterval = setInterval(() => {
      recordingTime += 0.1;
    }, 100);
  }

  function stopRecordingTimer() {
    if (recordingInterval) {
      clearInterval(recordingInterval);
    }
  }

  function formatTime(time) {
    const minutes = Math.floor(time / 60);
    const seconds = Math.floor(time % 60);
    const tenths = Math.floor((time * 10) % 10);
    return `${minutes}:${seconds.toString().padStart(2, '0')}.${tenths}`;
  }
</script>

<div class="audio-input-node">
  <!-- Timeline with markers -->
  <div class="timeline">
    <div class="time-markers">
      <span>-30</span>
      <span>-15</span>
      <span>0</span>
      <span>15</span>
      <span>30</span>
      <span>45</span>
    </div>
    <div class="timeline-ruler"></div>
  </div>
  
  <!-- Control panel -->
  <div class="control-panel">
    <!-- Media controls -->
    <div class="media-controls">
      <button class="control-button">
        <SkipBack size={16} />
      </button>
      <button class="control-button">
        {#if isListening}
          <Square size={16} />
        {:else}
          <Play size={16} />
        {/if}
      </button>
      <button class="control-button">
        <SkipForward size={16} />
      </button>
      <button class="control-button record-button" class:recording={isListening} on:click={toggleListening}>
        <Mic size={16} />
      </button>
    </div>
    
    <!-- Audio visualization -->
    <div class="visualization-container">
      <div class="track-info" class:recording={isListening}>
        {isListening ? 'RECORDING' : 'READY'} {formatTime(recordingTime)}
      </div>
      <canvas id="audio-visualization" width="300" height="80"></canvas>
    </div>
    
    <!-- Audio controls -->
    <div class="audio-controls">
      <div class="control-group">
        <button class="small-button">
          <X size={12} />
        </button>
        <button class="small-button">
          <Volume2 size={12} />
        </button>
        <button class="small-button">
          SOLO
        </button>
      </div>
      <div class="control-group">
        <button class="small-button">
          EFFECTS
        </button>
      </div>
      <div class="control-group knobs">
        <div class="knob-container">
          <div class="knob">
            <div class="knob-indicator"></div>
          </div>
          <span>L</span>
        </div>
        <div class="knob-container">
          <div class="knob">
            <div class="knob-indicator"></div>
          </div>
          <span>R</span>
        </div>
      </div>
    </div>
  </div>
  
  <!-- Transcript area -->
  <div class="transcript-area">
    <h4>Transcript</h4>
    <p class="transcript">{transcript || 'No transcript available. Click the record button to start.'}</p>
  </div>
</div>

<style>
  .audio-input-node {
    background: #f5f5f7;
    border-radius: 12px;
    box-shadow: 0 8px 30px rgba(0, 0, 0, 0.08);
    padding: 20px;
    width: 400px;
    font-family: 'Inter', 'Segoe UI', sans-serif;
    color: #333;
  }

  .timeline {
    margin-bottom: 15px;
  }

  .time-markers {
    display: flex;
    justify-content: space-between;
    font-size: 10px;
    color: #666;
    margin-bottom: 5px;
  }

  .timeline-ruler {
    height: 2px;
    background: #ddd;
    position: relative;
  }

  .timeline-ruler::before {
    content: '';
    position: absolute;
    height: 10px;
    width: 2px;
    background: #666;
    left: 50%;
    top: -4px;
  }

  .control-panel {
    display: flex;
    gap: 15px;
    margin-bottom: 15px;
  }

  .media-controls {
    display: flex;
    flex-direction: column;
    gap: 8px;
  }

  .control-button {
    width: 40px;
    height: 40px;
    border-radius: 8px;
    background: white;
    border: none;
    display: flex;
    align-items: center;
    justify-content: center;
    cursor: pointer;
    box-shadow: 0 2px 5px rgba(0, 0, 0, 0.05), 0 0 0 1px rgba(0, 0, 0, 0.03);
    transition: all 0.2s ease;
  }

  .control-button:hover {
    transform: translateY(-2px);
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1), 0 0 0 1px rgba(0, 0, 0, 0.03);
  }

  .record-button {
    color: #ff3b30;
  }

  .record-button.recording {
    background: #ff3b30;
    color: white;
    animation: pulse 2s infinite;
  }

  @keyframes pulse {
    0% { box-shadow: 0 0 0 0 rgba(255, 59, 48, 0.4); }
    70% { box-shadow: 0 0 0 10px rgba(255, 59, 48, 0); }
    100% { box-shadow: 0 0 0 0 rgba(255, 59, 48, 0); }
  }

  .visualization-container {
    flex-grow: 1;
    background: white;
    border-radius: 8px;
    overflow: hidden;
    box-shadow: 0 2px 5px rgba(0, 0, 0, 0.05), 0 0 0 1px rgba(0, 0, 0, 0.03);
    display: flex;
    flex-direction: column;
  }

  .track-info {
    padding: 5px 10px;
    font-size: 12px;
    font-weight: 600;
    background: #f0f0f0;
    color: #666;
  }

  .track-info.recording {
    background: #4285F4;
    color: white;
  }

  #audio-visualization {
    width: 100%;
    height: 100%;
    background: rgba(240, 240, 240, 0.5);
  }

  .audio-controls {
    display: flex;
    flex-direction: column;
    gap: 8px;
  }

  .control-group {
    display: flex;
    gap: 5px;
  }

  .small-button {
    padding: 5px 8px;
    font-size: 10px;
    background: white;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    box-shadow: 0 1px 3px rgba(0, 0, 0, 0.05), 0 0 0 1px rgba(0, 0, 0, 0.03);
    display: flex;
    align-items: center;
    justify-content: center;
    font-weight: 600;
  }

  .small-button:hover {
    background: #f5f5f5;
  }

  .knobs {
    display: flex;
    justify-content: space-around;
    margin-top: 5px;
  }

  .knob-container {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 2px;
  }

  .knob {
    width: 24px;
    height: 24px;
    border-radius: 50%;
    background: white;
    box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1), 0 0 0 1px rgba(0, 0, 0, 0.05);
    position: relative;
  }

  .knob-indicator {
    position: absolute;
    width: 2px;
    height: 8px;
    background: #4285F4;
    top: 3px;
    left: 50%;
    transform: translateX(-50%);
    border-radius: 1px;
  }

  .knob-container span {
    font-size: 10px;
    color: #666;
  }

  .transcript-area {
    background: white;
    border-radius: 8px;
    padding: 15px;
    box-shadow: 0 2px 5px rgba(0, 0, 0, 0.05), 0 0 0 1px rgba(0, 0, 0, 0.03);
  }

  h4 {
    margin: 0 0 10px 0;
    font-size: 14px;
    color: #333;
  }

  .transcript {
    margin: 0;
    color: #555;
    line-height: 1.5;
    font-size: 13px;
    max-height: 100px;
    overflow-y: auto;
  }

  @media (max-width: 500px) {
    .audio-input-node {
      width: 100%;
      padding: 15px;
    }
    
    .control-panel {
      flex-direction: column;
    }
    
    .media-controls {
      flex-direction: row;
    }
  }
</style>