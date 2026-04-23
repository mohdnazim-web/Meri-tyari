<!DOCTYPE html>
<html lang="hi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Tyari Pro - Final Master Edition</title>
    <script src="https://unpkg.com/react@18/umd/react.development.js"></script>
    <script src="https://unpkg.com/react-dom@18/umd/react-dom.development.js"></script>
    <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css">
    <style>
        body { background: radial-gradient(circle at top right, #0f172a, #020617); color: white; font-family: 'Inter', sans-serif; min-height: 100vh; }
        .glass { background: rgba(255, 255, 255, 0.03); backdrop-filter: blur(15px); border: 1px solid rgba(255, 255, 255, 0.08); }
        .hide-scrollbar::-webkit-scrollbar { display: none; }
        .active-opt { background: #4f46e5 !important; border-color: #818cf8 !important; transform: scale(1.02); box-shadow: 0 0 20px rgba(79, 70, 229, 0.3); }
        .admin-input { background: rgba(0,0,0,0.2); border: 1px solid rgba(255,255,255,0.1); padding: 12px; border-radius: 12px; color: white; width: 100%; outline: none; }
        .admin-input:focus { border-color: #6366f1; }
    </style>
</head>
<body class="hide-scrollbar">
    <div id="root"></div>

    <script type="text/babel">
        const { useState, useEffect } = React;

        function App() {
            // --- State Management ---
            const [view, setView] = useState('home'); 
            const [activeParent, setActiveParent] = useState(null);
            const [activeTest, setActiveTest] = useState(null);
            const [currentQuestions, setCurrentQuestions] = useState([]);
            const [currentIndex, setCurrentIndex] = useState(0);
            const [userAnswers, setUserAnswers] = useState({});
            const [timeLeft, setTimeLeft] = useState(0);

            const [db, setDb] = useState(() => {
                const saved = localStorage.getItem('tyari_pro_ultra_v1');
                return saved ? JSON.parse(saved) : {
                    sections: [],
                    ads: [{ id: 1, text: "Welcome to Tyari Pro! Start your success journey today.", active: true }],
                    appTitle: "TYARI PRO"
                };
            });

            useEffect(() => { localStorage.setItem('tyari_pro_ultra_v1', JSON.stringify(db)); }, [db]);

            // --- Timer Logic ---
            useEffect(() => {
                if (view === 'testing' && timeLeft > 0) {
                    const timer = setInterval(() => setTimeLeft(p => p - 1), 1000);
                    return () => clearInterval(timer);
                } else if (timeLeft === 0 && view === 'testing') setView('result');
            }, [timeLeft, view]);

            // --- Admin Security ---
            let clickCount = 0;
            const handleHiddenAdmin = () => {
                clickCount++;
                if (clickCount === 3) {
                    if(prompt("Enter Admin Master Key:") === "Nazim@483563") setView('admin');
                    clickCount = 0;
                }
                setTimeout(() => { clickCount = 0; }, 1000);
            };

            // --- Core Functions ---
            const startExam = (test) => {
                if (!test.questions || test.questions.length === 0) return alert("Is test mein sawal nahi hain!");
                const shuffled = [...test.questions].sort(() => 0.5 - Math.random()).slice(0, test.qLimit || 25);
                setCurrentQuestions(shuffled);
                setActiveTest(test);
                setTimeLeft((test.time || 60) * 60);
                setUserAnswers({});
                setCurrentIndex(0);
                setView('testing');
            };

            // --- UI Components ---
            const HomeView = () => (
                <div className="max-w-5xl mx-auto p-6 pt-10 animate-in fade-in">
                    <header className="flex justify-between items-center mb-12">
                        <h1 className="text-3xl font-black italic tracking-tighter text-indigo-500 uppercase">{db.appTitle}</h1>
                        <div onClick={handleHiddenAdmin} className="w-12 h-12 cursor-pointer flex items-center justify-center text-gray-700 hover:text-indigo-400 transition">
                            <i className="fas fa-shield-alt"></i>
                        </div>
                    </header>

                    {db.ads.map(ad => ad.active && (
                        <div key={ad.id} className="glass p-4 rounded-2xl mb-10 border-dashed border-indigo-500/40 text-center animate-pulse text-sm">
                            <i className="fas fa-bullhorn mr-2 text-indigo-400"></i> {ad.text}
                        </div>
                    ))}

                    <div className="grid grid-cols-1 md:grid-cols-3 gap-8">
                        {db.sections.length === 0 && <p className="col-span-3 text-center text-gray-500 italic py-20 border-2 border-dashed border-white/5 rounded-[3rem]">Admin Panel se Content add karein...</p>}
                        {db.sections.map(sec => (
                            <div key={sec.id} onClick={() => {setActiveParent(sec); setView('sub-list');}} className="glass p-10 rounded-[3rem] cursor-pointer hover:border-indigo-500 border border-transparent transition-all group">
                                <div className="w-14 h-14 bg-indigo-500/10 rounded-2xl flex items-center justify-center mb-6 group-hover:bg-indigo-500 transition-colors">
                                    <i className="fas fa-layer-group text-indigo-500 group-hover:text-white text-xl"></i>
                                </div>
                                <h3 className="text-2xl font-bold">{sec.title}</h3>
                                <p className="text-gray-500 text-xs mt-2 uppercase tracking-widest">{sec.tests?.length || 0} Test Packs</p>
                            </div>
                        ))}
                    </div>
                </div>
            );

            const SubListView = () => (
                <div className="max-w-2xl mx-auto p-6 pt-10">
                    <button onClick={() => setView('home')} className="mb-8 text-gray-500 hover:text-white font-bold"><i className="fas fa-chevron-left mr-2"></i> BACK</button>
                    <h2 className="text-4xl font-black mb-10 text-indigo-500 uppercase">{activeParent.title}</h2>
                    <div className="space-y-4">
                        {activeParent.tests.map(test => (
                            <div key={test.id} onClick={() => startExam(test)} className="glass p-6 rounded-[2rem] flex justify-between items-center cursor-pointer hover:bg-white/5 border border-transparent hover:border-indigo-500/30 transition group">
                                <div>
                                    <p className="font-bold text-lg group-hover:text-indigo-400">{test.title}</p>
                                    <p className="text-[10px] text-gray-500 uppercase mt-1 font-bold">Questions: {test.qLimit} | Time: {test.time}m | Neg: -{test.neg}</p>
                                </div>
                                <div className="w-10 h-10 rounded-full bg-indigo-500/10 flex items-center justify-center group-hover:bg-indigo-500 transition">
                                    <i className="fas fa-play text-xs text-indigo-500 group-hover:text-white"></i>
                                </div>
                            </div>
                        ))}
                    </div>
                </div>
            );

            const AdminPanel = () => {
                const [tab, setTab] = useState('structure');
                const [newSec, setNewSec] = useState("");
                const [selectedParentId, setSelectedParentId] = useState("");
                const [selectedTestId, setSelectedTestId] = useState("");
                const [qData, setQData] = useState({ text: '', options: ['', '', '', ''], correct: 0, analysis: '' });

                const createTest = (parentID) => {
                    const title = prompt("Test Name (e.g. Test 1):");
                    if(!title) return;
                    const qLimit = prompt("Test Question Limit (e.g. 25):", "25");
                    const time = prompt("Timer (Minutes):", "60");
                    const neg = prompt("Negative Marking (0.25, 0.33):", "0.25");
                    
                    const updated = db.sections.map(s => s.id === parentID ? {...s, tests: [...s.tests, {id: Date.now(), title, qLimit: parseInt(qLimit), time: parseInt(time), neg: parseFloat(neg), questions: []}]} : s);
                    setDb({...db, sections: updated});
                };

                return (
                    <div className="max-w-5xl mx-auto p-6">
                        <div className="glass p-8 rounded-[3rem] border border-white/10">
                            <div className="flex gap-8 mb-10 border-b border-white/5 pb-4">
                                <button onClick={() => setTab('structure')} className={`font-bold transition ${tab==='structure'?'text-indigo-400 border-b-2 border-indigo-400 pb-4': 'text-gray-500'}`}>1. Structure</button>
                                <button onClick={() => setTab('questions')} className={`font-bold transition ${tab==='questions'?'text-indigo-400 border-b-2 border-indigo-400 pb-4': 'text-gray-500'}`}>2. Questions</button>
                                <button onClick={() => setTab('ads')} className={`font-bold transition ${tab==='ads'?'text-indigo-400 border-b-2 border-indigo-400 pb-4': 'text-gray-500'}`}>3. Ads</button>
                                <button onClick={() => setView('home')} className="ml-auto text-red-400 font-bold">EXIT ADMIN</button>
                            </div>

                            {tab === 'structure' && (
                                <div className="space-y-8">
                                    <div className="flex gap-3">
                                        <input className="admin-input flex-1" placeholder="New Section (SSC, Railway...)" value={newSec} onChange={e=>setNewSec(e.target.value)} />
                                        <button onClick={()=>{setDb({...db, sections: [...db.sections, {id: Date.now(), title: newSec, tests: []}]}); setNewSec("");}} className="bg-indigo-600 px-8 rounded-xl font-bold">ADD</button>
                                    </div>
                                    <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
                                        {db.sections.map(s => (
                                            <div key={s.id} className="bg-white/5 p-6 rounded-[2rem] border border-white/5">
                                                <div className="flex justify-between items-center mb-4">
                                                    <span className="font-black text-indigo-400 uppercase tracking-tighter">{s.title}</span>
                                                    <button onClick={()=>createTest(s.id)} className="text-[10px] bg-emerald-600 px-3 py-1 rounded-lg font-bold">+ ADD TEST</button>
                                                </div>
                                                <div className="flex flex-wrap gap-2">
                                                    {s.tests.map(t => <span key={t.id} className="bg-black/40 px-2 py-1 rounded text-[9px] border border-white/5">{t.title}</span>)}
                                                </div>
                                            </div>
                                        ))}
                                    </div>
                                </div>
                            )}

                            {tab === 'questions' && (
                                <div className="grid gap-4">
                                    <div className="grid grid-cols-2 gap-4">
                                        <select className="admin-input opacity-60" onChange={e => setSelectedParentId(e.target.value)}>
                                            <option>Select Section</option>
                                            {db.sections.map(s => <option key={s.id} value={s.id} className="bg-slate-900">{s.title}</option>)}
                                        </select>
                                        <select className="admin-input opacity-60" onChange={e => setSelectedTestId(e.target.value)}>
                                            <option>Select Test</option>
                                            {db.sections.find(s => s.id === selectedParentId)?.tests.map(t => <option key={t.id} value={t.id} className="bg-slate-900">{t.title}</option>)}
                                        </select>
                                    </div>
                                    <textarea className="admin-input h-24" placeholder="Enter Question..." onChange={e=>setQData({...qData, text: e.target.value})} value={qData.text}></textarea>
                                    <div className="grid grid-cols-2 gap-3">
                                        {qData.options.map((o, i) => <input key={i} className="admin-input text-sm" placeholder={`Option ${String.fromCharCode(65+i)}`} value={o} onChange={e => {const copy=[...qData.options]; copy[i]=e.target.value; setQData({...qData, options: copy});}} />)}
                                    </div>
                                    <input type="number" className="admin-input" placeholder="Correct Option Index (0-3)" value={qData.correct} onChange={e=>setQData({...qData, correct: parseInt(e.target.value)})} />
                                    <textarea className="admin-input h-20" placeholder="AI Analysis / Explanation..." onChange={e=>setQData({...qData, analysis: e.target.value})} value={qData.analysis}></textarea>
                                    <button onClick={() => {
                                        if(!selectedTestId) return alert("Pehle Test select karein!");
                                        const updated = db.sections.map(s => s.id === selectedParentId ? {...s, tests: s.tests.map(t => t.id == selectedTestId ? {...t, questions: [...t.questions, qData]} : t)} : s);
                                        setDb({...db, sections: updated});
                                        setQData({ text: '', options: ['', '', '', ''], correct: 0, analysis: '' });
                                        alert("Question Added!");
                                    }} className="w-full bg-indigo-600 py-4 rounded-2xl font-bold mt-4 shadow-lg shadow-indigo-600/20">SAVE TO QUESTION BANK</button>
                                </div>
                            )}

                            {tab === 'ads' && (
                                <div className="space-y-4">
                                    {db.ads.map(ad => (
                                        <div key={ad.id} className="glass p-6 rounded-2xl flex gap-4 items-center">
                                            <input className="admin-input flex-1" value={ad.text} onChange={e => {
                                                const newAds = db.ads.map(a => a.id === ad.id ? {...a, text: e.target.value} : a);
                                                setDb({...db, ads: newAds});
                                            }} />
                                            <button onClick={() => {
                                                const newAds = db.ads.map(a => a.id === ad.id ? {...a, active: !a.active} : a);
                                                setDb({...db, ads: newAds});
                                            }} className={`px-6 py-2 rounded-xl font-bold ${ad.active ? 'bg-emerald-600':'bg-gray-700'}`}>{ad.active ? 'ON':'OFF'}</button>
                                        </div>
                                    ))}
                                </div>
                            )}
                        </div>
                    </div>
                );
            };

            const ResultView = () => {
                let correct = 0, wrong = 0;
                const negVal = activeTest?.neg || 0;

                currentQuestions.forEach((q, i) => {
                    if(userAnswers[i] !== undefined) {
                        if(userAnswers[i] === q.correct) correct++;
                        else wrong++;
                    }
                });

                const totalNeg = (wrong * negVal).toFixed(2);
                const score = (correct - totalNeg).toFixed(2);

                return (
                    <div className="max-w-4xl mx-auto p-6 pt-10 animate-in fade-in">
                        <div className="glass p-12 rounded-[3.5rem] text-center mb-10 border-b-8 border-indigo-500 shadow-2xl">
                            <h2 className="text-8xl font-black text-indigo-400 mb-4">{score}</h2>
                            <p className="text-gray-400 uppercase tracking-widest text-xs font-bold">Total Adjusted Score</p>
                            <div className="grid grid-cols-3 gap-6 mt-10 max-w-xl mx-auto">
                                <div className="bg-emerald-500/10 p-5 rounded-3xl border border-emerald-500/20"><p className="text-[10px] text-emerald-500 font-bold uppercase mb-1">Correct</p><p className="text-3xl font-bold">{correct}</p></div>
                                <div className="bg-red-500/10 p-5 rounded-3xl border border-red-500/20"><p className="text-[10px] text-red-500 font-bold uppercase mb-1">Wrong</p><p className="text-3xl font-bold">{wrong}</p></div>
                                <div className="bg-orange-500/10 p-5 rounded-3xl border border-orange-500/20"><p className="text-[10px] text-orange-500 font-bold uppercase mb-1">Penalty</p><p className="text-3xl font-bold">-{totalNeg}</p></div>
                            </div>
                            <button onClick={() => setView('home')} className="mt-12 bg-indigo-600 hover:bg-indigo-500 px-12 py-4 rounded-full font-bold transition shadow-xl shadow-indigo-600/20">TRY ANOTHER TEST</button>
                        </div>

                        <h3 className="text-2xl font-black mb-8 flex items-center gap-3"><i className="fas fa-brain text-indigo-500"></i> AI TEST INSIGHTS</h3>
                        <div className="space-y-8">
                            {currentQuestions.map((q, i) => (
                                <div key={i} className="glass rounded-[2.5rem] overflow-hidden border border-white/5">
                                    <div className="p-8 bg-white/5 border-b border-white/5">
                                        <div className="flex justify-between items-center mb-6">
                                            <span className="bg-indigo-500 text-[10px] px-3 py-1 rounded-full font-bold">QUESTION {i+1}</span>
                                            {userAnswers[i] === q.correct ? <span className="text-emerald-500 text-sm font-black italic">✓ CORRECT</span> : <span className="text-red-500 text-sm font-black italic">✗ WRONG</span>}
                                        </div>
                                        <p className="text-xl font-medium leading-relaxed">{q.text}</p>
                                    </div>
                                    <div className="p-8 grid md:grid-cols-2 gap-4">
                                        {q.options.map((opt, oIdx) => (
                                            <div key={oIdx} className={`p-5 rounded-2xl border flex items-center gap-4 text-sm ${oIdx === q.correct ? 'bg-emerald-500/10 border-emerald-500/40 text-emerald-300' : (userAnswers[i] === oIdx ? 'bg-red-500/10 border-red-500/40 text-red-300' : 'bg-white/5 border-white/10')}`}>

                                                <span className="text-[10px] font-black opacity-40">{String.fromCharCode(65+oIdx)}</span> {opt}
                                            </div>
                                        ))}
                                    </div>
                                    <div className="p-8 bg-indigo-500/5 border-t border-indigo-500/10">
                                        <div className="grid grid-cols-2 md:grid-cols-4 gap-4 mb-6">
                                            <div className="bg-black/20 p-4 rounded-2xl border border-white/5 text-center">
                                                <p className="text-[9px] text-gray-500 uppercase font-bold mb-1 tracking-widest">Topic</p>
                                                <p className="text-xs font-bold text-indigo-300">Exam Core</p>
                                            </div>
                                            <div className="bg-black/20 p-4 rounded-2xl border border-white/5 text-center">
                                                <p className="text-[9px] text-gray-500 uppercase font-bold mb-1 tracking-widest">Success Rate</p>
                                                <p className="text-xs font-bold text-emerald-400">75% Avg</p>
                                            </div>
                                            <div className="bg-black/20 p-4 rounded-2xl border border-white/5 text-center">
                                                <p className="text-[9px] text-gray-500 uppercase font-bold mb-1 tracking-widest">Ideal Time</p>
                                                <p className="text-xs font-bold">45s</p>
                                            </div>
                                            <div className="bg-black/20 p-4 rounded-2xl border border-white/5 text-center">
                                                <p className="text-[9px] text-gray-500 uppercase font-bold mb-1 tracking-widest">Priority</p>
                                                <p className="text-xs font-bold text-orange-400">High</p>
                                            </div>
                                        </div>
                                        <div className="p-6 bg-indigo-500/10 rounded-3xl border-l-4 border-indigo-500 text-sm italic text-indigo-100/70 leading-relaxed">
                                            <b className="not-italic text-indigo-400 block mb-2 uppercase text-[10px] tracking-widest">AI Expert View:</b>
                                            {q.analysis || "Iss sawal ka vistaar purvak vivran exam ke mukhya patterns par adharit hai."}
                                        </div>
                                    </div>
                                </div>
                            ))}
                        </div>
                    </div>
                );
            };
            
            return (
                <div className="pb-10">
                    {view === 'home' && <HomeView />}
                    {view === 'sub-list' && <SubListView />}
                    {view === 'admin' && <AdminPanel />}
                    {view === 'testing' && (
                        <div className="max-w-4xl mx-auto p-4 pt-10">
                            <div className="flex justify-between items-center mb-8 glass p-6 rounded-3xl border-b-2 border-indigo-500">
                                <div>
                                    <p className="text-[10px] text-gray-500 font-bold uppercase tracking-widest mb-1">{activeTest.title}</p>
                                    <p className="font-black text-indigo-400">Q {currentIndex+1} / {currentQuestions.length}</p>
                                </div>
                                <div className={`text-3xl font-mono px-6 py-2 rounded-2xl ${timeLeft < 60 ? 'bg-red-500 animate-pulse text-white' : 'bg-white/5 text-gray-300'}`}>
                                    {Math.floor(timeLeft/60)}:{(timeLeft%60).toString().padStart(2,'0')}
                                </div>
                            </div>
                            <div className="glass p-10 rounded-[3.5rem] mb-8 min-h-[400px] flex flex-col justify-center">
                                <h2 className="text-2xl md:text-3xl font-bold leading-snug mb-12">{currentQuestions[currentIndex].text}</h2>
                                <div className="grid gap-4">
                                    {currentQuestions[currentIndex].options.map((opt, i) => (
                                        <button key={i} onClick={() => setUserAnswers({...userAnswers, [currentIndex]: i})} className={`p-6 rounded-3xl text-left border-2 transition-all flex items-center gap-5 group ${userAnswers[currentIndex] === i ? 'active-opt' : 'bg-white/5 border-white/10 hover:border-white/20'}`}>
                                            <span className="w-10 h-10 rounded-xl bg-white/5 flex items-center justify-center font-black group-hover:bg-white/10">{String.fromCharCode(65+i)}</span>
                                            <span className="font-medium">{opt}</span>
                                        </button>
                                    ))}
                                </div>
                            </div>
                            <div className="flex gap-4">
                                <button disabled={currentIndex===0} onClick={()=>setCurrentIndex(currentIndex-1)} className="flex-1 glass p-6 rounded-3xl font-bold disabled:opacity-20 transition">PREVIOUS</button>
                                {currentIndex === currentQuestions.length-1 ? 
                                    <button onClick={()=>setView('result')} className="flex-1 bg-emerald-600 p-6 rounded-3xl font-bold shadow-xl shadow-emerald-600/20">FINISH TEST</button> :
                                    <button onClick={()=>setCurrentIndex(currentIndex+1)} className="flex-1 bg-indigo-600 p-6 rounded-3xl font-bold shadow-xl shadow-indigo-600/20">SAVE & NEXT</button>
                                }
                            </div>
                        </div>
                    )}
                    {view === 'result' && <ResultView />}
                </div>
            );
        }

        const root = ReactDOM.createRoot(document.getElementById('root'));
        root.render(<App />);
    </script>
</body>
</html>
