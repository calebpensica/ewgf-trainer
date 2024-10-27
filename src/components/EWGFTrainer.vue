<template>
  <div>
    <h1>EWGF Trainer</h1>
    <p>Press "d" for forward input, "s" for down input, "i" for 2 input.</p>

    <!-- Character Selection Dropdown -->
    <label for="characterSelect">Select Character:</label>
    <select id="characterSelect" v-model="selectedCharacter" @change="loadCharacterSounds">
      <option value="kazuya">Kazuya</option>
      <option value="reina">Reina</option>
      <option value="jin">Jin</option>
      <option value="devilJin">Devil Jin</option>
      <option value="heihachi">Heihachi</option>
    </select>

    <!-- Indicator Row -->
    <div class="indicator-row">
      <span :class="{ active: activeKeys.has('d')  && !activeKeys.has('s') && !activeKeys.has('i') }">f</span>
      <span :class="{ active: activeKeys.size === 0 }">☆</span>
      <span :class="{ active: activeKeys.has('s') }">d</span>
      <span :class="{ active: shouldBoldSecondF() }">f</span>
      <span :class="{ active: activeKeys.has('i') }">2</span>
    </div>

    <!-- Log Container -->
    <div class="log-container">
      <ul>
        <li v-for="log in keyLogs" :key="log.time">
          {{ log.message }} - {{ log.frameSinceLast }}
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
import { ref, onMounted, onBeforeUnmount, nextTick } from 'vue';

export default {
  setup() {
    const keyLogs = ref([]);
    const activeKeys = new Set();
    const sequence = [];
    let lastTime = null;
    const frameDuration = 1000 / 60;

    const selectedCharacter = ref("heihachi");
    const soundPaths = {
      kazuya: { ewgf: require('@/assets/sounds/kazuya-ewgf.mp3'), wgf: require('@/assets/sounds/kazuya-wgf.mp3') },
      reina: { ewgf: require('@/assets/sounds/reina-ewgf.mp3'), wgf: require('@/assets/sounds/reina-wgf.mp3') },
      jin: { ewgf: require('@/assets/sounds/jin-ewgf.mp3'), wgf: require('@/assets/sounds/jin-wgf.mp3') },
      devilJin: { ewgf: require('@/assets/sounds/devil-jin-ewgf.mp3'), wgf: require('@/assets/sounds/devil-jin-wgf.mp3') },
      heihachi: { ewgf: require('@/assets/sounds/heihachi-ewgf.mp3'), wgf: require('@/assets/sounds/heihachi-wgf.mp3') },
    };

    const loadCharacterSounds = () => {
      ewgfSoundPath.value = soundPaths[selectedCharacter.value].ewgf;
      wgfSoundPath.value = soundPaths[selectedCharacter.value].wgf;
    };

    const ewgfSoundPath = ref(soundPaths[selectedCharacter.value].ewgf);
    const wgfSoundPath = ref(soundPaths[selectedCharacter.value].wgf);

    function convertKeys(activeKeys) {
    // Define the mapping object
    const conversionMap = {
        's': 'd',
        'd': 'f',
        'i': '2'
    };

    // Convert the input set
    const convertedKeys = new Set();
    activeKeys.forEach(key => {
        // Use the mapping to convert keys, defaulting to the original key if not found
        convertedKeys.add(conversionMap[key] || key);
    });

    return convertedKeys;
  }

    const logKeys = async () => {
      const currentTime = new Date().getTime();
      const frameSinceLast = lastTime ? Math.round(((currentTime - lastTime) / frameDuration)) : 0;
      lastTime = currentTime;

      const pressedKeys = Array.from(convertKeys(activeKeys)).sort().join(" + ");
      const message = pressedKeys || "Neutral";

      keyLogs.value.push({ message, frameSinceLast, time: currentTime });
      sequence.push({ keys: pressedKeys || "Neutral", frameSinceLast, time: currentTime });

      if (sequence.length > 5) sequence.shift();

      checkForSpecialMoves();

      await nextTick();
      const logContainer = document.querySelector('.log-container');
      logContainer.scrollTop = logContainer.scrollHeight;
    };

    const checkForSpecialMoves = () => {
      if (sequence.length >= 5) {
        const [first, second, third, fourth, fifth] = sequence;
        if (first.keys === "f" && second.keys === "Neutral") {
          const isEWGF = ((third.keys === "d" && fourth.keys === "d + f" && fifth.keys ==="2 + d + f") &&
          (fifth.frameSinceLast === 0))

          const isWGF = ((third.keys === "d" && fourth.keys === "d + f" && fifth.keys ==="2 + d + f") &&
          (fifth.frameSinceLast >= 1 && fifth.frameSinceLast <= 10))

          if (isEWGF) {
            playSound(ewgfSoundPath.value); // Play EWGF sound for selected character
            keyLogs.value.push({ message: "EWGF detected!", frameSinceLast: 0, time: new Date().getTime() });
            sequence.length = 0;
          } else if (isWGF) {
            playSound(wgfSoundPath.value); // Play WGF sound for selected character
            keyLogs.value.push({ message: "WGF detected!", frameSinceLast: 0, time: new Date().getTime() });
            sequence.length = 0;
          } else {
              //debug logging
          }
        }
      }
    };

    const playSound = (soundPath) => {
      const soundInstance = new Audio(soundPath); // Create new instance to play sound concurrently
      soundInstance.play();
    };

    const shouldBoldSecondF = () => {
      return (
        (activeKeys.has('d') && activeKeys.has('s')) ||
        (activeKeys.has('d') && activeKeys.has('s') && activeKeys.has('i'))
      );
    };

    const onKeyDown = (event) => {
      if (["s", "d", "i", "f"].includes(event.key) && !activeKeys.has(event.key)) {
        activeKeys.add(event.key);
        logKeys();
      }
    };

    const onKeyUp = (event) => {
      if (activeKeys.has(event.key)) {
        activeKeys.delete(event.key);
        if (activeKeys.size === 0) {
          logKeys();
        }
      }
    };

    onMounted(() => {
      window.addEventListener("keydown", onKeyDown);
      window.addEventListener("keyup", onKeyUp);
    });

    onBeforeUnmount(() => {
      window.removeEventListener("keydown", onKeyDown);
      window.removeEventListener("keyup", onKeyUp);
    });

    return {
      keyLogs,
      activeKeys,
      selectedCharacter,
      loadCharacterSounds,
      shouldBoldSecondF,
    };
  },
};
</script>

<style scoped>
h1 {
  font-size: 24px;
}

p {
  font-size: 16px;
}

.indicator-row {
  display: flex;
  justify-content: space-around;
  font-size: 24px;
  margin: 10px 0;
}

.indicator-row span {
  transition: font-weight 0.2s;
}

.indicator-row .active {
  font-weight: bold;
}

.log-container {
  max-height: 200px;
  overflow-y: auto;
  border: 1px solid #ddd;
  padding: 10px;
  margin-top: 10px;
}

ul {
  list-style-type: none;
  padding: 0;
  margin: 0;
}

li {
  font-size: 14px;
  margin: 4px 0;
}
</style>
