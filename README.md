import { QuestionLookPreview } from "@/components/QuestionLookPreview";
import { QuestionSurface, QuestionChatStrip } from "@/components/QuestionSurface";
import { QuestionTimerToggle } from "@/components/QuestionTimerToggle";
import { DrawAudioControls } from "@/components/DrawAudioControls";
import { mergeAppearanceExtras } from "@/lib/appearance-patch";
import { SiteFontPanel } from "@/components/SiteFontPanel";
import { SiteFont } from "@/components/SiteFont";
import { TextStylesStyle } from "@/components/TextStyles";
import { AnswersBanner } from "@/components/AnswersBanner";
import { TextStylesPanel } from "@/components/TextStylesPanel";
import { OrderableSections } from "@/components/OrderableSections";
import { LayoutPanel } from "@/components/LayoutPanel";
import { broadcastSectionOrder, sectionOrder } from "@/lib/site-layout";
import { FanClubStudio } from "@/components/FanClubStudio";
import { ActivityStudio } from "@/components/ActivityStudio";
import { ContestsPanel } from "@/components/ContestsPanel";
import { createFileRoute, Link, useNavigate } from "@tanstack/react-router";
import { useQueryClient } from "@tanstack/react-query";
import { useEffect, useState } from "react";
import { toast } from "sonner";
import { Trash2, Plus, Upload, MonitorPlay, LogOut, Play, SkipForward, Square, Trophy, RotateCcw, Plug, MessageCircle, Users, Eye, EyeOff, Shuffle, Save, Volume2, Crown, ShieldCheck, Type, IdCard } from "lucide-react";
import { supabase } from "@/integrations/supabase/client";
import { AdminOnly } from "@/components/AdminOnly";
import { useQuestions, useSettings, type Settings } from "@/hooks/use-admin";
import { BulkImportPanel } from "@/components/BulkImportPanel";
import { categoryIcon, categoryLabel, difficultyLabel, DIFFICULTIES, isImageCategory, type ControlCmd } from "@/lib/quiz";
import { drawSpinDelay } from "@/lib/draw-spin";
import { useControl } from "@/hooks/use-control";
import { updateControl } from "@/lib/control";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Label } from "@/components/ui/label";
import { Switch } from "@/components/ui/switch";
import { Slider } from "@/components/ui/slider";
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from "@/components/ui/select";
import { RECITERS, verseAudioUrl } from "@/lib/recitation";
import { QUIZ_FONTS, typographyClass } from "@/lib/typography";
import { ReciterPreview } from "@/components/ReciterPreview";
import { ActivityWidget } from "@/components/ActivityWidget";
import { DrawScreen } from "@/components/DrawScreen";
import { DrawWinner } from "@/components/DrawWinner";
import { StageTimer } from "@/components/StageTimer";
import { BACKGROUNDS, TIMER_SOUNDS, playTick, stageBackground, STAGE_OPTIONS, STAGE_TOGGLES, DEFAULT_EXTRAS, stageExtras } from "@/lib/stage-look";
import { useControl as useLiveStatus } from "@/hooks/use-control";
import { useCategories } from "@/hooks/use-categories";
import { CategoryVotePanel } from "@/components/CategoryVotePanel";
import { BroadcastMessagePanel } from "@/components/BroadcastMessagePanel";
import { Pencil, Palette, Timer, ChevronUp, ChevronDown } from "lucide-react";
import { PlayersPanel } from "@/components/PlayersPanel";
import { Tabs, TabsContent, TabsList, TabsTrigger } from "@/components/ui/tabs";
import { AiQuestions } from "@/components/AiQuestions";
import { NextQuestionScreen } from "@/components/NextQuestionScreen";
import { SiteRehearsal } from "@/components/SiteRehearsal";
import { EntrantsAppearance } from "@/components/EntrantsAppearance";
import { RulesPanel } from "@/components/RulesPanel";
import { OwnerQuestionPreview } from "@/components/OwnerQuestionPreview";
import { BroadcastActions } from "@/components/BroadcastActions";
import { RankingPanel } from "@/components/RankingPanel";
import { CardStudio } from "@/components/CardStudio";
import { Accordion, AccordionContent, AccordionItem, AccordionTrigger } from "@/components/ui/accordion";

export const Route = createFileRoute("/_authenticated/admin")({
  validateSearch: (search: Record<string, unknown>): { tab?: string; question?: string } => ({ ...(typeof search["tab"] === "string" ? { tab: search["tab"] } : {}), ...(typeof search["question"] === "string" ? { question: search["question"] } : {}) }),
  head: () => ({ meta: [{ title: "لوحة VIP — مسابقات هيثم عبيد" }, { name: "description", content: "لوحة المالك الخاصة لإدارة الأسئلة والخطوط وصوت التلاوات والقرعة." }, { property: "og:title", content: "لوحة VIP — مسابقات هيثم عبيد" }, { property: "og:description", content: "لوحة المالك الخاصة لإدارة الأسئلة والخطوط وصوت التلاوات والقرعة." }, { property: "og:type", content: "website" }, { name: "twitter:card", content: "summary_large_image" }, { name: "robots", content: "noindex" }] }),
  component: () => (
    <AdminOnly>
      <SiteFont />
      <TextStylesStyle />
      <Admin />
    </AdminOnly>
  ),
});

function Admin() {
  const qc = useQueryClient();
  const search = Route.useSearch();
  const [tab, setTab] = useState(search.tab ?? "control");
  useEffect(() => { if (search.tab) setTab(search.tab); }, [search.tab]);
  useEffect(() => {
    const selectTab = (event: Event) => { if (event instanceof CustomEvent && typeof event.detail === "string") setTab(event.detail); };
    window.addEventListener("site-search-tab", selectTab);
    return () => window.removeEventListener("site-search-tab", selectTab);
  }, []);
  const navigate = useNavigate();
  async function signOut() {
    await qc.cancelQueries(); qc.clear();
    await supabase.auth.signOut();
    navigate({ to: "/auth", replace: true });
  }
  return (
    <div className="vip-owner"><main className="mx-auto max-w-4xl px-4 py-8">
      <header className="vip-heading mb-6 flex flex-wrap items-center justify-between gap-4 pb-6">
        <div><div className="mb-2 flex items-center gap-2 text-gold"><Crown className="size-6" /><span className="text-sm font-bold">VIP · هيثم عبيد</span></div><h1 className="text-2xl font-bold">لوحة التحكم الخاصة</h1><p className="mt-2 flex items-center gap-1 text-xs text-muted-foreground"><ShieldCheck className="size-4" />لصاحب البث فقط</p></div>
        <div className="flex flex-wrap items-center gap-2">
          <LiveBadge />
          <div className="flex flex-col gap-2"><Button asChild><a href="/live" target="_blank" rel="noopener noreferrer"><MonitorPlay />شاشة البث</a></Button><Button onClick={() => setTab("control")}><MonitorPlay />التحكم بالبث</Button></div>
          <Button variant="secondary" size="icon" aria-label="تسجيل الخروج" title="تسجيل الخروج" onClick={signOut}><LogOut /></Button>
        </div>
      </header>
      <Accordion type="single" collapsible className="mb-4 rounded-lg border px-4"><AccordionItem value="fonts" className="border-0"><AccordionTrigger className="text-lg font-bold text-gold">خطوط الموقع و البث</AccordionTrigger><AccordionContent><TextStylesPanel /></AccordionContent></AccordionItem></Accordion>
      <Tabs value={tab} onValueChange={setTab}>
        <TabsList className="mb-4 flex h-auto flex-wrap"><TabsTrigger value="promotions" className="promo-tab">أنواع المسابقات</TabsTrigger><TabsTrigger value="rehearsal">تجربة الموقع</TabsTrigger><TabsTrigger value="edit-all">التعديل الشامل</TabsTrigger><TabsTrigger value="questions">الأسئلة</TabsTrigger><TabsTrigger value="ai">أسئلة بالذكاء الاصطناعي</TabsTrigger><TabsTrigger value="players">المشتركون</TabsTrigger><TabsTrigger value="ranking">شاشة الترتيب المباشر</TabsTrigger><TabsTrigger value="look">شكل الموقع</TabsTrigger><TabsTrigger value="studio">استوديو البث</TabsTrigger><TabsTrigger value="stage">إعدادات شاشة البث</TabsTrigger><TabsTrigger value="settings">الإعدادات</TabsTrigger></TabsList>
        <TabsContent value="rehearsal"><SiteRehearsal /></TabsContent>
        <TabsContent value="promotions"><ContestsPanel /></TabsContent>
        <TabsContent value="edit-all"><AllEditingPanel /></TabsContent>
        <TabsContent value="control"><ControlPanel /></TabsContent>
        <TabsContent value="questions"><QuestionsPanel /></TabsContent>
        <TabsContent value="ai"><AiQuestions /></TabsContent>
        <TabsContent value="players"><PlayersPanel /></TabsContent>
        <TabsContent value="ranking"><RankingPanel /></TabsContent>
        <TabsContent value="look"><LookPanel /></TabsContent>
        <TabsContent value="studio"><StudioPanel /></TabsContent>
        <TabsContent value="stage"><LayoutPanel /><StageScreenPanel /></TabsContent>
        <TabsContent value="settings"><SettingsPanel /></TabsContent>
      </Tabs>
    </main></div>
  );
}

function StudioPanel() {
  return <section aria-label="استوديو البث" className="space-y-4">
    <h2 className="border-b pb-4 text-xl font-bold text-gold">استوديو البث — كل الإعدادات في مكان واحد</h2>
    <Accordion type="multiple" defaultValue={["stage"]}>
      <AccordionItem value="font"><AccordionTrigger>خط الموقع</AccordionTrigger><AccordionContent><SiteFontPanel /></AccordionContent></AccordionItem>
      <AccordionItem value="layout"><AccordionTrigger>أماكن عناصر شاشة البث</AccordionTrigger><AccordionContent><LayoutPanel /></AccordionContent></AccordionItem>
      <AccordionItem value="stage"><AccordionTrigger>شاشة البث والمؤقت والقرعة</AccordionTrigger><AccordionContent><StageScreenPanel /></AccordionContent></AccordionItem>
      <AccordionItem value="entrants"><AccordionTrigger>شكل المشتركين</AccordionTrigger><AccordionContent><EntrantsAppearance /></AccordionContent></AccordionItem>
      <AccordionItem value="players"><AccordionTrigger>الرتب والدخول الفخم</AccordionTrigger><AccordionContent><PlayersPanel /></AccordionContent></AccordionItem>
    </Accordion>
  </section>;
}

function AllEditingPanel() {
  return <section aria-label="التعديل الشامل" className="space-y-4">
    <h2 className="border-b pb-4 text-xl font-bold text-gold">التعديل الشامل</h2>
    <Accordion type="single" collapsible defaultValue="identity">
      <AccordionItem value="identity"><AccordionTrigger>العنوان وحساب TikTok والنصوص والخطوط والنقاط والقرعة</AccordionTrigger><AccordionContent><SettingsPanel /></AccordionContent></AccordionItem>
      <AccordionItem value="appearance"><AccordionTrigger>الخلفيات والألوان والأصوات والحركات وشاشة السؤال التالي</AccordionTrigger><AccordionContent><LookPanel /></AccordionContent></AccordionItem>
      <AccordionItem value="content"><AccordionTrigger>الفقرات والأسئلة والإجابات والصور والتلاوات</AccordionTrigger><AccordionContent><QuestionsPanel /></AccordionContent></AccordionItem>
      <AccordionItem value="live"><AccordionTrigger>المؤقت والانضمام للمسابقة وإدارة البث والترتيب</AccordionTrigger><AccordionContent><ControlPanel /></AccordionContent></AccordionItem>
      <AccordionItem value="ai"><AccordionTrigger>إنشاء أسئلة جديدة بالذكاء الاصطناعي</AccordionTrigger><AccordionContent><AiQuestions /></AccordionContent></AccordionItem>
    </Accordion>
  </section>;
}

function ControlPanel() {
  const { data: settings } = useSettings();
  const { data: control } = useControl();
  const { data: questions = [] } = useQuestions();
  const categories = useCategories();
  const qc = useQueryClient();
  const [busy, setBusy] = useState(false);
  const [order, setOrder] = useState(() => broadcastSectionOrder([]));
  const [orderDirty, setOrderDirty] = useState(false);
  useEffect(() => { if (settings && !orderDirty) setOrder(broadcastSectionOrder(stageExtras(settings.stage_extras).layout.order)); }, [settings, orderDirty]);
  async function saveOrder() {
    setBusy(true);
    try {
      const fresh = await supabase.from("settings").select("stage_extras").eq("id", 1).single(); if (fresh.error || !fresh.data) throw fresh.error;
      const fx = stageExtras(fresh.data.stage_extras);
      const result = await supabase.from("settings").update({ stage_extras: { ...fx, layout: { ...fx.layout, order: sectionOrder(order) } } as Settings["stage_extras"] }).eq("id", 1);
      if (result.error) throw result.error;
      setOrderDirty(false); await qc.invalidateQueries({ queryKey: ["settings"] }); toast.success("تم حفظ ترتيب الأقسام");
    } catch { toast.error("تعذر حفظ الترتيب"); } finally { setBusy(false); }
  }
  const [entryWord, setEntryWord] = useState("");
  useEffect(() => { if (control) setEntryWord(control.entry_word); }, [control?.entry_word]);
  const [cardWord, setCardWord] = useState("");
  useEffect(() => { if (control) setCardWord(control.card_word); }, [control?.card_word]);
  async function registration(patch: Parameters<typeof updateControl>[0]) {
    setBusy(true);
    try { await updateControl(patch); await qc.invalidateQueries({ queryKey: ["quiz-control"] }); }
    catch { toast.error("تعذر حفظ إعدادات الاشتراك"); }
    finally { setBusy(false); }
  }
  const category = settings?.active_category ?? "all";
  const alive = !!control?.stage_seen_at && Date.now() - new Date(control.stage_seen_at).getTime() < 6000;
  async function choose(category: string) {
    const { error } = await supabase.from("settings").update({ active_category: category }).eq("id", 1);
    if (error) { toast.error(error.message); return; }
    await qc.invalidateQueries({ queryKey: ["settings"] });
  }
  const level = settings?.active_difficulty ?? "all";
  async function chooseLevel(value: string) {
    const { error } = await supabase.from("settings").update({ active_difficulty: value }).eq("id", 1);
    if (error) { toast.error(error.message); return; }
    await qc.invalidateQueries({ queryKey: ["settings"] });
  }
  async function sound(patch: { verse_audio?: boolean; verse_reciter?: string }) {
    setBusy(true);
    try {
      const { error } = await supabase.from("settings").update(patch).eq("id", 1);
      if (error) throw error;
      await qc.invalidateQueries({ queryKey: ["settings"] });
    } catch { toast.error("تعذر حفظ صوت التلاوة"); }
    finally { setBusy(false); }
  }
  async function send(command: ControlCmd) {
    if (!alive) { toast.error("افتح شاشة البث أولًا واتركها مفتوحة"); return; }
    if (command === "reset" && !window.confirm("هل تريد مسح نقاط جميع المشاركين؟")) return;
    setBusy(true);
    try {
      await updateControl({ command_id: crypto.randomUUID(), command, category, ...(command === "start" || command === "next" ? { show_entrants: false, show_rules: false } : {}) });
    } catch { toast.error("تعذر إرسال الأمر؛ حاول مجددًا"); }
    finally { setBusy(false); }
  }
  return <section className="space-y-6">
    <div className="flex flex-wrap items-center gap-3 border-b pb-4 text-sm">
      <span className={alive ? "text-primary" : "text-muted-foreground"}>{alive ? "شاشة البث جاهزة" : "شاشة البث مغلقة"}</span>
      <div className="flex flex-col items-start gap-2">
        <LiveBadge />
        <Button size="sm" disabled={busy || !alive} onClick={() => send("connect")}><Plug />اتصال بالبث</Button>
      </div>
      {control?.message && <p className="w-full break-words text-muted-foreground">{control.message}</p>}
    </div>
    <OrderableSections order={order} onChange={v => { setOrder(v); setOrderDirty(true); }} onSave={saveOrder} busy={busy}>
    <div data-section="rules"><RulesPanel /></div>
    <div data-section="question"><OwnerQuestionPreview question={questions.find(q => q.id === control?.prepared_question_id)} phase={control?.phase} alive={alive} /></div>
    <div data-section="actions"><BroadcastActions disabled={busy || !alive} canDraw={control?.phase === "stats"} onCommand={send} onDraw={() => registration({ draw_id: crypto.randomUUID(), show_entrants: false })} /></div>
    <div data-section="timer"><TimerBox alive={alive} /></div>
    <div data-section="entry" className="space-y-4 border-b pb-6">
      <h2 className="flex items-center gap-2 font-bold"><Users className="size-5" />دخول المسابقة</h2>
      <Label htmlFor="entry-word">للانضمام إلى المسابقة اكتب في التعليقات</Label>
      <div className="flex gap-2"><Input id="entry-word" maxLength={80} value={entryWord} onChange={(e) => setEntryWord(e.target.value)} /><Button disabled={busy || !entryWord.trim()} onClick={() => registration({ entry_word: entryWord.trim() })}><Save />حفظ الكلمة</Button></div>
      <div className="flex flex-wrap items-center gap-3">
        <Switch id="registration-open" disabled={busy || !control?.entry_word.trim()} checked={control?.registration_open ?? false} onCheckedChange={(value) => registration({ registration_open: value })} />
        <Label htmlFor="registration-open">فتح التسجيل</Label>
        <span className="text-sm text-primary">{Object.keys(control?.entrants ?? {}).length} مشترك</span>
      </div>
      <div className="flex flex-wrap items-center gap-3">
        <Switch id="answers-without-entry" disabled={busy || !control} checked={control?.allow_answers_without_entry ?? false} onCheckedChange={(value) => registration({ allow_answers_without_entry: value })} />
        <Label htmlFor="answers-without-entry">السماح بالإجابة دون كلمة الدخول</Label>
      </div>
      <Button variant="secondary" disabled={busy} onClick={() => registration({ show_entrants: !control?.show_entrants, show_rules: false })}>{control?.show_entrants ? <EyeOff /> : <Eye />}{control?.show_entrants ? "إخفاء شاشة المشتركين" : "عرض شاشة المشتركين"}</Button>
      <EntrantsAppearance />
    </div>
    <div data-section="card" className="space-y-3 border-b pb-6">
      <h2 className="flex items-center gap-2 font-bold"><IdCard className="size-5" />بطاقة المشترك عند الطلب</h2>
      <p className="text-sm text-muted-foreground">يكتب المشترك كلمة الطلب في التعليقات فتظهر بطاقته على شاشة البث: نقاطه وترتيبه ومستواه ورتبته.</p>
      <Label htmlFor="card-word">كلمة طلب البطاقة</Label>
      <div className="flex gap-2"><Input id="card-word" maxLength={40} value={cardWord} onChange={(e) => setCardWord(e.target.value)} placeholder="بطاقتي" /><Button disabled={busy} onClick={() => registration({ card_word: cardWord.trim() })}><Save />حفظ الكلمة</Button></div>
      <div className="flex flex-wrap items-center gap-3">
        <Label htmlFor="card-seconds">مدة ظهور البطاقة</Label>
        <Select value={String(control?.card_seconds ?? 8)} disabled={busy} onValueChange={(value) => registration({ card_seconds: Number(value) })}>
          <SelectTrigger id="card-seconds" className="w-28"><SelectValue /></SelectTrigger>
          <SelectContent>{[5, 8, 12, 15, 20].map((s) => <SelectItem key={s} value={String(s)}>{s} ثانية</SelectItem>)}</SelectContent>
        </Select>
        <span className="text-xs text-muted-foreground">اترك الكلمة فارغة لإيقاف البطاقات.</span>
      </div>
      <CardStudio />
    </div>
    <div data-section="audio" className="space-y-3 border-b pb-6">
      <h2 className="flex items-center gap-2 font-bold"><Volume2 className="size-5" />صوت التلاوات</h2>
      <p role="status" className={`text-sm ${control?.audio_state === "playing" ? "text-primary" : "text-muted-foreground"}`}>{({ muted: "الصوت مكتوم", ready: "الصوت جاهز", playing: "التلاوة تعمل الآن", blocked: "المتصفح منع الصوت؛ اضغط مرة داخل شاشة البث ثم أعد التلاوة", error: "تعذر تحميل الصوت؛ أعد التلاوة أو اختر قارئًا آخر", missing: "لا يوجد تسجيل صالح لهذا السؤال مع القارئ المختار" } as const)[control?.audio_state ?? "muted"]}</p>
      <div className="flex items-center gap-3"><Switch id="live-verse-audio" checked={settings?.verse_audio ?? false} disabled={busy} onCheckedChange={(value) => sound({ verse_audio: value })} /><Label htmlFor="live-verse-audio">تشغيل صوت التلاوة</Label></div>
      <Label htmlFor="live-reciter">القارئ</Label>
      <Select value={settings?.verse_reciter ?? "original"} disabled={busy} onValueChange={(value) => sound({ verse_reciter: value })}><SelectTrigger id="live-reciter"><SelectValue /></SelectTrigger><SelectContent>{RECITERS.map((r) => <SelectItem key={r.id} value={r.id}>{r.label}</SelectItem>)}</SelectContent></Select>
      <Button variant="secondary" disabled={busy || !alive || control?.phase !== "running" || !settings?.verse_audio} onClick={() => registration({ recitation_id: crypto.randomUUID() })}><RotateCcw />إعادة التلاوة</Button>
      <ReciterPreview />
    </div>
    <div data-section="categories" className="space-y-4">
    <Button asChild variant="secondary"><a href="/leaderboard" target="_blank" rel="noopener noreferrer"><Trophy />شاشة الترتيب المباشر</a></Button>
    <div className="flex flex-wrap gap-2">{[{ id: "all", label: "كل الفقرات", picture: false }, ...categories].map((c) => <Button key={c.id} variant={category === c.id ? "default" : "secondary"} onClick={() => choose(c.id)}>{categoryIcon(c.id, c.picture)} {c.label} ({questions.filter((q) => q.active && (c.id === "all" || q.category === c.id)).length})</Button>)}</div>
    </div>
    <div data-section="vote"><CategoryVotePanel categories={categories} control={control as Record<string, unknown> | null | undefined} alive={alive} onWinner={choose} /></div>
    <div data-section="broadcast-message"><BroadcastMessagePanel alive={alive} /></div>
    <section data-section="question-options" aria-label="مستوى الأسئلة والقرعة والنقاط" className="space-y-4 border-y border-primary/40 py-5">
    <div className="grid grid-cols-[minmax(0,1fr)_auto_minmax(0,1fr)] items-end gap-2">
    <div className="min-w-0 space-y-2"><Label htmlFor="broadcast-difficulty">مستوى الأسئلة في البث</Label>
    <Select value={level} onValueChange={chooseLevel}><SelectTrigger id="broadcast-difficulty" className="w-full min-w-0 text-xs sm:text-sm"><SelectValue /></SelectTrigger><SelectContent>{[{ id: "all", label: "كل المستويات", dot: "" }, ...DIFFICULTIES].map(d => <SelectItem key={d.id} value={d.id}>{d.dot} {d.label} ({questions.filter(q => q.active && (category === "all" || q.category === category) && (d.id === "all" || q.difficulty === d.id)).length})</SelectItem>)}</SelectContent></Select></div>
    <Button className="px-2 text-xs sm:px-4 sm:text-sm" disabled={busy || !alive} variant="destructive" onClick={() => send("reset")}><RotateCcw className="hidden sm:block" />تصفير النقاط</Button>
    <div className="min-w-0 space-y-2"><Label htmlFor="broadcast-auto-draw">قرعة تلقائية</Label>
    <Select value={String(control?.auto_draw_seconds ?? 0)} disabled={busy} onValueChange={(v) => registration({ auto_draw_seconds: Number(v) })}>
      <SelectTrigger id="broadcast-auto-draw" aria-label="القرعة التلقائية" className="w-full min-w-0 text-xs sm:text-sm"><SelectValue /></SelectTrigger>
      <SelectContent><SelectItem value="0">يدويًا</SelectItem>{[3, 5, 8, 10, 15, 20, 30].map((n) => <SelectItem key={n} value={String(n)}>بعد {n} ث</SelectItem>)}</SelectContent>
    </Select></div>
    </div>
    <p className="text-xs text-muted-foreground">{level === "all" ? "سيختار البث أسئلة من كل المستويات" : `سيختار البث أسئلة ${difficultyLabel(level).label === "سهل" ? "سهلة" : difficultyLabel(level).label === "صعب" ? "صعبة" : "متوسطة"} فقط`}</p>
    {control?.phase === "waiting" && <p role="status" className="text-sm text-gold">شاشة «السؤال التالي» ظاهرة — اضغط «بدء السؤال» عندما تكون جاهزًا.</p>}
    </section></OrderableSections>
  </section>;
}

function TimerBox({ alive }: { alive: boolean }) {
  const { data: control } = useControl();
  const qc = useQueryClient();
  const [busy, setBusy] = useState(false);
  const [secs, setSecs] = useState(25);
  useEffect(() => { setSecs(control?.fixed_seconds ?? 25); }, [control?.fixed_seconds]);
  const running = control?.phase === "running";
  async function write(patch: Parameters<typeof updateControl>[0]) {
    setBusy(true);
    try { await updateControl(patch); await qc.invalidateQueries({ queryKey: ["quiz-control"] }); }
    catch { toast.error("تعذر إرسال أمر المؤقت"); }
    finally { setBusy(false); }
  }
  // One timer for every section: the arrows change the duration, and the running question follows.
  function change(n: number) {
    const v = Math.min(600, Math.max(3, Math.round(n) || 3));
    setSecs(v);
    const patch: Parameters<typeof updateControl>[0] = { fixed_seconds: v };
    if (running && alive) Object.assign(patch, { timer_id: crypto.randomUUID(), timer_action: "add", timer_value: v - secs });
    void write(patch);
  }
  return <div className="space-y-4 rounded-lg border p-4">
    <div className="flex flex-wrap items-center justify-between gap-2">
      <h2 className="flex items-center gap-2 font-bold"><Timer className="size-5" />مؤقت السؤال (لكل الفقرات)</h2>
      <span role="status" className={`text-sm tabular-nums ${control?.timer_disabled ? "text-primary" : "text-muted-foreground"}`}>{control?.timer_disabled ? "المؤقت متوقف" : running ? `المتبقي الآن: ${control?.time_left ?? "—"} ث${control?.timer_paused ? " (متوقف)" : ""}` : "لا يوجد سؤال جارٍ"}</span>
    </div>
    <div className="flex flex-wrap items-center justify-center gap-3">
      <Button size="icon" variant="secondary" aria-label="إنقاص الوقت" disabled={busy || secs <= 3} onClick={() => change(secs - 1)}><ChevronDown /></Button>
      <Input aria-label="مدة السؤال بالثواني" type="number" min={3} max={600} className="h-14 w-28 text-center text-3xl font-bold tabular-nums" value={secs} onChange={(e) => setSecs(+e.target.value)} onBlur={() => change(secs)} />
      <Button size="icon" variant="secondary" aria-label="زيادة الوقت" disabled={busy || secs >= 600} onClick={() => change(secs + 1)}><ChevronUp /></Button>
      <span className="font-bold">ثانية</span>
      <QuestionTimerToggle checked={control?.timer_disabled ?? false} disabled={busy || !control} onChange={(value) => void write({ timer_disabled: value })} />
    </div>
  </div>;
}

function LiveBadge() {
  const { data: control } = useLiveStatus();
  const { data: settings } = useSettings();
  const on = !!control?.connected;
  const user = settings?.tiktok_username?.replace(/^@/, "") || "بدون حساب";
  return <div role="status" className="flex items-center gap-2 rounded-full border px-3 py-1.5 text-sm" title={on ? "متصل بالبث المباشر" : "غير متصل بالبث"}>
    <span className={`size-3 rounded-full ${on ? "bg-primary animate-pulse" : "bg-destructive"}`} />
    <span dir="ltr" className="font-bold">@{user}</span>
    <span className={on ? "text-primary" : "text-muted-foreground"}>{on ? "مباشر" : "غير متصل"}</span>
  </div>;
}

function LookPanel() {
  const { data } = useSettings();
  const qc = useQueryClient();
  const [s, setS] = useState<Settings | null>(null);
  const [baseline, setBaseline] = useState<Settings | null>(null);
  const [dirty, setDirty] = useState(false);
  const [saving, setSaving] = useState(false);
  const [uploading, setUploading] = useState(false);
  useEffect(() => { if (data && !dirty) { setS(data); setBaseline(data); } }, [data, dirty]);
  if (!s) return null;
  const set = (patch: Partial<Settings>) => { setDirty(true); setS({ ...s, ...patch }); };
  const fx = stageExtras(s.stage_extras);
  async function uploadBg(file: File) {
    setUploading(true);
    const path = `bg-${crypto.randomUUID()}-${file.name.replace(/[^\w.]/g, "")}`;
    const { error } = await supabase.storage.from("quiz-images").upload(path, file);
    if (error) { setUploading(false); toast.error(error.message); return; }
    const { data: u } = await supabase.storage.from("quiz-images").createSignedUrl(path, 60 * 60 * 24 * 365 * 10);
    set({ background_image_url: u?.signedUrl ?? null, stage_background: "image" });
    setUploading(false);
  }
  async function save() {
    if (!s) return;
    if (s.stage_background === "image" && !s.background_image_url) { toast.error("ارفع صورة الخلفية أو اختر خلفية جاهزة"); return; }
    setSaving(true);
    try {
    const { data: fresh, error: readError } = await supabase.from("settings").select("stage_extras").eq("id", 1).single();
    if (readError || !fresh) throw readError ?? new Error("لم يتم العثور على الإعدادات");
    const { timer_sound, timer_volume, warning_seconds, end_sound, stage_background, background_image_url, background_dim, panel_opacity, show_title, show_counters, show_bottom_text, show_correct_list, accent_hue, stage_extras } = s;
    const { error } = await supabase.from("settings").update({ timer_sound, timer_volume, warning_seconds, end_sound, stage_background, background_image_url, background_dim, panel_opacity, show_title, show_counters, show_bottom_text, show_correct_list, accent_hue, stage_extras: mergeAppearanceExtras(fresh.stage_extras, baseline?.stage_extras, stage_extras) as Settings["stage_extras"] }).eq("id", 1).select("id").single();
    if (error) throw error;
    toast.success("تم حفظ شكل الموقع");
    setDirty(false);
    await qc.invalidateQueries({ queryKey: ["settings"] });
    } catch (error) { toast.error("تعذر حفظ الشكل: " + (error instanceof Error ? error.message : (error as { message?: string })?.message ?? "تحقق من الاتصال وحاول مجددًا")); } finally { setSaving(false); }
  }
  const toggles = [["show_title", "إظهار عنوان المسابقة"], ["show_counters", "إظهار عدادات المجيبين والمشاركين"], ["show_bottom_text", "إظهار النص أسفل السؤال"], ["show_correct_list", "إظهار المجيبين الصحيحين أسفل الشاشة"]] as const;
  return <div className="space-y-6 py-2">
    <DrawAudioControls value={fx.drawAudio} selectedStyle={fx.drawStyle} onChange={drawAudio => set({ stage_extras: { ...fx, drawAudio } as Settings["stage_extras"] })} />
    <section className="space-y-4 border-b pb-6">
      <h2 className="flex items-center gap-2 font-bold text-gold"><Volume2 className="size-5" />صوت المؤقت</h2>
      <Field label="نوع الصوت"><Select value={s.timer_sound} onValueChange={(v) => set({ timer_sound: v })}><SelectTrigger aria-label="نوع صوت المؤقت"><SelectValue /></SelectTrigger><SelectContent>{TIMER_SOUNDS.map((t) => <SelectItem key={t.id} value={t.id}>{t.label}</SelectItem>)}</SelectContent></Select></Field>
      <Field label={`مستوى الصوت: ${s.timer_volume}%`}><Slider min={0} max={100} step={5} value={[s.timer_volume]} onValueChange={([v = 0]) => set({ timer_volume: v })} /></Field>
      <Field label="ثواني التحذير الأخيرة (صوت أعلى)"><Input type="number" min={0} max={30} value={s.warning_seconds} onChange={(e) => set({ warning_seconds: Math.min(30, Math.max(0, +e.target.value)) })} /></Field>
      <div className="flex items-center gap-3"><Switch id="end-sound" checked={s.end_sound} onCheckedChange={(v) => set({ end_sound: v })} /><Label htmlFor="end-sound">صوت عند انتهاء الوقت</Label></div>
      <div className="flex flex-wrap gap-2">
        <Button variant="secondary" onClick={() => playTick(s.timer_sound, s.timer_volume, false)}>تجربة الثانية</Button>
        <Button variant="secondary" onClick={() => playTick(s.timer_sound, s.timer_volume, true)}>تجربة التحذير</Button>
        <Button variant="secondary" onClick={() => playTick(s.timer_sound, s.timer_volume, true, true)}>تجربة النهاية</Button>
      </div>
    </section>
    <section className="space-y-4 border-b pb-6">
      <h2 className="flex items-center gap-2 font-bold text-gold"><Palette className="size-5" />الخلفية خلف الأسئلة</h2>
      <div className="grid grid-cols-2 gap-3 sm:grid-cols-3">
        {BACKGROUNDS.map((b) => <button key={b.id} type="button" aria-pressed={s.stage_background === b.id} onClick={() => set({ stage_background: b.id })} className={`h-20 rounded-lg border-2 p-2 text-sm font-bold ${s.stage_background === b.id ? "border-primary" : "border-transparent"}`} style={stageBackground(b.id, b.id === "image" ? s.background_image_url : null, s.background_dim)}>{b.label}</button>)}
      </div>
      <label className="flex h-9 cursor-pointer items-center justify-center gap-2 rounded-md border border-input text-sm"><Upload className="size-4" />{uploading ? "جارٍ الرفع…" : "رفع صورة خلفية خاصة"}<input type="file" accept="image/*" hidden onChange={(e) => e.target.files?.[0] && uploadBg(e.target.files[0])} /></label>
      <Field label={`تعتيم الصورة: ${s.background_dim}%`}><Slider min={0} max={90} step={5} value={[s.background_dim]} onValueChange={([v = 0]) => set({ background_dim: v })} /></Field>
      <Field label={`وضوح بطاقة السؤال: ${s.panel_opacity}%`}><Slider min={20} max={100} step={5} value={[s.panel_opacity]} onValueChange={([v = 70]) => set({ panel_opacity: v })} /></Field>
      <Field label="لون الموقع"><Slider min={0} max={360} step={1} value={[s.accent_hue]} onValueChange={([v = 0]) => { set({ accent_hue: v }); document.documentElement.style.setProperty("--hue", String(v)); }} /></Field>
      <div className="rounded-lg p-6" style={stageBackground(s.stage_background, s.background_image_url, s.background_dim)}>
        <QuestionLookPreview look={fx.questionLook} questionFont={s.question_font} verseFont={s.verse_font} />
      </div>
    </section>
    <section className="space-y-3">
      <h2 className="font-bold text-gold">عناصر الشاشة</h2>
      {toggles.map(([key, label]) => <div key={key} className="flex items-center gap-3"><Switch id={key} checked={s[key]} onCheckedChange={(v) => set({ [key]: v })} /><Label htmlFor={key}>{label}</Label></div>)}
    </section>
    <section className="space-y-4 border-t pt-6">
      <h2 className="flex items-center gap-2 font-bold text-gold"><Palette className="size-5" />خيارات مميزة</h2>
      <div className="grid gap-4 sm:grid-cols-2">
        {(Object.entries(STAGE_OPTIONS) as [keyof typeof STAGE_OPTIONS, (typeof STAGE_OPTIONS)[keyof typeof STAGE_OPTIONS]][]).filter(([key]) => !key.startsWith("wait") && !key.startsWith("fan") && !key.startsWith("entrants") && !key.startsWith("timer")).map(([key, def]) => <Field key={key} label={def.label}><Select value={fx[key]} onValueChange={(v) => set({ stage_extras: { ...fx, [key]: v } as Settings["stage_extras"] })}><SelectTrigger aria-label={def.label}><SelectValue /></SelectTrigger><SelectContent>{def.options.map(([v, l]) => <SelectItem key={v} value={v}>{l}</SelectItem>)}</SelectContent></Select></Field>)}
        <Field label={`ضبابية التأثيرات المتحركة: ${fx.particlesBlur}%`}><Slider aria-label="ضبابية التأثيرات المتحركة" min={0} max={100} step={5} value={[fx.particlesBlur]} onValueChange={([v = 0]) => set({ stage_extras: { ...fx, particlesBlur: v } as Settings["stage_extras"] })} /></Field>
        <Field label={`شفافية خلفية الإشعارات: ${100 - fx.activityOpacity}%`}><Slider aria-label="شفافية خلفية الإشعارات" min={0} max={100} step={5} value={[100 - fx.activityOpacity]} onValueChange={([v = 65]) => set({ stage_extras: { ...fx, activityOpacity: 100 - v } as Settings["stage_extras"] })} /></Field>
      </div>
      {STAGE_TOGGLES.map(([key, label]) => <div key={key} className="flex items-center gap-3"><Switch id={`fx-${key}`} checked={fx[key]} onCheckedChange={(v) => set({ stage_extras: { ...fx, [key]: v } as Settings["stage_extras"] })} /><Label htmlFor={`fx-${key}`}>{label}</Label></div>)}
      <Button variant="secondary" onClick={() => set({ stage_extras: DEFAULT_EXTRAS as Settings["stage_extras"] })}><RotateCcw />استعادة الخيارات الافتراضية</Button>
    </section>
    <section className="space-y-4 border-t pt-6">
      <h2 className="flex items-center gap-2 font-bold text-gold"><SkipForward className="size-5" />شاشة السؤال التالي</h2>
      <div className="grid gap-4 sm:grid-cols-2">
        {(Object.entries(STAGE_OPTIONS) as [keyof typeof STAGE_OPTIONS, (typeof STAGE_OPTIONS)[keyof typeof STAGE_OPTIONS]][]).filter(([key]) => key.startsWith("wait")).map(([key, def]) => <Field key={key} label={def.label}><Select value={fx[key]} onValueChange={(v) => set({ stage_extras: { ...fx, [key]: v } as Settings["stage_extras"] })}><SelectTrigger aria-label={def.label}><SelectValue /></SelectTrigger><SelectContent>{def.options.map(([v, l]) => <SelectItem key={v} value={v}>{l}</SelectItem>)}</SelectContent></Select></Field>)}
      </div>
      <div className="overflow-hidden rounded-lg" style={stageBackground(s.stage_background, s.background_image_url, s.background_dim)}><NextQuestionScreen extras={fx} /></div>
    </section>
    <Button disabled={saving || uploading} onClick={save}><Save />{saving ? "جارٍ الحفظ…" : "حفظ شكل الموقع"}</Button>
  </div>;
}

const DEMO_DRAW = ["أحمد", "سارة", "محمد", "ليلى", "يوسف", "مريم", "علي", "هدى"].map((n, i) => ({ uniqueId: `d${i}`, nickname: n }));

function StageScreenPanel() {
  const { data } = useSettings();
  const qc = useQueryClient();
  const [s, setS] = useState<Settings | null>(null);
  const [baseline, setBaseline] = useState<Settings | null>(null);
  const [dirty, setDirty] = useState(false);
  const [demo, setDemo] = useState(18);
  useEffect(() => { if (data && !dirty) { setS(data); setBaseline(data); } }, [data, dirty]);
  const [previewSecs, setPreviewSecs] = useState(4);
  const [drawIdx, setDrawIdx] = useState(0);
  const [drawing, setDrawing] = useState(false);
  const [drawFinished, setDrawFinished] = useState(false);
  useEffect(() => {
    if (!drawing) return;
    const drawMs = previewSecs * 1000;
    const started = Date.now();
    let spinTimer: ReturnType<typeof setTimeout> | undefined;
    const spin = () => {
      setDrawIdx((i) => i + 1);
      const elapsed = Date.now() - started;
      if (elapsed < drawMs) spinTimer = setTimeout(spin, drawSpinDelay(elapsed, drawMs));
    };
    spin();
    const done = setTimeout(() => { setDrawing(false); setDrawFinished(true); }, drawMs);
    return () => { if (spinTimer) clearTimeout(spinTimer); clearTimeout(done); };
  }, [drawing, previewSecs]);
  if (!s) return null;
  const fx = stageExtras(s.stage_extras);
  const setFx = (patch: Record<string, string | boolean | number>) => { setDirty(true); setS({ ...s, stage_extras: { ...fx, ...patch } as Settings["stage_extras"] }); };
  const set = (patch: Partial<Settings>) => { setDirty(true); setS({ ...s, ...patch }); };
  async function save() {
    if (!s) return;
    const { data: fresh, error: readError } = await supabase.from("settings").select("stage_extras").eq("id", 1).single();
    if (readError || !fresh) { toast.error("تعذر قراءة إعدادات الشكل؛ حاول مجددًا"); return; }
    const { stage_extras, show_title, show_counters, show_bottom_text, show_correct_list, title, bottom_text, panel_opacity } = s;
    const { error } = await supabase.from("settings").update({ stage_extras: mergeAppearanceExtras(fresh.stage_extras, baseline?.stage_extras, stage_extras) as Settings["stage_extras"], show_title, show_counters, show_bottom_text, show_correct_list, title, bottom_text, panel_opacity }).eq("id", 1).select("id").single();
    if (error) { toast.error("تعذر حفظ إعدادات شاشة البث"); return; }
    setDirty(false);
    toast.success("تم حفظ إعدادات شاشة البث");
    qc.invalidateQueries({ queryKey: ["settings"] });
  }
  const opts = (Object.entries(STAGE_OPTIONS) as [keyof typeof STAGE_OPTIONS, (typeof STAGE_OPTIONS)[keyof typeof STAGE_OPTIONS]][]).filter(([k]) => !k.startsWith("wait") && !k.startsWith("entrants") && !k.startsWith("activity"));
  const toggles = [["show_title", "إظهار عنوان المسابقة"], ["show_counters", "إظهار عدادات المجيبين والمشاركين"], ["show_bottom_text", "إظهار النص أسفل السؤال"], ["show_correct_list", "إظهار المجيبين الصحيحين"]] as const;
  const warn = fx.warnPulse && demo <= s.warning_seconds;
  return <div className="space-y-6 py-2">
    <section className="space-y-4">
      <h2 className="font-bold text-gold">معاينة ما يراه المشاهدون</h2>
      <div data-question-look={fx.questionLook} data-real={fx.realism} data-motion={fx.realMotion} data-shape={fx.shape} data-glow={fx.glow} data-border={fx.border} data-shadow={fx.textShadow ? "on" : "off"} className="stage-root relative flex min-h-80 flex-col items-center gap-2 rounded-lg p-6" style={{ ...stageBackground(s.stage_background, s.background_image_url, s.background_dim), ["--panel-alpha" as string]: `${s.panel_opacity}%` }}>
        <ActivityWidget embedded extras={fx} counts={{ share: 4, follow: 9, comment: 37 }} events={[{ kind: "follow", uniqueId: "a", nickname: "سارة", avatar: "" }, { kind: "share", uniqueId: "b", nickname: "أحمد", avatar: "" }, { kind: "comment", uniqueId: "c", nickname: "ليلى", avatar: "", text: "طرابلس" }]} />
        {s.show_title && <div data-text="title" className="glass-pill px-5 py-1.5 text-xs font-bold text-muted-foreground">{s.title}</div>}
        {fx.categoryBadge && <div data-text="category" className={`rounded-2xl bg-primary px-8 py-2 font-black text-primary-foreground ${fx.titleSize === "lg" ? "text-3xl" : fx.titleSize === "sm" ? "text-base" : "text-xl"}`}>{categoryIcon("general")} ثقافة عامة</div>}
        {fx.questionNumber && <div className="glass-pill px-4 py-1 text-sm font-bold">السؤال 3 من 10</div>}
        <div className="stage-panel flex items-center gap-4 p-2">{s.show_counters && <span className="text-sm font-bold">12 أجاب</span>}<StageTimer extras={fx} label={demo} pct={demo / 25} warn={warn} />{s.show_counters && <span className="text-sm font-bold">40 مشارك</span>}</div>
        <QuestionSurface look={fx.questionLook} className="stage-panel w-full p-5 text-center"><h3 className={`quiz-text font-bold ${typographyClass(s.question_font, s.question_font_size)}`}>ما هي عاصمة ليبيا؟</h3><QuestionChatStrip /></QuestionSurface>
        <AnswersBanner phase="running" hasQuestion original={s.bottom_text} visible={s.show_bottom_text} settings={fx.answersBanner} />
      </div>
      <div className="rounded-lg p-4" style={stageBackground(s.stage_background, s.background_image_url, s.background_dim)}><p className="text-center font-bold text-gold">معاينة القرعة</p><div className="flex justify-center">{drawFinished ? <DrawWinner winners={DEMO_DRAW.filter((_, i) => i === drawIdx % DEMO_DRAW.length)} players={DEMO_DRAW} level={() => 2} points={() => 2} /> : <DrawScreen style={fx.drawStyle} players={DEMO_DRAW} current={DEMO_DRAW[drawIdx % DEMO_DRAW.length] ?? null} answers={["طرابلس"]} />}</div></div>
      <Field label={`ثواني المعاينة: ${previewSecs}`}><Slider min={1} max={30} step={1} value={[previewSecs]} onValueChange={([v = 4]) => setPreviewSecs(v)} /></Field>
      <Field label={`ثواني القرعة على البث: ${fx.drawSeconds}`}><Slider min={1} max={30} step={1} value={[fx.drawSeconds]} onValueChange={([v = 4]) => setFx({ drawSeconds: v })} /></Field>
      <Button variant="outline" disabled={drawing} onClick={() => { setDrawFinished(false); setDrawIdx(0); setDrawing(true); }}><Play />تشغيل معاينة القرعة</Button>
      <Field label={`ثواني المؤقت للمعاينة: ${demo}`}><Slider min={0} max={25} step={1} value={[demo]} onValueChange={([v = 0]) => setDemo(v)} /></Field>
    </section>
    <section className="space-y-4 border-t pt-6">
      <h2 className="font-bold text-gold">النصوص الظاهرة</h2>
      <Field label="عنوان المسابقة"><Input maxLength={80} value={s.title} onChange={(e) => set({ title: e.target.value })} /></Field>
      <Field label="النص أسفل الشاشة"><Input maxLength={160} value={s.bottom_text} onChange={(e) => set({ bottom_text: e.target.value })} /></Field>
      <Field label={`وضوح البطاقات: ${s.panel_opacity}%`}><Slider min={20} max={100} step={5} value={[s.panel_opacity]} onValueChange={([v = 70]) => set({ panel_opacity: v })} /></Field>
    </section>
    <section className="space-y-4 border-t pt-6">
      <h2 className="font-bold text-gold">المنضمون لمجتمع المعجبين</h2>
      <FanClubStudio fx={fx} onChange={(p) => setFx(p)} />
    </section>
    <section className="space-y-4 border-t pt-6">
      <h2 className="font-bold text-gold">إشعارات المشاركة والمتابعة</h2>
      <ActivityStudio fx={fx} onChange={(p) => setFx(p as Record<string, string | number>)} />
      <Field label={`شفافية خلفية الإشعارات: ${100 - fx.activityOpacity}%`}><Slider aria-label="شفافية خلفية الإشعارات" min={0} max={100} step={5} value={[100 - fx.activityOpacity]} onValueChange={([v = 65]) => setFx({ activityOpacity: 100 - v })} /></Field>
      <Button variant="secondary" onClick={() => setFx({ activityPosition: "top-center", activityStyle: "feed", activityShape: "rectangle", activityKinds: "social", activityOpacity: 35, activityCount: "3" })}><RotateCcw />اعتماد الشكل المقترح</Button>
    </section>
    <section className="space-y-4 border-t pt-6">
      <h2 className="font-bold text-gold">المؤقت وشكل الشاشة</h2>
      <div className="grid gap-4 sm:grid-cols-2">{opts.map(([k, def]) => <Field key={k} label={def.label}><Select value={fx[k]} onValueChange={(v) => setFx({ [k]: v })}><SelectTrigger aria-label={def.label}><SelectValue /></SelectTrigger><SelectContent>{def.options.map(([v, l]) => <SelectItem key={v} value={v}>{l}</SelectItem>)}</SelectContent></Select></Field>)}</div>
    </section>
    <section className="space-y-3 border-t pt-6">
      <h2 className="font-bold text-gold">عناصر تظهر أو تختفي</h2>
      {toggles.map(([k, l]) => <div key={k} className="flex items-center gap-3"><Switch id={`st-${k}`} checked={s[k]} onCheckedChange={(v) => set({ [k]: v })} /><Label htmlFor={`st-${k}`}>{l}</Label></div>)}
      {STAGE_TOGGLES.map(([k, l]) => <div key={k} className="flex items-center gap-3"><Switch id={`st-fx-${k}`} checked={fx[k]} onCheckedChange={(v) => setFx({ [k]: v })} /><Label htmlFor={`st-fx-${k}`}>{l}</Label></div>)}
    </section>
    <Button onClick={save}><Save />حفظ إعدادات شاشة البث</Button>
  </div>;
}

function SettingsPanel() {
  const { data } = useSettings();
  const qc = useQueryClient();
  const [s, setS] = useState<Settings | null>(null);
  useEffect(() => { if (data) setS(data); }, [data]);
  if (!s) return null;
  async function save() {
    if (!s || s.default_seconds < 3 || s.winners_per_question < 1 || s.points_per_level < 1 || s.winner_points < 0 || s.stats_seconds < 1 || s.max_level < 2) { toast.error("راجع قيم الوقت والنقاط والمستويات"); return; }
    const { error } = await supabase.from("settings").update({ ...s, updated_at: new Date().toISOString() }).eq("id", 1);
    if (error) { toast.error(error.message); return; }
    toast.success("تم الحفظ");
    qc.invalidateQueries({ queryKey: ["settings"] });
  }
  return (
    <div className="space-y-5 py-2">
      <Field label="عنوان المسابقة"><Input value={s.title} onChange={(e) => setS({ ...s, title: e.target.value })} /></Field>
      <Field label="اسم حسابك في TikTok (لاستقبال التعليقات)"><Input dir="ltr" placeholder="@username" value={s.tiktok_username} onChange={(e) => setS({ ...s, tiktok_username: e.target.value })} /></Field>
      <Field label="النص أسفل السؤال"><Input value={s.bottom_text} onChange={(e) => setS({ ...s, bottom_text: e.target.value })} /></Field>
      <div className="grid grid-cols-2 gap-4">
        <Field label="الوقت الافتراضي (ثانية)"><Input type="number" min={3} value={s.default_seconds} onChange={(e) => setS({ ...s, default_seconds: +e.target.value })} /></Field>
        <Field label="عدد الفائزين بالقرعة لكل سؤال"><Input type="number" min={1} value={s.winners_per_question} onChange={(e) => setS({ ...s, winners_per_question: +e.target.value })} /></Field>
      </div>
      <div className="grid grid-cols-2 gap-4">
        <Field label="نقاط كل مستوى"><Input type="number" min={1} value={s.points_per_level} onChange={(e) => setS({ ...s, points_per_level: +e.target.value })} /></Field>
        <Field label="النقاط الإضافية للفائز"><Input type="number" min={0} value={s.winner_points} onChange={(e) => setS({ ...s, winner_points: +e.target.value })} /></Field>
        <Field label="مستوى الاحتفال"><Input type="number" min={2} value={s.max_level} onChange={(e) => setS({ ...s, max_level: +e.target.value })} /></Field>
        <Field label="مدة عرض إحصاءات الإجابات (ثانية)"><Input type="number" min={1} value={s.stats_seconds} onChange={(e) => setS({ ...s, stats_seconds: +e.target.value })} /></Field>
      </div>
      <div className="flex items-center gap-3"><Switch id="settings-verse-audio" checked={s.verse_audio} onCheckedChange={(value) => setS({ ...s, verse_audio: value })} /><Label htmlFor="settings-verse-audio">تشغيل صوت التلاوة في أسئلة الآيات</Label></div>
      <Field label="قارئ التلاوات"><Select value={s.verse_reciter} onValueChange={(value) => setS({ ...s, verse_reciter: value })}><SelectTrigger aria-label="قارئ التلاوات"><SelectValue /></SelectT

... [truncated — file is 67468 bytes, showing first 51200]
