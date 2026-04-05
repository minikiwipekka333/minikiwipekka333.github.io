# minikiwipekka333.github.io
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>2nd London-Tarragona Tournament</title>
    <script src="https://unpkg.com/react@18/umd/react.production.min.js"></script>
    <script src="https://unpkg.com/react-dom@18/umd/react-dom.production.min.js"></script>
    <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Playfair+Display:ital,wght@0,700;1,400&family=Inter:wght@400;700;900&display=swap" rel="stylesheet">
    <style>
        body { font-family: 'Inter', sans-serif; background-color: #fdfcf0; color: #1a3a3a; }
        h1, h2, h3, .font-serif { font-family: 'Playfair Display', serif; }
        input::-webkit-outer-spin-button, input::-webkit-inner-spin-button { -webkit-appearance: none; margin: 0; }
        .glow-gold { filter: drop-shadow(0 0 8px rgba(197, 160, 89, 0.4)); }
    </style>
</head>
<body>
    <div id="root"></div>

    <script type="text/babel">
        const { useState, useMemo } = React;

        const TournamentApp = () => {
            const playersInitial = ["Jo", "Syd", "Mark", "Adelaida", "Wim", "Adrián", "Nere"];
            
            const generateMatches = (players) => {
                let pool = [...players];
                if (pool.length % 2 !== 0) pool.push("BYE");
                const n = pool.length;
                const rounds = n - 1;
                const matchesPerRound = n / 2;
                let scheduled = [];
                for (let r = 0; r < rounds; r++) {
                    for (let m = 0; m < matchesPerRound; m++) {
                        const p1 = pool[m];
                        const p2 = pool[n - 1 - m];
                        if (p1 !== "BYE" && p2 !== "BYE") {
                            scheduled.push({ id: `r${r}-m${m}`, round: r + 1, player1: p1, player2: p2, score1: 0, score2: 0, winner: null, completed: false });
                        }
                    }
                    pool.splice(1, 0, pool.pop());
                }
                return scheduled;
            };

            const [matches, setMatches] = useState(generateMatches(playersInitial));
            const [playoffs, setPlayoffs] = useState({
                semi1: { score1: 0, score2: 0, winner: null },
                semi2: { score1: 0, score2: 0, winner: null },
                final: { score1: 0, score2: 0, winner: null }
            });

            const standings = useMemo(() => {
                const stats = playersInitial.reduce((acc, name) => {
                    acc[name] = { name, played: 0, won: 0, lost: 0, points: 0, pointsScored: 0 };
                    return acc;
                }, {});

                matches.forEach(match => {
                    if (match.completed) {
                        stats[match.player1].played += 1;
                        stats[match.player2].played += 1;
                        stats[match.player1].pointsScored += Number(match.score1 || 0);
                        stats[match.player2].pointsScored += Number(match.score2 || 0);
                        if (match.winner === match.player1) { 
                            stats[match.player1].won += 1; stats[match.player1].points += 1; stats[match.player2].lost += 1; 
                        } else if (match.winner === match.player2) { 
                            stats[match.player2].won += 1; stats[match.player2].points += 1; stats[match.player1].lost += 1; 
                        }
                    }
                });
                return Object.values(stats).sort((a, b) => b.points - a.points || b.pointsScored - a.pointsScored);
            }, [matches]);

            const top4 = standings.slice(0, 4);

            const updateMatch = (matchId, field, value) => {
                setMatches(prev => prev.map(m => {
                    if (m.id === matchId) {
                        const updated = { ...m, [field]: value };
                        const s1 = field === 'score1' ? Number(value) : Number(m.score1);
                        const s2 = field === 'score2' ? Number(value) : Number(m.score2);
                        if (s1 > s2) updated.winner = m.player1; 
                        else if (s2 > s1) updated.winner = m.player2;
                        updated.completed = true;
                        return updated;
                    }
                    return m;
                }));
            };

            const updatePlayoff = (game, field, value) => {
                setPlayoffs(prev => {
                    const updatedGame = { ...prev[game], [field]: value };
                    const s1 = field === 'score1' ? Number(value) : Number(updatedGame.score1);
                    const s2 = field === 'score2' ? Number(value) : Number(updatedGame.score2);
                    if (s1 > s2) updatedGame.winner = 1;
                    else if (s2 > s1) updatedGame.winner = 2;
                    return { ...prev, [game]: updatedGame };
                });
            };

            return (
                <div className="max-w-6xl mx-auto p-4 md:p-8">
                    <header className="bg-[#1a3a3a] border-b-4 border-[#c5a059] rounded-t-3xl p-10 text-center shadow-2xl mb-8">
                        <h1 className="text-3xl md:text-5xl font-bold text-[#fdfcf0] uppercase tracking-widest mb-2">Tournament Dashboard</h1>
                        <h2 className="text-[#c5a059] text-xl italic">2nd Edition London-Tarragona</h2>
                        <div className="mt-4 inline-block border border-[#c5a059] px-6 py-2 text-[#fdfcf0] text-sm font-bold tracking-[0.2em] uppercase">Annual 2026</div>
                    </header>

                    <div className="grid grid-cols-1 lg:grid-cols-12 gap-8 mb-12">
                        <div className="lg:col-span-7 bg-white rounded-xl shadow-lg overflow-hidden border border-zinc-200">
                            <div className="p-6 bg-[#fafaf5] border-b border-zinc-100 font-bold text-[#1a3a3a] uppercase tracking-tight">Group Stage Standings</div>
                            <table className="w-full text-left">
                                <thead className="bg-[#1a3a3a] text-[#fdfcf0] text-[10px] uppercase font-bold tracking-widest">
                                    <tr>
                                        <th className="px-6 py-4">Player</th>
                                        <th className="px-2 py-4 text-center">P</th>
                                        <th className="px-2 py-4 text-center text-green-400">W</th>
                                        <th className="px-2 py-4 text-center text-red-400">L</th>
                                        <th className="px-2 py-4 text-center text-[#c5a059]">PS</th>
                                        <th className="px-6 py-4 text-right">Wins</th>
                                    </tr>
                                </thead>
                                <tbody className="divide-y divide-zinc-50">
                                    {standings.map((p, i) => (
                                        <tr key={p.name} className={`${i < 4 ? 'bg-[#fffbeb]' : ''}`}>
                                            <td className="px-6 py-4 font-bold text-[#374151] flex items-center gap-3">
                                                <span className={`w-6 h-6 rounded-full flex items-center justify-center text-[10px] ${i < 4 ? 'bg-[#c5a059] text-white' : 'bg-zinc-200 text-zinc-500'}`}>{i+1}</span> {p.name}
                                                {i < 4 && <span className="text-[8px] text-[#c5a059] border border-[#c5a059] px-1 rounded font-black">TOP 4</span>}
                                            </td>
                                            <td className="text-center text-zinc-400 text-sm">{p.played}</td>
                                            <td className="text-center font-bold text-green-700">{p.won}</td>
                                            <td className="text-center font-bold text-red-500">{p.lost}</td>
                                            <td className="text-center font-bold text-zinc-600">{p.pointsScored}</td>
                                            <td className="px-6 py-4 text-right font-black text-2xl text-[#1a3a3a]">{p.points}</td>
                                        </tr>
                                    ))}
                                </tbody>
                            </table>
                        </div>

                        <div className="lg:col-span-5 bg-white rounded-xl shadow-lg overflow-hidden flex flex-col max-h-[600px] border border-zinc-200">
                            <div className="p-6 bg-[#fafaf5] border-b border-zinc-100 font-bold text-[#1a3a3a] uppercase">Group Schedule</div>
                            <div className="p-6 overflow-y-auto space-y-6">
                                {[...Array(Math.max(...matches.map(m => m.round)))].map((_, rIdx) => (
                                    <div key={rIdx} className="space-y-3">
                                        <div className="text-[10px] font-black text-[#c5a059] uppercase tracking-widest border-b border-zinc-50">Round {rIdx+1}</div>
                                        {matches.filter(m => m.round === rIdx+1).map(m => (
                                            <div key={m.id} className="p-3 border rounded-xl bg-[#fdfcfb] flex items-center justify-between gap-2">
                                                <span className="flex-1 text-center font-bold text-xs truncate">{m.player1}</span>
                                                <div className="flex gap-1 items-center px-2 bg-white rounded border border-zinc-100">
                                                    <input type="number" value={m.score1} onChange={(e) => updateMatch(m.id, 'score1', e.target.value)} className="w-6 text-center font-black outline-none" />
                                                    <span className="text-zinc-300">-</span>
                                                    <input type="number" value={m.score2} onChange={(e) => updateMatch(m.id, 'score2', e.target.value)} className="w-6 text-center font-black outline-none" />
                                                </div>
                                                <span className="flex-1 text-center font-bold text-xs truncate">{m.player2}</span>
                                            </div>
                                        ))}
                                    </div>
                                ))}
                            </div>
                        </div>
                    </div>

                    <div className="mt-12 bg-[#1a3a3a]/5 rounded-3xl p-8 border-2 border-dashed border-[#c5a059]/30">
                        <div className="text-center mb-12">
                            <h2 className="text-4xl font-bold text-[#1a3a3a] uppercase tracking-tighter">Final Series</h2>
                            <p className="text-[#c5a059] font-medium italic">Top 4 Knockout Stage</p>
                        </div>

                        <div className="flex flex-col lg:flex-row items-center justify-center gap-12 lg:gap-0">
                            <div className="flex flex-col gap-16 lg:w-1/3">
                                <div className="relative p-6 bg-white rounded-2xl shadow-xl border-l-8 border-[#c5a059]">
                                    <div className="text-[9px] font-bold text-zinc-400 uppercase tracking-widest mb-3">Semi-Final 1</div>
                                    <div className="space-y-4">
                                        <div className="flex justify-between items-center">
                                            <span className="font-bold">{top4[0]?.name || "1st Place"}</span>
                                            <input type="number" value={playoffs.semi1.score1} onChange={(e) => updatePlayoff('semi1', 'score1', e.target.value)} className="w-10 bg-zinc-100 rounded text-center font-black" />
                                        </div>
                                        <div className="flex justify-between items-center">
                                            <span className="font-bold">{top4[3]?.name || "4th Place"}</span>
                                            <input type="number" value={playoffs.semi1.score2} onChange={(e) => updatePlayoff('semi1', 'score2', e.target.value)} className="w-10 bg-zinc-100 rounded text-center font-black" />
                                        </div>
                                    </div>
                                </div>
                                <div className="relative p-6 bg-white rounded-2xl shadow-xl border-l-8 border-[#c5a059]">
                                    <div className="text-[9px] font-bold text-zinc-400 uppercase tracking-widest mb-3">Semi-Final 2</div>
                                    <div className="space-y-4">
                                        <div className="flex justify-between items-center">
                                            <span className="font-bold">{top4[1]?.name || "2nd Place"}</span>
                                            <input type="number" value={playoffs.semi2.score1} onChange={(e) => updatePlayoff('semi2', 'score1', e.target.value)} className="w-10 bg-zinc-100 rounded text-center font-black" />
                                        </div>
                                        <div className="flex justify-between items-center">
                                            <span className="font-bold">{top4[2]?.name || "3rd Place"}</span>
                                            <input type="number" value={playoffs.semi2.score2} onChange={(e) => updatePlayoff('semi2', 'score2', e.target.value)} className="w-10 bg-zinc-100 rounded text-center font-black" />
                                        </div>
                                    </div>
                                </div>
                            </div>

                            <div className="hidden lg:flex flex-col justify-around h-64 w-16">
                                <div className="h-px bg-[#c5a059] w-full"></div>
                                <div className="h-px bg-[#c5a059] w-full"></div>
                            </div>

                            <div className="lg:w-1/3 flex flex-col items-center">
                                <div className="p-8 bg-white rounded-3xl shadow-2xl border-t-8 border-[#c5a059] w-full glow-gold">
                                    <div className="text-center mb-6">
                                        <div className="inline-block bg-[#1a3a3a] text-[#c5a059] px-4 py-1 rounded-full text-[10px] font-black uppercase tracking-widest mb-2">Grand Final</div>
                                        <h3 className="text-2xl font-black">Championship</h3>
                                    </div>
                                    <div className="space-y-6">
                                        <div className="flex justify-between items-center border-b pb-2">
                                            <span className="font-black text-xl text-zinc-700">
                                                {playoffs.semi1.winner === 1 ? top4[0].name : playoffs.semi1.winner === 2 ? top4[3].name : "Winner Semi 1"}
                                            </span>
                                            <input type="number" value={playoffs.final.score1} onChange={(e) => updatePlayoff('final', 'score1', e.target.value)} className="w-12 h-10 bg-[#fafaf5] rounded-lg text-center font-black text-xl border border-[#c5a059]/30" />
                                        </div>
                                        <div className="flex justify-between items-center">
                                            <span className="font-black text-xl text-zinc-700">
                                                {playoffs.semi2.winner === 1 ? top4[1].name : playoffs.semi2.winner === 2 ? top4[2].name : "Winner Semi 2"}
                                            </span>
                                            <input type="number" value={playoffs.final.score2} onChange={(e) => updatePlayoff('final', 'score2', e.target.value)} className="w-12 h-10 bg-[#fafaf5] rounded-lg text-center font-black text-xl border border-[#c5a059]/30" />
                                        </div>
                                    </div>
                                </div>
                                {playoffs.final.winner && (
                                    <div className="mt-8 text-center animate-bounce">
                                        <div className="text-[#c5a059] text-xs font-bold uppercase tracking-widest">National Champion</div>
                                        <div className="text-3xl font-black uppercase italic">
                                            {playoffs.final.winner === 1 
                                                ? (playoffs.semi1.winner === 1 ? top4[0].name : top4[3].name)
                                                : (playoffs.semi2.winner === 1 ? top4[1].name : top4[2].name)
                                            }
                                        </div>
                                    </div>
                                )}
                            </div>
                        </div>
                    </div>
                    <footer className="mt-16 text-center border-t border-zinc-200 pt-8 pb-12">
                        <p className="text-zinc-400 text-[10px] font-bold uppercase tracking-[0.4em]">London-Tarragona Table Tennis Committee 2026</p>
                    </footer>
                </div>
            );
        };

        const root = ReactDOM.createRoot(document.getElementById('root'));
        root.render(<TournamentApp />);
    </script>
</body>
</html>
