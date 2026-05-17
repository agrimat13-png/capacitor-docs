export default function MonsterDenEscapeGame() {
  const React = window.React;
  const { useState, useEffect, useRef } = React;

  const GAME_WIDTH = 900;
  const GAME_HEIGHT = 500;
  const PLAYER_SIZE = 40;
  const MONSTER_SIZE = 55;
  const DEN_WIDTH = 80;

  const [playerX, setPlayerX] = useState(80);
  const [playerY, setPlayerY] = useState(GAME_HEIGHT / 2);
  const [monsterX, setMonsterX] = useState(20);
  const [monsterY, setMonsterY] = useState(GAME_HEIGHT / 2);
  const [gameState, setGameState] = useState("start");
  const [message, setMessage] = useState("Run to the den before the monster catches you!");
  const keysRef = useRef({});

  const resetGame = () => {
    setPlayerX(80);
    setPlayerY(GAME_HEIGHT / 2);
    setMonsterX(20);
    setMonsterY(GAME_HEIGHT / 2);
    setGameState("playing");
    setMessage("Escape to the den!");
  };

  useEffect(() => {
    const down = (e) => {
      keysRef.current[e.key.toLowerCase()] = true;
    };

    const up = (e) => {
      keysRef.current[e.key.toLowerCase()] = false;
    };

    window.addEventListener("keydown", down);
    window.addEventListener("keyup", up);

    return () => {
      window.removeEventListener("keydown", down);
      window.removeEventListener("keyup", up);
    };
  }, []);

  useEffect(() => {
    if (gameState !== "playing") return;

    const interval = setInterval(() => {
      setPlayerX((prev) => {
        let next = prev;
        if (keysRef.current["d"] || keysRef.current["arrowright"]) next += 7;
        if (keysRef.current["a"] || keysRef.current["arrowleft"]) next -= 7;
        return Math.max(0, Math.min(GAME_WIDTH - PLAYER_SIZE, next));
      });

      setPlayerY((prev) => {
        let next = prev;
        if (keysRef.current["w"] || keysRef.current["arrowup"]) next -= 7;
        if (keysRef.current["s"] || keysRef.current["arrowdown"]) next += 7;
        return Math.max(0, Math.min(GAME_HEIGHT - PLAYER_SIZE, next));
      });

      setMonsterX((prev) => prev + 2.8);

      setMonsterY((prev) => {
        if (prev < playerY) return prev + 1.6;
        if (prev > playerY) return prev - 1.6;
        return prev;
      });
    }, 30);

    return () => clearInterval(interval);
  }, [gameState, playerY]);

  useEffect(() => {
    if (gameState !== "playing") return;

    const playerCenterX = playerX + PLAYER_SIZE / 2;
    const playerCenterY = playerY + PLAYER_SIZE / 2;
    const monsterCenterX = monsterX + MONSTER_SIZE / 2;
    const monsterCenterY = monsterY + MONSTER_SIZE / 2;

    const distance = Math.hypot(playerCenterX - monsterCenterX, playerCenterY - monsterCenterY);

    if (distance < 45) {
      setGameState("lost");
      setMessage("The monster caught you!");
    }

    if (playerX + PLAYER_SIZE >= GAME_WIDTH - DEN_WIDTH) {
      setGameState("won");
      setMessage("You reached the den safely!");
    }
  }, [playerX, playerY, monsterX, monsterY, gameState]);

  return (
    <div className="min-h-screen bg-slate-950 text-white flex flex-col items-center justify-center p-6 font-sans">
      <div className="text-center mb-6">
        <h1 className="text-5xl font-black tracking-tight mb-2">Monster Den Escape</h1>
        <p className="text-slate-300 text-lg">Use WASD or Arrow Keys to run toward the glowing den.</p>
      </div>

      <div
        className="relative overflow-hidden rounded-3xl border-4 border-slate-700 shadow-2xl"
        style={{ width: GAME_WIDTH, height: GAME_HEIGHT }}
      >
        <div className="absolute inset-0 bg-gradient-to-r from-emerald-950 via-slate-900 to-stone-900" />

        <div className="absolute inset-0 opacity-20">
          {[...Array(25)].map((_, i) => (
            <div
              key={i}
              className="absolute bg-white rounded-full"
              style={{
                width: Math.random() * 4 + 2,
                height: Math.random() * 4 + 2,
                left: Math.random() * GAME_WIDTH,
                top: Math.random() * GAME_HEIGHT,
              }}
            />
          ))}
        </div>

        <div
          className="absolute right-0 top-0 h-full flex items-center justify-center"
          style={{ width: DEN_WIDTH }}
        >
          <div className="w-24 h-36 bg-amber-700 rounded-t-full border-4 border-yellow-300 shadow-[0_0_40px_rgba(255,215,0,0.9)] flex items-center justify-center text-black font-bold text-lg">
            DEN
          </div>
        </div>

        <div
          className="absolute rounded-full bg-cyan-400 border-4 border-white shadow-[0_0_30px_rgba(34,211,238,1)] transition-all duration-75"
          style={{
            width: PLAYER_SIZE,
            height: PLAYER_SIZE,
            left: playerX,
            top: playerY,
          }}
        />

        <div
          className="absolute transition-all duration-75"
          style={{
            width: MONSTER_SIZE,
            height: MONSTER_SIZE,
            left: monsterX,
            top: monsterY,
          }}
        >
          <div className="w-full h-full bg-red-700 rounded-full border-4 border-red-300 shadow-[0_0_35px_rgba(239,68,68,1)] flex items-center justify-center text-3xl">
            👹
          </div>
        </div>

        {(gameState === "start" || gameState === "won" || gameState === "lost") && (
          <div className="absolute inset-0 bg-black/70 flex flex-col items-center justify-center text-center backdrop-blur-sm">
            <h2 className="text-4xl font-extrabold mb-4">{message}</h2>
            <button
              onClick={resetGame}
              className="px-8 py-4 bg-emerald-500 hover:bg-emerald-400 text-black font-bold text-xl rounded-2xl shadow-xl transition-all hover:scale-105"
            >
              {gameState === "start" ? "Start Running" : "Play Again"}
            </button>
          </div>
        )}
      </div>

      <div className="mt-6 grid grid-cols-2 gap-4 text-center">
        <div className="bg-slate-900 border border-slate-700 rounded-2xl p-4 w-72 shadow-lg">
          <h3 className="font-bold text-xl mb-2">Goal</h3>
          <p className="text-slate-300">Reach the safe den on the right side before the monster catches you.</p>
        </div>

        <div className="bg-slate-900 border border-slate-700 rounded-2xl p-4 w-72 shadow-lg">
          <h3 className="font-bold text-xl mb-2">Controls</h3>
          <p className="text-slate-300">Move with WASD or the Arrow Keys.</p>
        </div>
      </div>
    </div>
  );
}
