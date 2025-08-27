# Joke-generater61
Joke generater
import React, { useEffect, useMemo, useRef, useState } from "react"; import { motion, AnimatePresence } from "framer-motion"; import { Button } from "@/components/ui/button"; import { Card, CardContent, CardHeader, CardTitle, CardFooter } from "@/components/ui/card"; import { Input } from "@/components/ui/input"; import { Label } from "@/components/ui/label"; import { Switch } from "@/components/ui/switch"; import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from "@/components/ui/select"; import { Textarea } from "@/components/ui/textarea"; import { Badge } from "@/components/ui/badge"; import { Dice5, Copy, Check, Sparkles, Heart, HeartOff, Trash2, Share2, Download, Upload, Plus, Filter } from "lucide-react";

// --- Tiny helper utils --- const clamp = (n, min, max) => Math.min(Math.max(n, min), max); const sample = (arr) => arr[Math.floor(Math.random() * arr.length)]; const byChars = (max) => (j) => j.full.length <= max;

// --- Joke seeds (clean) --- // tip: add your own jokes in-app via the "Add Joke" panel const JOKES = [ { cat: "Dad", setup: "Why did the scarecrow win an award?", punch: "Because he was outstanding in his field.", rating: 1 }, { cat: "Dad", setup: "I only know 25 letters of the alphabet.", punch: "I don't know y.", rating: 1 }, { cat: "Dad", setup: "I ordered a chicken and an egg online.", punch: "I'll let you know which comes first.", rating: 1 }, { cat: "Dad", setup: "What do you call fake spaghetti?", punch: "An impasta.", rating: 1 }, { cat: "Dad", setup: "Why don't eggs tell jokes?", punch: "They'd crack each other up.", rating: 1 },

{ cat: "Tech", setup: "Why do programmers prefer dark mode?", punch: "Because light attracts bugs.", rating: 1 }, { cat: "Tech", setup: "I changed my password to 'incorrect'.", punch: "So when I forget it, the computer says 'Your password is incorrect.'", rating: 1 }, { cat: "Tech", setup: "There are 10 types of people in the world.", punch: "Those who understand binary and those who don't.", rating: 1 }, { cat: "Tech", setup: "Why did the developer go broke?", punch: "Because he used up all his cache.", rating: 1 }, { cat: "Tech", setup: "My computer beat me at chess.", punch: "But it was no match for me at karate.", rating: 1 },

{ cat: "Puns", setup: "I stayed up all night to see where the sun went.", punch: "Then it dawned on me.", rating: 1 }, { cat: "Puns", setup: "I'm reading a book about anti-gravity.", punch: "It's impossible to put down.", rating: 1 }, { cat: "Puns", setup: "I used to be a baker.", punch: "Then I couldn't make enough dough.", rating: 1 }, { cat: "Puns", setup: "I don't trust stairs.", punch: "They're always up to something.", rating: 1 }, { cat: "Puns", setup: "I relish the fact that you've mustard the strength to ketchup to me.", punch: "Hot dog!", rating: 1 },

{ cat: "Knock‑Knock", setup: "Knock, knock. Who's there? Lettuce.", punch: "Lettuce in, it's cold out here!", rating: 1 }, { cat: "Knock‑Knock", setup: "Knock, knock. Who's there? Cow says.", punch: "Cow says who? No, cow says moooo!", rating: 1 }, { cat: "Knock‑Knock", setup: "Knock, knock. Who's there? Tank.", punch: "You're welcome.", rating: 1 },

{ cat: "One‑liners", setup: "I told my suitcases there's no vacation this year.", punch: "Now I'm dealing with emotional baggage.", rating: 1 }, { cat: "One‑liners", setup: "I used to play piano by ear.", punch: "Now I use my hands.", rating: 1 }, { cat: "One‑liners", setup: "Claustrophobic people are more productive", punch: "thinking outside the box.", rating: 1 }, { cat: "One‑liners", setup: "I have a joke about time travel.", punch: "But you didn't like it.", rating: 1 }, { cat: "One‑liners", setup: "I told a chemistry joke.", punch: "There was no reaction.", rating: 1 }, ].map(j => ({ ...j, full: ${j.setup} ${j.punch} }));

const CATS = ["All", "Dad", "Tech", "Puns", "Knock‑Knock", "One‑liners"];

export default function JokeGeneratorApp() { const [category, setCategory] = useState("All"); const [maxChars, setMaxChars] = useState(120); const [showSetupFirst, setShowSetupFirst] = useState(true); const [current, setCurrent] = useState(null); const [revealed, setRevealed] = useState(false); const [copied, setCopied] = useState(false); const [favs, setFavs] = useState([]); const [customOpen, setCustomOpen] = useState(false); const setupRef = useRef(""); const punchRef = useRef(""); const catRef = useRef("Dad");

// load & persist favorites useEffect(() => { const saved = JSON.parse(localStorage.getItem("joke-favs") || "[]"); setFavs(saved); }, []); useEffect(() => { localStorage.setItem("joke-favs", JSON.stringify(favs)); }, [favs]);

// keyboard: Space = new joke, Enter = reveal useEffect(() => { const onKey = (e) => { if (e.code === "Space") { e.preventDefault(); generate(); } if (e.code === "Enter") { e.preventDefault(); setRevealed(true); } }; window.addEventListener("keydown", onKey); return () => window.removeEventListener("keydown", onKey); }, [category, maxChars, showSetupFirst]);

const filtered = useMemo(() => { const base = category === "All" ? JOKES : JOKES.filter(j => j.cat === category); return base.filter(byChars(maxChars)); }, [category, maxChars]);

const generate = () => { const list = filtered.length ? filtered : JOKES.filter(byChars(maxChars)); const j = sample(list); setCurrent(j); setRevealed(!showSetupFirst); setCopied(false); };

const copy = async () => { if (!current) return; const text = showSetupFirst && !revealed ? current.setup : ${current.setup} ${current.punch}; try { await navigator.clipboard.writeText(text); setCopied(true); setTimeout(()=>setCopied(false), 1200); } catch {} };

const toggleFav = () => { if (!current) return; const key = current.full; setFavs(prev => prev.includes(key) ? prev.filter(x => x !== key) : [...prev, key]); };

const shareTweet = () => { if (!current) return; const text = encodeURIComponent(${current.setup} ${current.punch}); const url = https://twitter.com/intent/tweet?text=${text}; window.open(url, "_blank"); };

const exportFavs = () => { const blob = new Blob([JSON.stringify(favs, null, 2)], { type: "application/json" }); const url = URL.createObjectURL(blob); const a = document.createElement("a"); a.href = url; a.download = "my-favorite-jokes.json"; a.click(); URL.revokeObjectURL(url); };

const importFavs = (e) => { const file = e.target.files?.[0]; if (!file) return; const reader = new FileReader(); reader.onload = () => { try { const arr = JSON.parse(reader.result); if (Array.isArray(arr)) setFavs(arr.slice(0, 300)); } catch {} }; reader.readAsText(file); };

const addCustomJoke = () => { const setup = setupRef.current?.value?.trim(); const punch = punchRef.current?.value?.trim(); const cat = catRef.current || "Dad"; if (!setup || !punch) return; const j = { cat, setup, punch, rating: 1, full: ${setup} ${punch} }; JOKES.push(j); setupRef.current.value = ""; punchRef.current.value = ""; setCustomOpen(false); setCategory(cat); setCurrent(j); setRevealed(!showSetupFirst); };

return ( <div className="min-h-screen w-full bg-gradient-to-b from-white to-slate-50 dark:from-slate-900 dark:to-slate-950 text-slate-800 dark:text-slate-100"> <div className="max-w-5xl mx-auto px-4 py-8"> {/* Header */} <header className="flex items-center justify-between gap-3 mb-6"> <div className="flex items-center gap-3"> <motion.div initial={{ rotate: -10, scale: 0.9 }} animate={{ rotate: 0, scale: 1 }} transition={{ type: "spring", stiffness: 200 }} className="p-2 rounded-2xl bg-slate-100 dark:bg-slate-800"> <Sparkles className="w-6 h-6" /> </motion.div> <div> <h1 className="text-2xl sm:text-3xl font-bold leading-tight">Joke Generator</h1> <p className="text-sm opacity-70">Tap the dice, press <kbd className="px-1 py-0.5 rounded bg-slate-200 dark:bg-slate-700">Space</kbd>, or filter for the perfect quip.</p> </div> </div>

<div className="hidden sm:flex items-center gap-3">
        <Button variant="secondary" className="rounded-2xl" onClick={generate}>
          <Dice5 className="mr-2 h-4 w-4" /> Random
        </Button>
        <Button variant="outline" className="rounded-2xl" onClick={shareTweet}>
          <Share2 className="mr-2 h-4 w-4" /> Share
        </Button>
      </div>
    </header>

    {/* Controls */}
    <Card className="rounded-3xl shadow-sm mb-6">
      <CardHeader className="pb-2">
        <div className="flex items-center justify-between">
          <CardTitle className="text-base flex items-center gap-2"><Filter className="w-4 h-4"/> Filters</CardTitle>
          <div className="flex items-center gap-3">
            <div className="flex items-center gap-2">
              <Switch id="setup-first" checked={showSetupFirst} onCheckedChange={setShowSetupFirst} />
              <Label htmlFor="setup-first" className="text-sm">Show setup first</Label>
            </div>
          </div>
        </div>
      </CardHeader>
      <CardContent className="grid sm:grid-cols-3 gap-4">
        <div className="space-y-2">
          <Label>Category</Label>
          <Select value={category} onValueChange={setCategory}>
            <SelectTrigger className="rounded-2xl"><SelectValue placeholder="Pick a category"/></SelectTrigger>
            <SelectContent>
              {CATS.map(c => (<SelectItem key={c} value={c}>{c}</SelectItem>))}
            </SelectContent>
          </Select>
        </div>
        <div className="space-y-2">
          <Label>Max length: <span className="opacity-70">{maxChars} chars</span></Label>
          <input type="range" min={40} max={240} value={maxChars} onChange={(e)=>setMaxChars(parseInt(e.target.value))} className="w-full"/>
        </div>
        <div className="space-y-2">
          <Label>Add your own punchline</Label>
          <div className="flex gap-2">
            <Button className="rounded-2xl w-full" variant="outline" onClick={()=>setCustomOpen(v=>!v)}>
              <Plus className="mr-2 h-4 w-4"/>Add Joke
            </Button>
          </div>
        </div>
      </CardContent>
    </Card>

    {/* Joke Card */}
    <AnimatePresence mode="wait">
      <motion.div key={(current?.full||"none") + String(revealed)}
        initial={{ opacity: 0, y: 10 }}
        animate={{ opacity: 1, y: 0 }}
        exit={{ opacity: 0, y: -10 }}
        transition={{ duration: 0.18 }}
      >
      <Card className="rounded-3xl shadow-md">
        <CardHeader className="pb-1">
          <div className="flex items-center justify-between">
            <div className="flex items-center gap-2">
              <Badge className="rounded-xl" variant="secondary">{current?.cat || category}</Badge>
              <span className="text-xs opacity-60">{filtered.length} matches</span>
            </div>
            <div className="flex items-center gap-2">
              <Button variant="ghost" size="sm" className="rounded-2xl" onClick={toggleFav} disabled={!current}>
                {current && favs.includes(current.full) ? <Heart className="w-4 h-4 fill-current"/> : <Heart className="w-4 h-4"/>}
              </Button>
              <Button variant="ghost" size="sm" className="rounded-2xl" onClick={copy} disabled={!current}>
                {copied ? <Check className="w-4 h-4"/> : <Copy className="w-4 h-4"/>}
              </Button>
            </div>
          </div>
        </CardHeader>
        <CardContent className="text-center pt-0">
          {!current && (
            <div className="py-12">
              <p className="text-sm opacity-70 mb-3">No joke yet</p>
              <Button className="rounded-2xl" onClick={generate}><Dice5 className="mr-2 h-4 w-4"/> Roll first joke</Button>
            </div>
          )}

          {current && (
            <div className="py-6">
              <p className="text-lg sm:text-xl font-medium leading-relaxed">
                {showSetupFirst ? current.setup : `${current.setup} ${current.punch}`}
              </p>
              {showSetupFirst && (
                <AnimatePresence>
                  {revealed ? (
                    <motion.p initial={{ opacity: 0, y: -4 }} animate={{ opacity: 1, y: 0 }} exit={{ opacity: 0, y: -4 }} className="text-base sm:text-lg mt-4 opacity-90">
                      {current.punch}
                    </motion.p>
                  ) : (
                    <motion.div initial={{ opacity: 0 }} animate={{ opacity: 1 }} className="mt-4">
                      <Button className="rounded-2xl" variant="secondary" onClick={()=>setRevealed(true)}>Reveal punchline</Button>
                    </motion.div>
                  )}
                </AnimatePresence>
              )}
            </div>
          )}
        </CardContent>
        <CardFooter className="flex items-center justify-between gap-2">
          <div className="flex items-center gap-2">
            <Button variant="secondary" className="rounded-2xl" onClick={generate}><Dice5 className="mr-2 h-4 w-4"/> New</Button>
            <Button variant="outline" className="rounded-2xl" onClick={shareTweet} disabled={!current}><Share2 className="mr-2 h-4 w-4"/> Share</Button>
          </div>
          <div className="flex items-center gap-2">
            <Button size="icon" variant="ghost" className="rounded-2xl" onClick={exportFavs} title="Export favorites"><Download className="w-4 h-4"/></Button>
            <label className="cursor-pointer inline-flex items-center gap-2 text-sm opacity-80 hover:opacity-100">
              <Upload className="w-4 h-4"/>
              <span>Import</span>
              <input type="file" accept="application/json" className="hidden" onChange={importFavs} />
            </label>
          </div>
        </CardFooter>
      </Card>
      </motion.div>
    </AnimatePresence>

    {/* Favorites */}
    <section className="mt-6">
      <div className="flex items-center justify-between mb-2">
        <h2 className="text-lg font-semibold">Favorites</h2>
        <Button variant="ghost" size="sm" className="rounded-2xl" onClick={()=>setFavs([])} disabled={!favs.length}><Trash2 className="w-4 h-4 mr-2"/>Clear</Button>
      </div>
      {favs.length === 0 ? (
        <p className="text-sm opacity-70">No favorites yet. Tap the heart on jokes you love.</p>
      ) : (
        <div className="grid gap-3 sm:grid-cols-2">
          {favs.map((f, i) => (
            <Card key={i} className="rounded-2xl">
              <CardContent className="p-4 flex items-start justify-between gap-3">
                <p className="text-sm leading-relaxed">{f}</p>
                <Button variant="ghost" size="icon" className="rounded-2xl" onClick={()=>setFavs(prev=>prev.filter(x=>x!==f))}><HeartOff className="w-4 h-4"/></Button>
              </CardContent>
            </Card>
          ))}
        </div>
      )}
    </section>

    {/* Add custom joke panel */}
    <AnimatePresence>
      {customOpen && (
        <motion.div initial={{ opacity: 0, y: 8 }} animate={{ opacity: 1, y: 0 }} exit={{ opacity: 0, y: -8 }} className="mt-6">
          <Card className="rounded-3xl">
            <CardHeader>
              <CardTitle className="text-base">Add a custom joke</CardTitle>
            </CardHeader>
            <CardContent className="grid sm:grid-cols-3 gap-4">
              <div className="space-y-2">
                <Label>Category</Label>
                <Select defaultValue="Dad" onValueChange={(v)=>{ catRef.current = v; }}>
                  <SelectTrigger className="rounded-2xl"><SelectValue/></SelectTrigger>
                  <SelectContent>
                    {CATS.filter(c=>c!=="All").map(c => (<SelectItem key={c} value={c}>{c}</SelectItem>))}
                  </SelectContent>
                </Select>
              </div>
              <div className="space-y-2 sm:col-span-2">
                <Label>Setup</Label>
                <Input ref={setupRef} placeholder="e.g. Why did the tomato blush?" className="rounded-2xl"/>
              </div>
              <div className="space-y-2 sm:col-span-3">
                <Label>Punchline</Label>
                <Textarea ref={punchRef} placeholder="e.g. Because it saw the salad dressing!" className="rounded-2xl" rows={3}/>
              </div>
            </CardContent>
            <CardFooter className="justify-end gap-2">
              <Button variant="ghost" className="rounded-2xl" onClick={()=>setCustomOpen(false)}>Cancel</Button>
              <Button className="rounded-2xl" onClick={addCustomJoke}><Plus className="mr-2 h-4 w-4"/>Add</Button>
            </CardFooter>
          </Card>
        </motion.div>
      )}
    </AnimatePresence>

    {/* Footer */}
    <footer className="mt-10 text-center text-xs opacity-70">
      <p>Built with ❤️ using React, Tailwind, shadcn/ui, lucide-react & framer-motion.</p>
      <p>Pro tip: press <kbd className="px-1 py-0.5 rounded bg-slate-200 dark:bg-slate-700">Space</kbd> for a new joke, <kbd className="px-1 py-0.5 rounded bg-slate-200 dark:bg-slate-700">Enter</kbd> to reveal.</p>
    </footer>
  </div>
</div>

); }

