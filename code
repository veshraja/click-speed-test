import React, { useEffect, useMemo, useRef, useState } from "react";

// Helper: format seconds to mm:ss.ms
const fmt = (t) => {
  const clamped = Math.max(0, t);
  const m = Math.floor(clamped / 60);
  const s = Math.floor(clamped % 60);
  const ms = Math.floor((clamped - Math.floor(clamped)) * 1000)
    .toString()
    .padStart(3, "0");
  return `${m}:${s.toString().padStart(2, "0")}.${ms}`;
};

const PRESETS = [5, 10, 15, 30, 60, 100];

export default function ClickSpeedTest() {
  const [duration, setDuration] = useState(5);
  const [timeLeft, setTimeLeft] = useState(5);
  const [clicks, setClicks] = useState(0);
  const [started, setStarted] = useState(false);
  const [running, setRunning] = useState(false);
  const [finalCps, setFinalCps] = useState(null);
  const [bestCps, setBestCps] = useState(() => {
    const v = localStorage.getItem("cps_best");
    return v ? parseFloat(v) : null;
  });

  const startTsRef = useRef(null);
  const rafRef = useRef(0);

  const elapsed = useMemo(() => {
    if (!started) return 0;
    return duration - timeLeft;
  }, [duration, timeLeft, started]);

  const liveCps = useMemo(() => {
    if (!started || elapsed <= 0) return 0;
    return clicks / elapsed;
  }, [clicks, elapsed, started]);

  const clickOnce = () => {
    if (!running) return;
    setClicks((c) => c + 1);
  };

  const handleTap = () => {
    if (!started) {
      begin();
    }
    clickOnce();
  };

  const handleKeyDown = (e) => {
    if (e.code === "Space" || e.key === " ") {
      e.preventDefault();
      if (!started) begin();
      clickOnce();
    }
  };

  const begin = () => {
    setStarted(true);
    setRunning(true);
    setFinalCps(null);
    startTsRef.current = performance.now();
  };

  const reset = (newDur = duration) => {
    setDuration(newDur);
    setTimeLeft(newDur);
    setClicks(0);
    setStarted(false);
    setRunning(false);
    setFinalCps(null);
    cancelAnimationFrame(rafRef.current);
  };

  // Timer loop using RAF for smoothness and accuracy
  useEffect(() => {
    if (!running) return;

    const tick = () => {
      const elapsedSec = (performance.now() - startTsRef.current) / 1000;
      const tl = Math.max(0, duration - elapsedSec);
      setTimeLeft(tl);
      if (tl <= 0) {
        setRunning(false);
        const cps = clicks / Math.max(1e-9, duration);
        setFinalCps(parseFloat(cps.toFixed(2)));
        setBestCps((prev) => {
          const best = prev == null ? cps : Math.max(prev, cps);
          localStorage.setItem("cps_best", String(best));
          return best;
        });
        return; // stop loop
      }
      rafRef.current = requestAnimationFrame(tick);
    };

    rafRef.current = requestAnimationFrame(tick);
    return () => cancelAnimationFrame(rafRef.current);
  }, [running, duration, clicks]);

  useEffect(() => {
    window.addEventListener("keydown", handleKeyDown);
    return () => window.removeEventListener("keydown", handleKeyDown);
  }, [started, running]);

  const handleShare = async () => {
    const title = "My CPS Test Score";
    const text = finalCps != null
      ? `I clicked ${clicks} times in ${duration}s — ${finalCps.toFixed(2)} CPS!`
      : `Try this Click Speed Test and see your CPS!`;
    const url = typeof window !== "undefined" ? window.location.href : undefined;

    // Web Share API if available
    if (navigator.share) {
      try {
        await navigator.share({ title, text, url });
        return;
      } catch (_) {}
    }

    // Fallback: open social share links
    const encoded = encodeURIComponent(`${text} ${url ?? ""}`.trim());
    const menu = [
      { name: "X", href: `https://twitter.com/intent/tweet?text=${encoded}` },
      { name: "Facebook", href: `https://www.facebook.com/sharer/sharer.php?u=${encodeURIComponent(url ?? "")}&quote=${encoded}` },
      { name: "WhatsApp", href: `https://wa.me/?text=${encoded}` },
      { name: "Telegram", href: `https://t.me/share/url?url=${encodeURIComponent(url ?? "")}&text=${encoded}` },
    ];
    const w = window.open(menu[0].href, "_blank");
    if (!w || w.closed) {
      alert("Pop-up blocked. Please allow pop-ups to share.");
    }
  };

  const ResultCard = () => (
    <div className="grid gap-3 sm:grid-cols-3 mt-6">
      <Stat label="Time" value={`${duration}s`} />
      <Stat label="Clicks" value={clicks} />
      <Stat label="CPS" value={(finalCps ?? liveCps).toFixed(2)} />
    </div>
  );

  return (
    <div className="min-h-screen w-full bg-gradient-to-b from-slate-50 to-white text-slate-900 flex items-center justify-center p-4">
      <div className="w-full max-w-4xl">
        <header className="mb-6 flex flex-col gap-3 sm:flex-row sm:items-end sm:justify-between">
          <div>
            <h1 className="text-3xl sm:text-4xl font-bold tracking-tight">Click Speed Test (CPS)</h1>
            <p className="text-sm text-slate-600 mt-1">Measure how many clicks you can make per second. Timer starts on your first click or spacebar press.</p>
          </div>
          <div className="flex flex-wrap items-center gap-2">
            {PRESETS.map((p) => (
              <button
                key={p}
                onClick={() => reset(p)}
                className={`px-3 py-1.5 rounded-2xl border transition shadow-sm ${
                  duration === p ? "bg-slate-900 text-white" : "bg-white hover:bg-slate-50"
                }`}
                aria-pressed={duration === p}
              >
                {p}s
              </button>
            ))}
          </div>
        </header>

        <div className="grid gap-6">
          <div
            onClick={handleTap}
            onMouseDown={(e) => e.preventDefault()}
            onTouchStart={(e) => {
              e.preventDefault();
              handleTap();
            }}
            role="button"
            aria-label="Click area"
            className={`select-none rounded-2xl border bg-white shadow-md p-6 sm:p-10 text-center cursor-pointer active:scale-[.998] transition grid place-items-center min-h-[240px]`}
          >
            <div>
              {!started && (
                <div>
                  <p className="text-sm uppercase tracking-wider text-slate-500 mb-2">Ready?</p>
                  <p className="text-xl sm:text-2xl font-semibold">Click (or press Space) to start</p>
                </div>
              )}
              {started && running && (
                <div>
                  <p className="text-sm uppercase tracking-wider text-slate-500 mb-1">Time Left</p>
                  <p className="text-4xl sm:text-5xl font-bold tabular-nums">{fmt(timeLeft)}</p>
                  <p className="mt-3 text-lg">Clicks: <span className="font-semibold tabular-nums">{clicks}</span></p>
                  <p className="text-sm text-slate-600">Live CPS: <span className="font-semibold tabular-nums">{liveCps.toFixed(2)}</span></p>
                </div>
              )}
              {started && !running && (
                <div>
                  <p className="text-sm uppercase tracking-wider text-slate-500 mb-1">Time's up!</p>
                  <p className="text-4xl sm:text-5xl font-bold tabular-nums">{(finalCps ?? 0).toFixed(2)} CPS</p>
                  <p className="mt-2 text-lg">You made <span className="font-semibold tabular-nums">{clicks}</span> clicks in <span className="font-semibold">{duration}s</span>.</p>
                  <div className="mt-4 flex flex-wrap items-center gap-3 justify-center">
                    <button onClick={() => reset(duration)} className="px-4 py-2 rounded-2xl bg-slate-900 text-white shadow">Play again</button>
                    <button onClick={handleShare} className="px-4 py-2 rounded-2xl border bg-white shadow">Share score</button>
                  </div>
                </div>
              )}
            </div>
          </div>

          <ResultCard />

          <div className="grid gap-3 sm:grid-cols-3">
            <Stat label="Best CPS (this device)" value={bestCps == null ? "–" : bestCps.toFixed(2)} />
            <Stat label="Clicks per minute (est.)" value={((finalCps ?? liveCps) * 60).toFixed(0)} />
            <Stat label="Elapsed" value={`${elapsed.toFixed(2)}s`} />
          </div>

          <div className="rounded-2xl border bg-white p-5 shadow-sm">
            <h2 className="text-lg font-semibold mb-2">Tips</h2>
            <ul className="list-disc pl-5 text-sm text-slate-700 space-y-1">
              <li>Timer starts on your first click or spacebar press.</li>
              <li>Use the preset buttons above to switch test duration.</li>
              <li>For mobile, tap anywhere in the big box to register clicks.</li>
              <li>Try different clicking techniques (Jitter, Butterfly, Drag) to compare results.</li>
            </ul>
          </div>

          <div className="flex flex-wrap gap-2">
            <button onClick={() => reset(duration)} className="px-4 py-2 rounded-2xl border bg-white shadow">Restart</button>
            <button onClick={() => reset(5)} className="px-4 py-2 rounded-2xl border bg-white">5s</button>
            <button onClick={() => reset(10)} className="px-4 py-2 rounded-2xl border bg-white">10s</button>
            <button onClick={() => reset(60)} className="px-4 py-2 rounded-2xl border bg-white">60s</button>
            <button onClick={() => reset(100)} className="px-4 py-2 rounded-2xl border bg-white">100s</button>
          </div>

          <footer className="text-xs text-slate-500 mt-4">
            <p>
              Built as a demo. No personal data is collected; best score is stored locally in your browser.
            </p>
          </footer>
        </div>
      </div>
    </div>
  );
}

function Stat({ label, value }) {
  return (
    <div className="rounded-2xl border bg-white p-4 shadow-sm">
      <p className="text-xs uppercase tracking-wider text-slate-500 mb-1">{label}</p>
      <p className="text-2xl font-semibold tabular-nums">{value}</p>
    </div>
  );
}
