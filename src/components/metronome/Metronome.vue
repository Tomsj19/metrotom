<script setup lang="ts">
import { ref, watch, onUnmounted } from 'vue';

import { Button } from '@/components/ui/button';
import AppButton from '../AppButton.vue';
import AppCard from '../AppCard.vue';
import TempoDisplay from './TempoDisplay.vue';
import { Slider } from '@/components/ui/slider';
import { Play, Square, Volume2, VolumeX } from 'lucide-vue-next';

// --- ESTADOS DO METRÔNOMO ---
const bpm = ref<number>(120);
const beatsPerMeasure = ref<number>(4);
const isPlaying = ref<boolean>(false);
const currentBeat = ref<number>(0);
const isMuted = ref<boolean>(false);

// --- CONFIGURAÇÕES DE ÁUDIO & TIMING ---
let audioCtx: AudioContext | null = null;
let timerID: number | null = null;

const lookahead = 25.0; // Frequência do loop de agendamento (ms)
const scheduleAheadTime = 0.1; // Janela de tempo para agendar áudio no futuro (s)

let nextNoteTime = 0.0; // Quando a próxima nota deve tocar (segundos do AudioContext)
let noteInMeasure = 0; // Qual tempo da compasso estamos (0 a beatsPerMeasure - 1)

function nextNote() {
  const secondsPerBeat = 60.0 / bpm.value;
  nextNoteTime += secondsPerBeat;
  noteInMeasure = (noteInMeasure + 1) % beatsPerMeasure.value;
}

function scheduleNote(beatNumber: number, time: number) {
  if (!audioCtx || isMuted.value) return;

  // Atualiza a UI no tempo exato da nota
  const timeUntilBeat = Math.max(0, (time - audioCtx.currentTime) * 1000);
  setTimeout(() => {
    if (isPlaying.value) {
      currentBeat.value = beatNumber;
    }
  }, timeUntilBeat);

  // Cria o tom do áudio (Oscilador)
  const osc = audioCtx.createOscillator();
  const gain = audioCtx.createGain();

  // O primeiro tempo do compasso tem um som mais agudo (Acentuado)
  if (beatNumber === 0) {
    osc.frequency.value = 1000; // 1000 Hz
  } else {
    osc.frequency.value = 800; // 800 Hz
  }

  // Envelope para evitar ruídos de clique no som
  gain.gain.setValueAtTime(1, time);
  gain.gain.exponentialRampToValueAtTime(0.001, time + 0.05);

  osc.connect(gain);
  gain.connect(audioCtx.destination);

  osc.start(time);
  osc.stop(time + 0.05);
}

function scheduler() {
  if (!audioCtx) return;

  while (nextNoteTime < audioCtx.currentTime + scheduleAheadTime) {
    scheduleNote(noteInMeasure, nextNoteTime);
    nextNote();
  }
}

function togglePlay() {
  if (isPlaying.value) {
    stopMetronome();
  } else {
    startMetronome();
  }
}

function startMetronome() {
  if (!audioCtx) {
    audioCtx = new (window.AudioContext || (window as unknown as { webkitAudioContext: typeof AudioContext; }).webkitAudioContext)();
  }

  if (audioCtx.state === 'suspended') {
    audioCtx.resume();
  }

  isPlaying.value = true;
  noteInMeasure = 0;
  nextNoteTime = audioCtx.currentTime;
  currentBeat.value = 0;

  timerID = window.setInterval(scheduler, lookahead);
}

function stopMetronome() {
  isPlaying.value = false;
  if (timerID !== null) {
    clearInterval(timerID);
    timerID = null;
  }
  currentBeat.value = 0;
}

// Se alterar o BPM com ele rodando, ajusta sem desincronizar drasticamente
watch(bpm, () => {
  if (isPlaying.value && audioCtx) {
    nextNoteTime = audioCtx.currentTime;
  }
});

onUnmounted(() => {
  stopMetronome();
  if (audioCtx) {
    audioCtx.close();
  }
});

const decreaseBpm = () => {
  if (bpm.value > 40) {
    bpm.value--;
  }
};

const increaseBpm = () => {
  if (bpm.value < 240) {
    bpm.value++;
  }
};

function handleSliderUpdate(val: number[] | undefined) {
  if (val && val.length > 0 && typeof val[0] === 'number') {
    bpm.value = val[0];
  }
  // <!-- @update:model-value="(val) => bpm = val?.[0] ?? bpm" -->
}
</script>

<template>
  <section class="flex flex-col items-center gap-8">

    <AppCard class="flex items-center">
      <header class="flex justify-between">
        <h3>Tempo</h3>
        <h4>Allegro</h4>
      </header>
      <div class="flex items-center gap-6">
        <AppButton
          class="flex size-12 items-center justify-center rounded-full border border-border text-2xl transition hover:bg-accent"
          @click="decreaseBpm">
          −
        </AppButton>

        <TempoDisplay :bpm="bpm" />

        <AppButton
          class="flex size-12 items-center justify-center rounded-full border border-border text-2xl transition hover:bg-accent"
          @click="increaseBpm">
          +
        </AppButton>
      </div>

      <!-- Indicador de Tempos (Visualizador do Compasso) -->
      <div class="flex w-full justify-center items-center gap-3 py-2">
        <div v-for="beat in beatsPerMeasure" :key="beat" class="h-4 flex-1 rounded-full transition-all duration-75"
          :class="[
            isPlaying && currentBeat === beat - 1
              ? (beat === 1 ? 'bg-primary scale-110 shadow-md' : 'bg-primary/80 scale-105')
              : 'bg-muted'
          ]" />
      </div>

      <Slider :model-value="[bpm]" @update:model-value="handleSliderUpdate" :min="40" :max="208" :step="1"
        class="w-full" />

      <!-- Seleção de Compasso -->
      <div class="flex justify-center gap-2">
        <Button v-for="measure in [2, 3, 4, 6]" :key="measure" size="sm"
          :variant="beatsPerMeasure === measure ? 'default' : 'outline'" @click="beatsPerMeasure = measure"
          class="w-12">
          {{ measure }}/4
        </Button>
      </div>

      <!-- Controles de Play e Mute -->
      <div class="flex items-center gap-3 pt-2">
        <Button size="icon" variant="ghost" @click="isMuted = !isMuted" class="h-12 w-12 rounded-full">
          <VolumeX v-if="isMuted" class="h-5 w-5 text-muted-foreground" />
          <Volume2 v-else class="h-5 w-5" />
        </Button>

        <Button size="lg" class="flex-1 h-12 text-base font-semibold gap-2 rounded-full"
          :variant="isPlaying ? 'destructive' : 'default'" @click="togglePlay">
          <Square v-if="isPlaying" class="h-5 w-5 fill-current" />
          <Play v-else class="h-5 w-5 fill-current" />
          {{ isPlaying ? 'Parar' : 'Iniciar' }}
        </Button>
      </div>

      <!-- <AppButton variant="default">
        Iniciar
      </AppButton> -->

    </AppCard>
  </section>
</template>