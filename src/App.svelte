<script>
  let start = false;
  let context;
  let audio;
  let drawCanvas = false;
  let canvas;
  let canvasCtx;
  let player;
  let dataArray;
  let bufferLength;
  let analyser;
  let gainNode;
  let biquadFilter;
  let WIDTH = 500;
  let HEIGHT = 200;
  let data = [];
  let message = [];
  $: messageData = message.join("");

  const letters = {
    "00000": "\0",
    "00100": " ",
    "10111": "Q",
    "10011": "W",
    "00001": "E",
    "01010": "R",
    "10000": "T",
    "10101": "Y",
    "00111": "U",
    "00110": "I",
    "11000": "O",
    "10110": "P",
    "00011": "A",
    "00101": "S",
    "01001": "D",
    "01101": "F",
    "11010": "G",
    "10100": "H",
    "01011": "J",
    "01111": "K",
    "10010": "L",
    "10001": "Z",
    "11101": "X",
    "01110": "C",
    "11110": "V",
    "11001": "B",
    "01100": "N",
    "11100": "M",
    "01000": "\r",
    "00010": "\n"
  };

  const figures = {
    "00000": "\0",
    "00100": " ",
    "10111": "1",
    "10011": "2",
    "00001": "3",
    "01010": "4",
    "10000": "5",
    "10101": "6",
    "00111": "7",
    "00110": "8",
    "11000": "9",
    "10110": "0",
    "00011": "–",
    "00101": "Bell",
    "01001": "$",
    "01101": "!",
    "11010": "&",
    "10100": "#",
    "01011": "'",
    "01111": "(",
    "10010": ")",
    "10001": '"',
    "11101": "/",
    "01110": ":",
    "11110": ";",
    "11001": "?",
    "01100": ",",
    "11100": ".",
    "01000": "\r",
    "00010": "\n"
  };

  const figureShift = "11011";
  const letterShift = "11111";

  let fps = 50; // transmission speed is 50 Baud
  let now;
  let then = Date.now();
  let interval = 1000 / fps;
  let delta;

  // MARK is 1275 Hz, SPACE is 1445 Hz
  const MARK = 118; // i cant actually remember how i arrived at these values so they *may* be wrong
  const SPACE = 134;

  let currentLetter = [];

  function draw() {
    requestAnimationFrame(draw);

    now = Date.now();
    delta = now - then;

    if (delta > interval) {
      then = now - (delta % interval);

      analyser.getByteTimeDomainData(dataArray);
      data = [...data, dataArray[MARK] >= dataArray[SPACE]]; // probably a better way of doing this

      if (data.length > 8000) {
        data = []; // i dunno, just incase it fills up too much :D
      }

      if (data.length === 8) {
        data.pop(); // remove stop pulse
        data.pop(); // remove stop pulse
        data.shift(); // remove start pulse

        const binaryString = data.map(v => (v ? 1 : 0)).join("");
        const letter = letters[binaryString];
        message = [...message, letter];
        data = [];
      }
    }

    if (drawCanvas) {
      canvasCtx.fillStyle = "rgb(0, 0, 0)";
      canvasCtx.fillRect(0, 0, WIDTH, HEIGHT);
      const barWidth = WIDTH / 2;
      const barHeight1 = dataArray[SPACE];
      const barHeight2 = dataArray[MARK];
      canvasCtx.fillStyle = "rgb(255,0,0)";
      canvasCtx.fillRect(0, HEIGHT - barHeight1 / 2, barWidth, barHeight1);
      canvasCtx.fillRect(
        WIDTH / 2,
        HEIGHT - barHeight2 / 2,
        barWidth,
        barHeight2
      );
    }
  }

  const handleClicked = () => {
    window.AudioContext = window.AudioContext || window.webkitAudioContext;
    context = new AudioContext();
    analyser = context.createAnalyser();
    analyser.fftSize = 4096;

    let source = context.createMediaElementSource(
      document.querySelector("audio")
    );

    bufferLength = analyser.frequencyBinCount;
    console.log(bufferLength, "length");
    dataArray = new Uint8Array(bufferLength);
    analyser.getByteTimeDomainData(dataArray);

    gainNode = context.createGain();

    source.connect(analyser);

    // analyser.connect(context.destination); // <--- uncomment this if you want to hear it

    if (drawCanvas) {
      canvas = document.getElementById("thing");
      canvasCtx = canvas.getContext("2d");
    }

    player.play();
    start = true;
    draw();
  };
</script>

<style>
  .startButton {
    cursor: pointer;
  }
</style>

<main>
  {#if !start}
    <button class="startButton" on:click={handleClicked}>Start</button>
  {/if}

  <audio
    id="player"
    bind:this={player}
    src="http://internet-tty.net:8040/EUROPE"
    crossorigin="anonymous"
    controls>
    <track kind="captions" />
  </audio>
  {#if drawCanvas}
    <canvas id="thing" width="500" height="200" />
  {/if}
  <p>Message: {messageData}</p>

</main>
