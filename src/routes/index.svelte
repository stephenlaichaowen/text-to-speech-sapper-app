<script>
  import { onMount } from 'svelte'

  let speed = 1;
  let text = "Many people say they have no choice but to embrace the changes, even as they come to terms with the loss of freedom and spontaneity.";
  let textInput;
  let speedInput 
  let speedDes = 'speed'
  let playDes = 'play'
  let pauseDes = 'pause'
  let stopDes = 'stop'
  let currentCharacter;
  let lang = true

  const LOCAL_STORAGE_TEXT_TO_SPEECH_KEY = 'stephen.text_to_speech_app'

  $: if (lang) {
    speedDes = 'Speed'
    playDes = 'Play'
    pauseDes = 'Pause'
    stopDes = 'Stop'
  }

  $: if (!lang) {
    speedDes = '速度'
    playDes = '播放'
    pauseDes = '暫停'
    stopDes = '停止'
  }

  onMount(() => {
    lang = JSON.parse(localStorage.getItem(LOCAL_STORAGE_TEXT_TO_SPEECH_KEY)) 
    
    const utterance = new SpeechSynthesisUtterance();
    utterance.addEventListener("end", () => {
      textInput.disabled = false;
    });
    utterance.addEventListener("boundary", e => {
      currentCharacter = e.charIndex;
    });
  })

  function playText(text) {
    speedInput.disabled = false;

    if (speechSynthesis.paused & speechSynthesis.speaking) {
      return speechSynthesis.resume();
    }

    if (speechSynthesis.speaking) return;

    utterance.text = text;
    utterance.rate = speed || 1;
    textInput.disabled = true;
    speechSynthesis.speak(utterance);
  }

  function pause() {
    if (speechSynthesis.speaking) speechSynthesis.pause();
    speedInput.disabled = true;
  }

  function stop() {
    speechSynthesis.resume();
    speechSynthesis.cancel();
    speedInput.disabled = true;
  }

  function changeSpeed() {
    stop();
    playText(utterance.text.substring(currentCharacter));
  }

  function switchLang() { 
    lang = !lang 
    localStorage.setItem(LOCAL_STORAGE_TEXT_TO_SPEECH_KEY, JSON.stringify(lang))
  }
</script>

<style>
  .wrap {
    width: 90%;
    margin: 0 auto;
    margin-top: 1rem;
  }

  .box {
    display: flex;
    justify-content: space-between;
  }

  .text {
    width: 100%;
    height: 50vh;
  }

  .right {
    cursor: pointer;
  }
</style>

<div class="wrap">
  <textarea class="text" bind:value={text} bind:this={textInput} />
  <div class="box">
    <div class="left">
      <label for="speed" style="display: inline-block">
        { speedDes }
        <input
          type="number"
          min=".5"
          max="3"
          step=".5"
          bind:value={speed}
          on:input={changeSpeed}
          bind:this={speedInput} />
      </label>
      <button on:click={() => playText(text)}>{ playDes }</button>
      <button on:click={pause}>{ pauseDes }</button>
      <button on:click={stop}>{ stopDes }</button>
    </div>
    <div class="right" on:click={switchLang}>
      <i class="fas fa-globe"></i>
    </div>
  </div>
</div>
