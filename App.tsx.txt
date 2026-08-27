import { useEffect, useMemo, useState } from 'react'
import type { CSSProperties, FormEvent, ReactNode } from 'react'
import {
  ArrowDown,
  ArrowUpRight,
  BookOpen,
  Check,
  ChevronDown,
  CircleUserRound,
  Download,
  ExternalLink,
  Code2,
  Globe2,
  Hammer,
  Heart,
  Instagram,
  Linkedin,
  Lock,
  Mail,
  Menu,
  Moon,
  MoveRight,
  Play,
  Rocket,
  Settings2,
  Sparkles,
  Star,
  Sun,
  Sword,
  Trophy,
  UserRound,
  Volume2,
  VolumeX,
  X,
  Zap,
} from 'lucide-react'
import { portfolio } from './data/portfolio'

type SectionKey = typeof portfolio.nav[number]['key']

type Project = typeof portfolio.projects[number]

const sectionIds: Record<SectionKey, string> = {
  home: 'home',
  about: 'about',
  skills: 'skills',
  projects: 'projects',
  experience: 'experience',
  achievements: 'achievements',
  resume: 'resume',
  contact: 'contact',
}

function App() {
  const [loading, setLoading] = useState(true)
  const [progress, setProgress] = useState(0)
  const [active, setActive] = useState<SectionKey>('home')
  const [soundOn, setSoundOn] = useState(false)
  const [themeNight, setThemeNight] = useState(false)
  const [mobileNav, setMobileNav] = useState(false)
  const [selectedProject, setSelectedProject] = useState<Project | null>(null)

  useEffect(() => {
    const started = performance.now()
    let frame = 0
    const tick = (time: number) => {
      const next = Math.min(100, Math.floor(((time - started) / 2300) * 100))
      setProgress(next)
      if (next < 100) frame = requestAnimationFrame(tick)
      else setTimeout(() => setLoading(false), 250)
    }
    frame = requestAnimationFrame(tick)
    return () => cancelAnimationFrame(frame)
  }, [])

  useEffect(() => {
    const onKey = (event: KeyboardEvent) => {
      if (event.key === 'Escape') {
        setSelectedProject(null)
        setMobileNav(false)
        return
      }
      const item = portfolio.nav.find((nav) => nav.keybind === event.key)
      if (item && !selectedProject) goTo(item.key)
    }
    window.addEventListener('keydown', onKey)
    return () => window.removeEventListener('keydown', onKey)
  }, [selectedProject])

  useEffect(() => {
    const observer = new IntersectionObserver(
      (entries) => {
        const visible = entries
          .filter((entry) => entry.isIntersecting)
          .sort((a, b) => b.intersectionRatio - a.intersectionRatio)[0]
        if (visible) setActive(visible.target.id as SectionKey)
      },
      { rootMargin: '-25% 0px -55% 0px', threshold: [0.1, 0.35, 0.6] },
    )
    Object.values(sectionIds).forEach((id) => {
      const el = document.getElementById(id)
      if (el) observer.observe(el)
    })
    return () => observer.disconnect()
  }, [loading])

  const hudStatus = useMemo(() => (soundOn ? 'SOUND ENABLED' : 'SOUND OFF'), [soundOn])

  function goTo(section: SectionKey) {
    setActive(section)
    setMobileNav(false)
    document.getElementById(sectionIds[section])?.scrollIntoView({ behavior: 'smooth', block: 'start' })
  }

  if (loading) return <LoadingScreen progress={progress} />

  return (
    <div className={themeNight ? 'app-shell night' : 'app-shell'}>
      <WorldAtmosphere night={themeNight} />
      <Hud active={active} soundOn={soundOn} onSound={() => setSoundOn((value) => !value)} onTheme={() => setThemeNight((value) => !value)} />

      <header className="mobile-topbar">
        <button className="pixel-button compact" onClick={() => setMobileNav((value) => !value)} aria-label="Open navigation">
          <Menu size={20} /> MENU
        </button>
        <div className="brand-mini">ANDYCRAFT</div>
        <button className="hud-icon" onClick={() => setThemeNight((value) => !value)} aria-label="Toggle day/night">
          {themeNight ? <Sun size={17} /> : <Moon size={17} />}
        </button>
      </header>

      {mobileNav && (
        <div className="mobile-drawer">
          {portfolio.nav.map((item) => (
            <button key={item.key} className={active === item.key ? 'drawer-link active' : 'drawer-link'} onClick={() => goTo(item.key)}>
              <NavIcon name={item.icon} size={18} />
              <span>{item.label}</span>
              <kbd>{item.keybind}</kbd>
            </button>
          ))}
        </div>
      )}

      <main>
        <Hero onEnter={() => goTo('about')} />
        <About />
        <Skills />
        <Projects onInspect={setSelectedProject} />
        <Experience />
        <Achievements />
        <Resume />
        <Contact />
      </main>

      <Hotbar active={active} onSelect={goTo} />
      <footer className="world-footer">
        <span>© {new Date().getFullYear()} {portfolio.profile.name}</span>
        <span className="footer-dot">•</span>
        <span>{portfolio.profile.world}</span>
        <span className="footer-dot">•</span>
        <span>{hudStatus}</span>
      </footer>

      {selectedProject && <ProjectModal project={selectedProject} onClose={() => setSelectedProject(null)} />}
    </div>
  )
}

function LoadingScreen({ progress }: { progress: number }) {
  return (
    <div className="loading-screen">
      <div className="loading-sky"></div>
      <div className="loading-particles">
        {Array.from({ length: 24 }).map((_, i) => <span key={i} style={{ '--i': i } as CSSProperties} />)}
      </div>
      <div className="loading-panel">
        <div className="loading-cube">
          <div className="cube-top"></div><div className="cube-front"></div><div className="cube-side"></div>
        </div>
        <div className="eyebrow">CHUNK 00 / 00</div>
        <h1>WORLD GENERATING...</h1>
        <p>Assembling blocks, quests and stories.</p>
        <div className="loading-bar"><span style={{ width: `${progress}%` }} /></div>
        <div className="loading-stats"><span>{progress}%</span><span>BUILDING WORLD</span></div>
      </div>
      <div className="loading-footer">ORIGINAL PIXEL-ART PORTFOLIO EXPERIENCE</div>
    </div>
  )
}

function WorldAtmosphere({ night }: { night: boolean }) {
  return (
    <div className={night ? 'world-atmosphere night' : 'world-atmosphere'} aria-hidden="true">
      <div className="sun-or-moon" />
      <div className="cloud cloud-a" /><div className="cloud cloud-b" /><div className="cloud cloud-c" />
      <div className="mountain ridge-a" /><div className="mountain ridge-b" />
      {Array.from({ length: 18 }).map((_, i) => <i className="firefly" key={i} style={{ '--i': i } as CSSProperties} />)}
      <div className="ground-grid" />
    </div>
  )
}

function Hud({ active, soundOn, onSound, onTheme }: { active: SectionKey; soundOn: boolean; onSound: () => void; onTheme: () => void }) {
  return (
    <div className="hud-wrap">
      <div className="hud-corner hud-left">
        <div className="status-strip"><span className="status-dot" /> WORLD ONLINE</div>
        <div className="hud-name">{portfolio.profile.name}</div>
        <div className="hud-role">{portfolio.profile.title}</div>
      </div>
      <div className="hud-corner hud-right">
        <div className="hud-actions">
          <button className="hud-icon" onClick={onSound} aria-label={soundOn ? 'Turn sound off' : 'Turn sound on'}>
            {soundOn ? <Volume2 size={17} /> : <VolumeX size={17} />}
          </button>
          <button className="hud-icon" onClick={onTheme} aria-label="Toggle day and night">
            <Moon size={17} />
          </button>
        </div>
        <div className="hud-coords">{portfolio.profile.coordinates}</div>
        <div className="hud-zone">ACTIVE CHUNK · {active.toUpperCase()}</div>
      </div>
      <div className="hearts" aria-label="Health 10 out of 10">
        {Array.from({ length: 10 }).map((_, i) => <Heart key={i} size={17} fill="currentColor" />)}
      </div>
      <div className="hunger" aria-label="Hunger 10 out of 10">
        {Array.from({ length: 10 }).map((_, i) => <span key={i} className="food">◆</span>)}
      </div>
      <div className="hud-xp">
        <span>XP</span><div className="xp-track"><span /></div><strong>{portfolio.profile.level}</strong>
      </div>
    </div>
  )
}

function Hero({ onEnter }: { onEnter: () => void }) {
  return (
    <section id="home" className="section hero-section section-snap">
      <div className="hero-grid">
        <div className="hero-copy">
          <div className="eyebrow with-icon"><Sparkles size={14} /> SPAWN POINT</div>
          <div className="hero-title-row">
            <div>
              <div className="world-title">ANDYCRAFT</div>
              <h1>{portfolio.profile.name}<span className="cursor-block" /></h1>
              <p className="hero-subtitle">{portfolio.profile.title}</p>
            </div>
          </div>
          <p className="hero-intro">{portfolio.profile.shortIntro}</p>
          <div className="hero-actions">
            <button className="pixel-button primary" onClick={onEnter}><Play size={18} fill="currentColor" /> ENTER WORLD</button>
            <a className="ghost-button" href={portfolio.profile.github} target="_blank" rel="noreferrer"><Github size={17} /> VIEW GITHUB <ArrowUpRight size={15} /></a>
          </div>
          <div className="hero-meta">
            <span><span className="tiny-block green" /> WORLD 01</span>
            <span><span className="tiny-block gold" /> LEVEL {portfolio.profile.level}</span>
            <span><span className="tiny-block cyan" /> ONLINE</span>
          </div>
        </div>
        <div className="spawn-stage" aria-label="Pixel art avatar display">
          <div className="portal-rune">⟐</div>
          <div className="spawn-frame">
            <div className="avatar-shadow" />
            <div className="avatar">
              <div className="avatar-head"><span /><span /></div>
              <div className="avatar-body"><div className="shirt-logo">⌘</div></div>
              <div className="avatar-legs"><span /><span /></div>
            </div>
            <div className="spawn-nameplate">PLAYER · {portfolio.profile.name.toUpperCase()}</div>
          </div>
          <div className="block stack-a" /><div className="block stack-b" /><div className="block stack-c" />
          <div className="crosshair"><span /><span /></div>
          <div className="spawn-label">LOOK AROUND · SCROLL TO EXPLORE</div>
        </div>
      </div>
      <button className="scroll-cue" onClick={() => document.getElementById('about')?.scrollIntoView({ behavior: 'smooth' })} aria-label="Scroll to About"><ChevronDown size={18} /> SCROLL DOWN</button>
    </section>
  )
}

function About() {
  return (
    <section id="about" className="section section-snap">
      <SectionHeading index="02" title="PLAYER INVENTORY" subtitle="A quick look at the builder behind the world." icon={<CircleUserRound size={20} />} />
      <div className="inventory-layout">
        <div className="inventory-card profile-card block-panel">
          <div className="inventory-header"><span>CHARACTER</span><span className="muted">LVL {portfolio.profile.level}</span></div>
          <div className="character-preview"><div className="avatar avatar-large"><div className="avatar-head"><span /><span /></div><div className="avatar-body"><div className="shirt-logo">⌘</div></div><div className="avatar-legs"><span /><span /></div></div></div>
          <div className="player-ident"><strong>{portfolio.profile.name}</strong><span>{portfolio.profile.role}</span></div>
          <div className="mini-stat"><span>HEALTH</span><div className="mini-hearts">{Array.from({ length: 5 }).map((_, i) => <Heart key={i} size={13} fill="currentColor" />)}</div></div>
          <div className="mini-stat"><span>STATUS</span><b className="status-text">OPEN TO QUESTS</b></div>
        </div>
        <div className="inventory-card bio-card block-panel">
          <div className="inventory-header"><span>PLAYER NOTES</span><span className="muted">READ ME</span></div>
          <p className="body-copy">{portfolio.profile.bio}</p>
          <div className="divider" />
          <div className="fact-grid">
            <Fact label="EDUCATION" value={portfolio.profile.education} icon={<BookOpen size={16} />} />
            <Fact label="CURRENT ROLE" value={portfolio.profile.role} icon={<Sword size={16} />} />
            <Fact label="HOME BASE" value={portfolio.profile.world} icon={<Globe2 size={16} />} />
            <Fact label="GOAL" value={portfolio.profile.goals[0]} icon={<Rocket size={16} />} />
          </div>
          <div className="inventory-items">
            <InventoryItem icon="💎" title="Developer" detail="Builds useful things" />
            <InventoryItem icon="📚" title="Student" detail="Always leveling up" />
            <InventoryItem icon="⚙" title="Problem Solver" detail="Likes hard quests" />
            <InventoryItem icon="🧠" title="AI Explorer" detail="Experiments with models" />
          </div>
        </div>
      </div>
    </section>
  )
}

function Skills() {
  return (
    <section id="skills" className="section section-snap">
      <SectionHeading index="03" title="SKILL INVENTORY" subtitle="Equipped tools, constantly being upgraded." icon={<Hammer size={20} />} />
      <div className="skill-layout">
        <div className="skill-grid">
          {portfolio.skills.map((skill) => <SkillCard key={skill.name} skill={skill} />)}
        </div>
        <div className="enchant-panel block-panel">
          <div className="book-cover"><BookOpen size={24} /><span>ENCHANTMENTS</span></div>
          <div className="enchant-line"><span>Curiosity</span><b>III</b></div>
          <div className="enchant-line"><span>Problem Solving</span><b>IV</b></div>
          <div className="enchant-line"><span>Communication</span><b>III</b></div>
          <div className="enchant-line"><span>Learning Speed</span><b>IV</b></div>
          <div className="enchant-note">Skill bars are configurable in <code>src/data/portfolio.ts</code>.</div>
        </div>
      </div>
    </section>
  )
}

function Projects({ onInspect }: { onInspect: (project: Project) => void }) {
  return (
    <section id="projects" className="section section-snap">
      <SectionHeading index="04" title="PROJECTS WORLD" subtitle="Builds, experiments and structures I have explored." icon={<Building2Icon />} />
      <div className="project-world-grid">
        {portfolio.projects.map((project, i) => <ProjectCard key={project.name} project={project} order={i} onInspect={() => onInspect(project)} />)}
      </div>
    </section>
  )
}

function Experience() {
  return (
    <section id="experience" className="section section-snap">
      <SectionHeading index="05" title="QUEST LOG" subtitle="Milestones, missions and lessons learned along the way." icon={<ScrollIcon />} />
      <div className="quest-log">
        {portfolio.experience.map((item, i) => (
          <div className="quest-row" key={`${item.title}-${item.period}`}>
            <div className="quest-node"><span>{String(i + 1).padStart(2, '0')}</span></div>
            <div className="quest-card block-panel">
              <div className="quest-top"><span className="quest-tag">QUEST COMPLETED</span><span className="muted">{item.period}</span></div>
              <h3>{item.title}</h3>
              <p><strong>Objective:</strong> {item.objective}</p>
              <div className="quest-rewards"><span><Zap size={14} /> {item.reward}</span><span><Trophy size={14} /> {item.achievement}</span></div>
            </div>
          </div>
        ))}
      </div>
    </section>
  )
}

function Achievements() {
  return (
    <section id="achievements" className="section section-snap">
      <SectionHeading index="06" title="ADVANCEMENTS" subtitle="Small wins that unlocked bigger quests." icon={<Trophy size={20} />} />
      <div className="achievement-grid">
        {portfolio.achievements.map((item) => (
          <div className={item.unlocked ? 'achievement-card unlocked block-panel' : 'achievement-card locked block-panel'} key={item.title}>
            <div className="achievement-icon">{item.unlocked ? <AchievementIcon name={item.icon} /> : <Lock size={26} />}</div>
            <div><span className="achievement-state">{item.unlocked ? 'UNLOCKED' : 'LOCKED'}</span><h3>{item.title}</h3><p>{item.description}</p></div>
            {item.unlocked && <Sparkles className="achievement-spark" size={16} />}
          </div>
        ))}
      </div>
    </section>
  )
}

function Resume() {
  return (
    <section id="resume" className="section section-snap">
      <SectionHeading index="07" title="ENCHANTED BOOK" subtitle="A clean, printable resume view—ready to replace with your real PDF." icon={<BookOpen size={20} />} />
      <div className="resume-wrap">
        <div className="resume-book">
          <div className="book-spine" />
          <div className="resume-page">
            <div className="resume-ribbon">RESUME</div>
            <div className="resume-header"><div><span className="eyebrow">PLAYER PROFILE</span><h3>{portfolio.profile.name}</h3><p>{portfolio.profile.title}</p></div><div className="resume-level">LVL {portfolio.profile.level}</div></div>
            <div className="resume-columns"><div><h4>ABOUT</h4><p>{portfolio.profile.bio}</p><h4>FOCUS</h4><p>{portfolio.profile.interests.join(' · ')}</p></div><div><h4>QUESTS</h4><ul>{portfolio.experience.map((item) => <li key={item.title}>{item.title} <span>{item.period}</span></li>)}</ul><h4>CONTACT</h4><p>{portfolio.profile.email}</p></div></div>
          </div>
        </div>
        <div className="resume-actions"><a className="pixel-button primary" href={portfolio.profile.resume} download><Download size={17} /> DOWNLOAD</a><a className="ghost-button" href={portfolio.profile.resume} target="_blank" rel="noreferrer"><ExternalLink size={16} /> OPEN RESUME</a></div>
      </div>
    </section>
  )
}

function Contact() {
  return (
    <section id="contact" className="section section-snap contact-section">
      <SectionHeading index="08" title="VILLAGE TRADING POST" subtitle="Looking for a collaboration? Pick a trade or send a message." icon={<Mail size={20} />} />
      <div className="contact-grid">
        <div className="villager-card block-panel"><div className="villager"><div className="villager-head"><span className="brow one"/><span className="brow two"/><span className="nose"/></div><div className="villager-body" /></div><div className="villager-copy"><span className="eyebrow">VILLAGER</span><div className="dialogue">“Hmm... Looking for a collaboration?”</div><p>Pick a trade below, or use the form to send a custom message.</p></div></div>
        <div className="trade-grid">
          <Trade href={`mailto:${portfolio.profile.email}`} icon={<Mail size={18} />} label="SEND EMAIL" value={portfolio.profile.email} />
          <Trade href={portfolio.profile.github} icon={<Github size={18} />} label="GITHUB" value="View projects" />
          <Trade href={portfolio.profile.linkedin} icon={<Linkedin size={18} />} label="LINKEDIN" value="Connect" />
          <Trade href={portfolio.profile.resume} icon={<Download size={18} />} label="RESUME" value="Download PDF" />
        </div>
      </div>
      <ContactForm />
    </section>
  )
}

function ContactForm() {
  const [submitted, setSubmitted] = useState(false)
  const [errors, setErrors] = useState<Record<string, string>>({})
  const [fields, setFields] = useState({ name: '', email: '', message: '' })

  const submit = (event: FormEvent) => {
    event.preventDefault()
    const next: Record<string, string> = {}
    if (!fields.name.trim()) next.name = 'Name is required.'
    if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(fields.email)) next.email = 'Enter a valid email.'
    if (fields.message.trim().length < 12) next.message = 'Message should be at least 12 characters.'
    setErrors(next)
    setSubmitted(Object.keys(next).length === 0)
  }

  return (
    <div className="contact-form block-panel">
      <div className="inventory-header"><span>MESSAGE CHEST</span><span className="muted">FORM-ONLY MODE</span></div>
      {submitted ? <div className="success-state"><Check size={28} /><h3>MESSAGE PACKED!</h3><p>Your form passed validation. Connect a backend or email provider to deliver it.</p><button className="ghost-button" onClick={() => setSubmitted(false)}>SEND ANOTHER</button></div> : (
        <form onSubmit={submit} noValidate>
          <div className="form-grid"><label>Name<input value={fields.name} onChange={(e) => setFields({ ...fields, name: e.target.value })} placeholder="Steve / Alex / Your Name" />{errors.name && <small>{errors.name}</small>}</label><label>Email<input value={fields.email} onChange={(e) => setFields({ ...fields, email: e.target.value })} placeholder="you@example.com" />{errors.email && <small>{errors.email}</small>}</label></div>
          <label>Message<textarea rows={5} value={fields.message} onChange={(e) => setFields({ ...fields, message: e.target.value })} placeholder="What are we building?" />{errors.message && <small>{errors.message}</small>}</label>
          <div className="form-bottom"><span className="muted">No backend configured — this demo validates locally.</span><button className="pixel-button primary" type="submit"><MoveRight size={17} /> SEND MESSAGE</button></div>
        </form>
      )}
    </div>
  )
}

function ProjectCard({ project, order, onInspect }: { project: Project; order: number; onInspect: () => void }) {
  const colors = ['grass', 'stone', 'portal']
  return (
    <article className={`project-card ${colors[order % colors.length]} block-panel`}>
      <div className="project-sky"><div className="project-sun" /><div className="project-cloud" /><div className="pixel-tree"><span /><span /></div><div className="project-building"><span /><span /><span /><span /></div><div className="project-ground" /></div>
      <div className="project-content"><div className="project-top"><span className="quest-tag">{project.type}</span><span className="project-order">#{String(order + 1).padStart(2, '0')}</span></div><h3>{project.name}</h3><p>{project.description}</p><div className="tech-pills">{project.tech.map((tech) => <span key={tech}>{tech}</span>)}</div><div className="project-actions"><button className="pixel-button small" onClick={onInspect}><Settings2 size={15} /> INSPECT</button><a className="icon-link" href={project.github} target="_blank" rel="noreferrer" aria-label={`${project.name} GitHub`}><Github size={17} /></a><a className="icon-link" href={project.demo} target="_blank" rel="noreferrer" aria-label={`${project.name} live demo`}><Globe2 size={17} /></a></div></div>
    </article>
  )
}

function ProjectModal({ project, onClose }: { project: Project; onClose: () => void }) {
  return (
    <div className="modal-backdrop" role="dialog" aria-modal="true" aria-label={`${project.name} project details`}>
      <div className="modal chest-modal">
        <button className="modal-close" onClick={onClose} aria-label="Close project"><X size={18} /></button>
        <div className="chest-header"><span>CHEST CONTENTS</span><b>{project.name}</b></div>
        <div className="chest-body"><div className="project-modal-hero"><div className="portal-mini" /><div className="project-building large" /></div><div className="modal-info"><span className="quest-tag">INSPECTED BUILD</span><p>{project.description}</p><InfoBlock title="FEATURES" items={project.features} /><InfoBlock title="TECH STACK" items={project.tech} /><InfoBlock title="CHALLENGE" items={[project.challenge]} /><InfoBlock title="RESULT" items={[project.results]} /><div className="modal-links"><a className="pixel-button small" href={project.github} target="_blank" rel="noreferrer"><Github size={15} /> GITHUB</a><a className="ghost-button" href={project.demo} target="_blank" rel="noreferrer"><ExternalLink size={15} /> LIVE DEMO</a></div></div></div>
      </div>
    </div>
  )
}

function SectionHeading({ index, title, subtitle, icon }: { index: string; title: string; subtitle: string; icon: ReactNode }) {
  return <div className="section-heading"><div className="section-index">{index}</div><div className="section-icon">{icon}</div><div><h2>{title}</h2><p>{subtitle}</p></div></div>
}

function Fact({ label, value, icon }: { label: string; value: string; icon: ReactNode }) {
  return <div className="fact"><span>{icon}</span><div><small>{label}</small><b>{value}</b></div></div>
}

function InventoryItem({ icon, title, detail }: { icon: string; title: string; detail: string }) {
  return <div className="item-slot"><span className="item-icon">{icon}</span><div><b>{title}</b><small>{detail}</small></div></div>
}

function SkillCard({ skill }: { skill: (typeof portfolio.skills)[number] }) {
  return <div className="skill-card block-panel"><div className={`skill-icon ${skill.tone}`}>{skill.icon === 'diamond' ? '◆' : skill.icon === 'redstone' ? '●' : skill.icon === 'pickaxe' ? '⛏' : skill.icon === 'command' ? '⌘' : skill.icon === 'book' ? '▣' : skill.icon === 'emerald' ? '◇' : skill.icon === 'slime' ? '●' : '✦'}</div><div className="skill-main"><div className="skill-top"><b>{skill.name}</b><span>{skill.level}%</span></div><div className="skill-track"><span style={{ width: `${skill.level}%` }} /></div><small>CONFIGURABLE LEVEL</small></div></div>
}

function Trade({ href, icon, label, value }: { href: string; icon: ReactNode; label: string; value: string }) {
  return <a className="trade-card" href={href} target={href.startsWith('http') ? '_blank' : undefined} rel={href.startsWith('http') ? 'noreferrer' : undefined}><div className="trade-icon">{icon}</div><div><b>{label}</b><span>{value}</span></div><ArrowUpRight size={15} /></a>
}

function InfoBlock({ title, items }: { title: string; items: readonly string[] }) {
  return <div className="info-block"><small>{title}</small><div>{items.map((item) => <span key={item}>{item}</span>)}</div></div>
}

function Hotbar({ active, onSelect }: { active: SectionKey; onSelect: (section: SectionKey) => void }) {
  return <nav className="hotbar" aria-label="Portfolio navigation">{portfolio.nav.map((item) => <button key={item.key} className={active === item.key ? 'hotbar-slot active' : 'hotbar-slot'} onClick={() => onSelect(item.key)} aria-label={`Go to ${item.label}`}><NavIcon name={item.icon} size={20} /><span className="hotbar-key">{item.keybind}</span><span className="tooltip">{item.label}</span></button>)}</nav>
}

function NavIcon({ name, size }: { name: string; size?: number }) {
  const props = { size, strokeWidth: 2 }
  const icons: Record<string, ReactNode> = { Home: <Sun {...props} />, UserRound: <UserRound {...props} />, Hammer: <Hammer {...props} />, Building2: <Building2Icon />, ScrollText: <ScrollIcon />, Trophy: <Trophy {...props} />, BookOpen: <BookOpen {...props} />, Mail: <Mail {...props} /> }
  return icons[name] ?? <Sparkles {...props} />
}

function Building2Icon() { return <Globe2 size={20} strokeWidth={2} /> }
function ScrollIcon() { return <BookOpen size={20} strokeWidth={2} /> }
function AchievementIcon({ name }: { name: string }) { if (name === 'sprout') return <Sparkles size={26} />; if (name === 'cog') return <Settings2 size={26} />; if (name === 'diamond') return <Star size={26} />; if (name === 'rocket') return <Rocket size={26} />; if (name === 'globe') return <Globe2 size={26} />; return <Trophy size={26} /> }

export default App
