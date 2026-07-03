import { createFileRoute } from "@tanstack/react-router";
import { useRef, useState } from "react";
import kalDiceBg from "@/assets/kal-dice-bg.png.asset.json";

export const Route = createFileRoute("/")({
  component: Index,
});

const COLORS = [
  { name: "RED", value: "#ff2d2d" },
  { name: "ORANGE", value: "#ff9500" },
  { name: "YELLOW", value: "#FFD600" },
  { name: "GREEN", value: "#22c55e" },
  { name: "BLUE", value: "#0080ff" },
  { name: "PURPLE", value: "#a855f7" },
];

const BACKGROUNDS = [
  { name: "BLACK", type: "color" as const, value: "#0a0a0a" },
  { name: "BLUE", type: "color" as const, value: "#0033cc" },
  { name: "GREEN", type: "image" as const, value: kalDiceBg.url },
  { name: "RED", type: "color" as const, value: "#cc0000" },
  { name: "BRONZE", type: "color" as const, value: "#cd7f32" },
  { name: "SILVER", type: "color" as const, value: "#c0c0c0" },
  { name: "GOLD", type: "color" as const, value: "#d4af37" },
];

function secureRandomIndex(max: number) {
  const buf = new Uint32Array(1);
  crypto.getRandomValues(buf);
  return buf[0] % max;
}

function Index() {
  const [diceCount, setDiceCount] = useState(4);
  const [dice, setDice] = useState<number[]>(
    Array.from({ length: 4 }, (_, i) => i % COLORS.length)
  );
  const [rolling, setRolling] = useState(false);
  const [whitePhase, setWhitePhase] = useState(false);
  const [bg, setBg] = useState("GREEN");
  const [history, setHistory] = useState<{ id: number; time: string; dice: number[] }[]>([]);
  const audioCtxRef = useRef<AudioContext | null>(null);

  const getAudioContext = () => {
    if (typeof window === "undefined") return null;
    if (!audioCtxRef.current) {
      const AC =
        window.AudioContext ||
        (window as unknown as { webkitAudioContext: typeof AudioContext }).webkitAudioContext;
      if (AC) audioCtxRef.current = new AC();
    }
    return audioCtxRef.current;
  };

  const playClack = (ctx: AudioContext, when: number, intensity = 1) => {
    const dur = 0.04;
    const bufferSize = Math.floor(ctx.sampleRate * dur);
    const buffer = ctx.createBuffer(1, bufferSize, ctx.sampleRate);
    const data = buffer.getChannelData(0);
    for (let i = 0; i < bufferSize; i++) {
      const env = Math.pow(1 - i / bufferSize, 3);
      data[i] = (Math.random() * 2 - 1) * env;
    }
    const source = ctx.createBufferSource();
    source.buffer = buffer;

    const bp = ctx.createBiquadFilter();
    bp.type = "bandpass";
    bp.frequency.value = 2200 + Math.random() * 3000;
    bp.Q.value = 1.8;

    const hp = ctx.createBiquadFilter();
    hp.type = "highpass";
    hp.frequency.value = 900;

    const gain = ctx.createGain();
    const vol = 0.22 * intensity;
    gain.gain.setValueAtTime(vol, when);
    gain.gain.exponentialRampToValueAtTime(0.0001, when + dur);

    source.connect(bp);
    bp.connect(hp);
    hp.connect(gain);
    gain.connect(ctx.destination);
    source.start(when);
    source.stop(when + dur);
  };

  const playRollSound = () => {
    const ctx = getAudioContext();
    if (!ctx) return;
    if (ctx.state === "suspended") ctx.resume();

    const now = ctx.currentTime;
    const totalDur = 0.55;
    // Dense tumbling rattle: many overlapping clacks like dice in a cup
    const numClacks = 42;
    for (let i = 0; i < numClacks; i++) {
      const progress = i / numClacks;
      // slightly denser in the middle
      const t = now + progress * totalDur + (Math.random() - 0.5) * 0.03;
      // envelope: quick rise, sustain, fade out at end
      let intensity;
      if (progress < 0.15) intensity = 0.5 + progress * 3;
      else if (progress > 0.75) intensity = 1 - (progress - 0.75) * 3;
      else intensity = 0.9 + Math.random() * 0.2;
      playClack(ctx, t, intensity);
    }
  };

  const playRevealSound = () => {
    const ctx = getAudioContext();
    if (!ctx) return;
    if (ctx.state === "suspended") ctx.resume();

    const osc = ctx.createOscillator();
    const gain = ctx.createGain();
    osc.type = "sine";
    osc.frequency.setValueAtTime(880, ctx.currentTime);
    osc.frequency.exponentialRampToValueAtTime(440, ctx.currentTime + 0.15);
    gain.gain.setValueAtTime(0.15, ctx.currentTime);
    gain.gain.exponentialRampToValueAtTime(0.001, ctx.currentTime + 0.15);
    osc.connect(gain);
    gain.connect(ctx.destination);
    osc.start();
    osc.stop(ctx.currentTime + 0.15);
  };

  const roll = () => {
    if (rolling) return;
    setRolling(true);
    setWhitePhase(true);
    playRollSound();

    setTimeout(() => {
      const final = dice.map((current) => {
        let n = secureRandomIndex(COLORS.length);
        while (n === current) n = secureRandomIndex(COLORS.length);
        return n;
      });
      setDice(final);
      setWhitePhase(false);
      setRolling(false);
      playRevealSound();
      const time = new Date().toLocaleTimeString([], { hour: "2-digit", minute: "2-digit", second: "2-digit" });
      setHistory((h) => [{ id: Date.now(), time, dice: final }, ..
