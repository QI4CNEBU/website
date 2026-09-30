<script setup>
import { onBeforeUnmount, onMounted, ref, watch } from 'vue'

const heroImage =
  'https://trae-api-cn.mchost.guru/api/ide/v1/text_to_image?prompt=dark%20fantasy%20minecraft%20style%20game%20key%20art%3A%20floating%20glowing%20purple%20magic%20runes%20and%20spell%20circles%20in%20a%20misty%20night%20forest%2C%20deep%20violet%20and%20cyan%20lighting%2C%20cinematic%20wide%20shot%2C%20highly%20detailed%2C%20atmospheric&image_size=landscape_16_9'

const gameplayImage =
  'https://trae-api-cn.mchost.guru/api/ide/v1/text_to_image?prompt=minecraft%20style%20wizard%20tower%20interior%20with%20glowing%20enchanted%20bookshelves%2C%20floating%20purple%20magic%20particles%2C%20arcane%20circle%20on%20the%20stone%20floor%2C%20dark%20cozy%20fantasy%20lighting%2C%20game%20screenshot%20style%2C%20highly%20detailed&image_size=landscape_4_3'

const navLinks = [
  { label: '玩法特色', href: '#features' },
  { label: '魔法玩法', href: '#gameplay' },
  { label: '服务器信息', href: '#server' },
  { label: '加入我们', href: '#join' },
]

const stats = [
  { value: 'Java 版 1.20+', label: '支持版本' },
  { value: '魔法生存 · RPG', label: '玩法定位' },
  { value: '全天候开放', label: '在线时间' },
]

const features = [
  {
    title: '自定义法术体系',
    desc: '从咒语吟唱到符文刻印，自由组合法术效果与施法方式，打造只属于你的战斗风格。',
    icon: 'book',
  },
  {
    title: '元素派系进阶',
    desc: '火、水、风、雷等元素派系各具特色，随着修行深入，逐步解锁更强大的大魔法。',
    icon: 'flame',
  },
  {
    title: '遗迹探索与副本',
    desc: '主世界中散布着神秘魔法遗迹，与伙伴组队挑战高难副本，赢取稀有材料与专属装备。',
    icon: 'compass',
  },
  {
    title: '稳定社区与活动',
    desc: '长期稳定的服务器环境与友善活跃的社区，定期举办魔法主题活动与竞技联赛。',
    icon: 'shield',
  },
]

const gameplayPoints = [
  {
    title: '咒语吟唱与即时施法',
    desc: '学习咒语后可通过快捷栏与法杖即时施法，战斗节奏流畅，也支持编写专属法术组合。',
  },
  {
    title: '法杖、符文与炼金三条成长线',
    desc: '锻造法杖、铭刻符文、调配药剂，不同成长路线相互搭配，形成多样的养成体验。',
  },
  {
    title: '世界 BOSS 与限时魔法事件',
    desc: '定期刷新世界 BOSS 与限时魔法事件，全服玩家共同参与，争夺稀有奖励与称号。',
  },
]

const serverMeta = [
  { label: '支持版本', value: '即将推出' },
  { label: '玩法模式', value: '即将推出' },
  { label: '登录方式', value: '即将推出' },
]

/* ---------- 滚动进度 / 当前区块高亮 / 回到顶部 ---------- */
const sectionIds = ['top', 'features', 'gameplay', 'server', 'join']

const progress = ref(0)
const activeId = ref('top')
const showToTop = ref(false)

let sectionObserver = null

function handleScroll() {
  const doc = document.documentElement
  const scrollable = doc.scrollHeight - doc.clientHeight
  progress.value = scrollable > 0 ? Math.min(1, doc.scrollTop / scrollable) : 0
  showToTop.value = doc.scrollTop > 600
}

function scrollToTop() {
  window.scrollTo({ top: 0, behavior: 'smooth' })
}

/* ---------- 魔法弹窗 ---------- */
/* 目标链接（如交流群）尚未确定时，用弹窗给出反馈，避免按钮点了没反应 */
const modalOpen = ref(false)

function openModal() {
  modalOpen.value = true
}

function closeModal() {
  modalOpen.value = false
}

function handleKeydown(event) {
  if (event.key === 'Escape') closeModal()
}

/* 弹窗打开期间锁定页面滚动，并补偿滚动条宽度避免页面横向跳动 */
watch(modalOpen, (open) => {
  const gap = window.innerWidth - document.documentElement.clientWidth
  document.body.style.overflow = open ? 'hidden' : ''
  document.body.style.paddingRight = open && gap > 0 ? `${gap}px` : ''
})

/* ---------- 站内锚点平滑滚动 ---------- */
function handleAnchorClick(event) {
  const anchor = event.target.closest?.('a[href^="#"]')
  if (!anchor) return

  const id = anchor.getAttribute('href').slice(1)
  const target = id ? document.getElementById(id) : null
  if (!target) return

  event.preventDefault()

  // 尊重系统「减少动态效果」偏好
  const reduceMotion = window.matchMedia('(prefers-reduced-motion: reduce)').matches
  target.scrollIntoView({ behavior: reduceMotion ? 'auto' : 'smooth', block: 'start' })
  history.replaceState(null, '', `#${id}`)
}

/* 星尘的确定性伪随机分布（避免每次渲染抖动） */
function starStyle(index) {
  const seed = (index * 9301 + 49297) % 233280
  const a = seed / 233280
  const b = ((index * 4517 + 1777) % 9973) / 9973
  return {
    left: `${(a * 100).toFixed(2)}%`,
    top: `${(b * 100).toFixed(2)}%`,
    '--size': `${(1 + b * 2.2).toFixed(2)}px`,
    '--delay': `${(b * 7).toFixed(2)}s`,
    '--duration': `${(3.4 + a * 4).toFixed(2)}s`,
  }
}

onMounted(() => {
  handleScroll()
  window.addEventListener('scroll', handleScroll, { passive: true })
  window.addEventListener('keydown', handleKeydown)

  sectionObserver = new IntersectionObserver(
    (entries) => {
      entries.forEach((entry) => {
        if (entry.isIntersecting) activeId.value = entry.target.id
      })
    },
    { rootMargin: '-45% 0px -50% 0px', threshold: 0 },
  )

  sectionIds.forEach((id) => {
    const el = document.getElementById(id)
    if (el) sectionObserver.observe(el)
  })
})

onBeforeUnmount(() => {
  window.removeEventListener('scroll', handleScroll)
  window.removeEventListener('keydown', handleKeydown)
  document.body.style.overflow = ''
  document.body.style.paddingRight = ''
  sectionObserver?.disconnect()
})
</script>

<template>
  <div class="home" @click="handleAnchorClick">
    <header class="nav">
      <div class="nav-progress" :style="{ transform: `scaleX(${progress})` }"></div>
      <div class="container nav-inner">
        <a class="brand" href="#top">
          <svg class="brand-mark" viewBox="0 0 24 24" aria-hidden="true">
            <path
              d="M12 2.5 14.4 9.6 21.5 12 14.4 14.4 12 21.5 9.6 14.4 2.5 12 9.6 9.6Z"
              fill="currentColor"
            />
          </svg>
          <span class="brand-name">香草猫娘</span>
          <small class="brand-tag">Minecraft 魔法服务器</small>
        </a>

        <nav class="nav-links" aria-label="站内导航">
          <a
            v-for="link in navLinks"
            :key="link.href"
            :href="link.href"
            :class="{ 'is-active': activeId === link.href.slice(1) }"
            >{{ link.label }}</a
          >
        </nav>

        <button class="btn btn-primary btn-sm" type="button" @click="openModal">
          立即游玩
        </button>
      </div>
    </header>

    <section id="top" class="hero">
      <div class="hero-bg" :style="{ backgroundImage: `url(${heroImage})` }"></div>
      <div class="hero-overlay"></div>

      <div class="hero-aurora" aria-hidden="true">
        <span class="aurora aurora-a"></span>
        <span class="aurora aurora-b"></span>
        <span class="aurora aurora-c"></span>
      </div>

      <div class="hero-stars" aria-hidden="true">
        <span v-for="n in 28" :key="n" class="star" :style="starStyle(n)"></span>
      </div>

      <div class="container hero-inner">
        <span class="eyebrow">Minecraft 魔法主题服务器</span>
        <h1>香草猫娘</h1>
        <p class="hero-lead">
          在方块世界里编织你的魔法传说。香草猫娘是以魔法玩法为核心的 Minecraft
          服务器：自定义法术体系、元素派系、遗迹探索与团队副本，等待你书写属于自己的篇章。
        </p>

        <div class="hero-actions">
          <button class="btn btn-primary" type="button" @click="openModal">加入我们</button>
          <a class="btn btn-ghost" href="#features">了解玩法</a>
        </div>

        <ul class="hero-stats">
          <li v-for="item in stats" :key="item.label">
            <strong>{{ item.value }}</strong>
            <span>{{ item.label }}</span>
          </li>
        </ul>
      </div>
    </section>

    <section id="features" class="section">
      <div class="container">
        <header class="section-head">
          <span class="eyebrow">玩法特色</span>
          <h2>以魔法为核心的方块世界</h2>
          <p class="section-desc">
            从第一次点燃法力水晶，到掌握禁咒与元素共鸣，香草猫娘为你准备了完整的魔法成长体验。
          </p>
        </header>

        <div class="feature-grid">
          <article v-for="feature in features" :key="feature.title" class="feature-card">
            <div class="feature-icon">
              <svg
                v-if="feature.icon === 'book'"
                viewBox="0 0 24 24"
                fill="none"
                stroke="currentColor"
                stroke-width="1.6"
                stroke-linecap="round"
                stroke-linejoin="round"
                aria-hidden="true"
              >
                <path d="M4 5.5A2.5 2.5 0 0 1 6.5 3H19v15H6.5A2.5 2.5 0 0 0 4 20.5Z" />
                <path d="M4 20.5A2.5 2.5 0 0 1 6.5 18H19v3H6.5" />
                <path d="M12 7 12.9 9.6 15.5 10.5 12.9 11.4 12 14 11.1 11.4 8.5 10.5 11.1 9.6Z" />
              </svg>
              <svg
                v-else-if="feature.icon === 'flame'"
                viewBox="0 0 24 24"
                fill="none"
                stroke="currentColor"
                stroke-width="1.6"
                stroke-linecap="round"
                stroke-linejoin="round"
                aria-hidden="true"
              >
                <path d="M12 3.5c2.8 3.3 4.8 5.7 4.8 8.6a4.8 4.8 0 0 1-9.6 0c0-2.9 2-5.3 4.8-8.6Z" />
                <path d="M12 20.5a2.9 2.9 0 0 0 2.9-2.9c0-1.5-1-2.7-2.9-4.3-1.9 1.6-2.9 2.8-2.9 4.3A2.9 2.9 0 0 0 12 20.5Z" />
              </svg>
              <svg
                v-else-if="feature.icon === 'compass'"
                viewBox="0 0 24 24"
                fill="none"
                stroke="currentColor"
                stroke-width="1.6"
                stroke-linecap="round"
                stroke-linejoin="round"
                aria-hidden="true"
              >
                <circle cx="12" cy="12" r="8.5" />
                <path d="M14.9 9.1 13 13l-3.9 1.9L11 11Z" />
              </svg>
              <svg
                v-else
                viewBox="0 0 24 24"
                fill="none"
                stroke="currentColor"
                stroke-width="1.6"
                stroke-linecap="round"
                stroke-linejoin="round"
                aria-hidden="true"
              >
                <path d="M12 3.5 19 6v6c0 4-3 6.7-7 8.5-4-1.8-7-4.5-7-8.5V6Z" />
                <path d="m9.2 12.2 2 2 3.6-3.9" />
              </svg>
            </div>
            <h3>{{ feature.title }}</h3>
            <p>{{ feature.desc }}</p>
          </article>
        </div>
      </div>
    </section>

    <section id="gameplay" class="section section-alt">
      <div class="container gameplay-grid">
        <div class="gameplay-media">
          <!-- 客户要求页面更简洁，暂时关闭魔法阵装饰。
               恢复方式：把下面这行 div 移出注释即可，样式（.magic-circle）仍完整保留。
          <div class="magic-circle" aria-hidden="true"></div>
          -->
          <figure class="gameplay-visual">
            <img :src="gameplayImage" alt="服务器内的魔法师塔：发光书架与地面上的法阵" />
          </figure>
        </div>

        <div class="gameplay-copy">
          <span class="eyebrow">魔法玩法</span>
          <h2>独特的魔法体系，等你来修行</h2>
          <p class="section-desc">
            香草猫娘围绕魔法重构了战斗与成长体验，让每一次施法、每一场探索都充满仪式感。
          </p>

          <ul class="gameplay-list">
            <li v-for="point in gameplayPoints" :key="point.title">
              <div>
                <strong>{{ point.title }}</strong>
                <span>{{ point.desc }}</span>
              </div>
            </li>
          </ul>
        </div>
      </div>
    </section>

    <section id="server" class="section">
      <div class="container">
        <header class="section-head">
          <span class="eyebrow">服务器信息</span>
          <h2>香草猫娘即将开放</h2>
          <p class="section-desc">服务器正在筹备中，开服时间与连接方式将在本站与社区同步公布。</p>
        </header>

        <div class="server-card">
          <div class="server-address">
            <span class="server-label">服务器地址</span>
            <button class="server-pending" type="button" @click="openModal">即将推出</button>
          </div>

          <ul class="server-meta">
            <li v-for="item in serverMeta" :key="item.label">
              <span>{{ item.label }}</span>
              <strong>{{ item.value }}</strong>
            </li>
          </ul>

          <p class="server-note">开服前本站会更新服务器地址与进入方式，敬请期待。</p>
        </div>
      </div>
    </section>

    <section id="join" class="section">
      <div class="container">
        <div class="cta-inner">
          <h2>准备好开始你的魔法之旅了吗？</h2>
          <p class="section-desc">
            无论是初入魔法之门的新人，还是追寻禁咒的资深法师，香草猫娘都为你留好了位置。
          </p>
          <div class="hero-actions">
            <button class="btn btn-primary" type="button" @click="openModal">立即加入</button>
            <a class="btn btn-ghost" href="#gameplay">了解魔法玩法</a>
          </div>
        </div>
      </div>
    </section>

    <footer class="footer">
      <div class="container">
        <div class="footer-inner">
          <div class="footer-brand">
            <svg class="brand-mark" viewBox="0 0 24 24" aria-hidden="true">
              <path
                d="M12 2.5 14.4 9.6 21.5 12 14.4 14.4 12 21.5 9.6 14.4 2.5 12 9.6 9.6Z"
                fill="currentColor"
              />
            </svg>
            <div>
              <strong>香草猫娘</strong>
              <p>以魔法为核心的 Minecraft 服务器</p>
            </div>
          </div>

          <nav class="footer-col" aria-label="页脚导航">
            <h3 class="footer-title">站内导航</h3>
            <ul class="footer-list">
              <li><a href="#top">首页</a></li>
              <li v-for="link in navLinks" :key="link.href">
                <a :href="link.href">{{ link.label }}</a>
              </li>
            </ul>
          </nav>

          <div class="footer-col">
            <h3 class="footer-title">开服状态</h3>
            <ul class="footer-list footer-list--meta">
              <li v-for="item in serverMeta" :key="item.label">
                <span>{{ item.label }}</span>
                <strong>{{ item.value }}</strong>
              </li>
            </ul>
          </div>
        </div>

        <div class="footer-bottom">
          <span>© 2026 香草猫娘 · 保留所有权利</span>
          <span>本站为玩家自建服务器，与 Mojang Studios 及 Microsoft 无隶属关系。</span>
        </div>
      </div>
    </footer>

    <Transition name="modal">
      <div
        v-if="modalOpen"
        class="modal"
        role="dialog"
        aria-modal="true"
        aria-labelledby="magic-modal-title"
        @click.self="closeModal"
      >
        <div class="modal-card">
          <div class="modal-ring" aria-hidden="true"></div>

          <span class="modal-icon" aria-hidden="true">
            <svg viewBox="0 0 24 24">
              <path
                d="M12 2.5 14.4 9.6 21.5 12 14.4 14.4 12 21.5 9.6 14.4 2.5 12 9.6 9.6Z"
                fill="currentColor"
              />
            </svg>
          </span>

          <h2 id="magic-modal-title" class="modal-title">魔法大门尚未开启</h2>
          <p class="modal-text">敬请期待！</p>

          <button class="btn btn-primary" type="button" @click="closeModal">我知道了</button>
        </div>
      </div>
    </Transition>

    <Transition name="to-top">
      <button
        v-show="showToTop"
        class="to-top"
        type="button"
        aria-label="回到顶部"
        @click="scrollToTop"
      >
        <svg
          viewBox="0 0 24 24"
          fill="none"
          stroke="currentColor"
          stroke-width="1.8"
          stroke-linecap="round"
          stroke-linejoin="round"
          aria-hidden="true"
        >
          <path d="M12 19V5.5" />
          <path d="m5.8 11.7 6.2-6.2 6.2 6.2" />
        </svg>
      </button>
    </Transition>
  </div>
</template>

<style scoped>
.home {
  position: relative;
  /* 保持透明，让 body 的魔法光晕透出 */
  background: transparent;
  overflow-x: clip;
}

.container {
  width: min(1180px, 100% - 48px);
  margin-inline: auto;
}

/* ---------- 导航 ---------- */
.nav {
  position: sticky;
  top: 0;
  z-index: 50;
  background: rgba(6, 6, 11, 0.72);
  backdrop-filter: blur(14px);
  /* 去掉底部实线，改由滚动进度条与文字高亮提示位置 */
}

.nav-inner {
  display: flex;
  align-items: center;
  gap: 32px;
  height: 68px;
}

/* 顶部滚动进度光条（已降调：1px、无发光，避免成为一条突兀的亮紫边） */
.nav-progress {
  position: absolute;
  inset: auto 0 0;
  height: 1px;
  transform: scaleX(0);
  transform-origin: left center;
  background: linear-gradient(
    90deg,
    rgba(124, 58, 237, 0.5),
    rgba(196, 132, 252, 0.75) 45%,
    rgba(34, 211, 238, 0.5)
  );
  pointer-events: none;
}

.brand {
  display: flex;
  align-items: center;
  gap: 10px;
}

.brand-mark {
  width: 24px;
  height: 24px;
  color: var(--primary);
  filter: drop-shadow(0 0 10px rgba(168, 85, 247, 0.7));
}

.brand-name {
  font-size: 17px;
  font-weight: 600;
  letter-spacing: 0.06em;
  color: #fff;
}

.brand-tag {
  padding-left: 12px;
  border-left: 1px solid var(--border-soft);
  font-size: 12px;
  font-weight: 400;
  color: var(--text-muted);
  letter-spacing: 0.04em;
}

.nav-links {
  display: flex;
  gap: 28px;
  margin-left: auto;
  font-size: 14px;
  color: var(--text-muted);
}

.nav-links a {
  position: relative;
  transition: color 0.2s ease;
}

.nav-links a::after {
  content: "";
  position: absolute;
  left: 0;
  right: 0;
  bottom: -7px;
  height: 2px;
  border-radius: 999px;
  background: linear-gradient(90deg, transparent, #c084fc, transparent);
  opacity: 0;
  transform: scaleX(0.35);
  transition:
    opacity 0.25s ease,
    transform 0.25s ease;
}

.nav-links a:hover {
  color: #fff;
}

.nav-links a.is-active {
  color: #fff;
}

.nav-links a:hover::after,
.nav-links a.is-active::after {
  opacity: 1;
  transform: scaleX(1);
}

/* ---------- 按钮 ---------- */
.btn {
  position: relative;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  height: 46px;
  padding: 0 26px;
  overflow: hidden;
  border: 1px solid transparent;
  border-radius: 999px;
  font-size: 15px;
  font-weight: 500;
  cursor: pointer;
  transition:
    transform 0.2s ease,
    filter 0.25s ease,
    box-shadow 0.25s ease,
    background-color 0.25s ease,
    border-color 0.25s ease,
    color 0.25s ease;
}

/* 悬停时掠过的流光，制造“魔法充能”感 */
.btn::after {
  content: "";
  position: absolute;
  inset: 0 auto 0 -60%;
  width: 45%;
  background: linear-gradient(100deg, transparent, rgba(255, 255, 255, 0.38), transparent);
  transform: skewX(-18deg);
  transition: left 0.65s ease;
  pointer-events: none;
}

.btn:hover::after {
  left: 118%;
}

/*
 * 白字对比度：原 #a855f7 → #7c3aed 的最亮端仅 3.96:1，未达 WCAG AA（4.5:1）。
 * 整体压深一档为 #9333ea → #6d28d9 后：
 *   最亮端 5.38:1、最深端 7.10:1，全按钮区间均 ≥ AA，较深一半达 AAA（7:1）。
 */
.btn-primary {
  background: linear-gradient(135deg, #9333ea, #6d28d9);
  color: #fff;
  box-shadow: 0 12px 32px -14px rgba(168, 85, 247, 0.9);
}

/* 悬停：整体提亮 + 明显的紫色外发光
   （提亮系数从 1.16 收到 1.10，否则悬停时最亮端会掉到 4.37:1、重新跌破 AA） */
.btn-primary:hover {
  transform: translateY(-2px);
  filter: brightness(1.1) saturate(1.05);
  box-shadow:
    0 18px 38px -12px rgba(168, 85, 247, 1),
    0 0 28px rgba(168, 85, 247, 0.7);
}

.btn-ghost {
  background: rgba(138, 43, 226, 0.12);
  border-color: var(--border);
  color: var(--text);
}

/* 悬停：底色加深、描边提亮、再叠一层柔光 */
.btn-ghost:hover {
  transform: translateY(-2px);
  color: #fff;
  background: rgba(138, 43, 226, 0.26);
  border-color: rgba(196, 150, 255, 0.75);
  box-shadow: 0 0 24px -6px rgba(168, 85, 247, 0.85);
}

.btn-sm {
  height: 38px;
  padding: 0 20px;
  font-size: 14px;
}

/* 点击回弹，避免“按了没反应”的错觉 */
.btn:active {
  transform: translateY(0) scale(0.97);
}

/* ---------- 英雄区 ---------- */
.hero {
  position: relative;
  padding: clamp(96px, 13vw, 168px) 0 clamp(80px, 10vw, 128px);
  text-align: center;
  overflow: hidden;
}

.hero-bg {
  position: absolute;
  inset: 0;
  background-size: cover;
  background-position: center;
  opacity: 0.55;
  /* 底部把背景图淡出：首屏底边的切线其实来自背景图被 overflow 硬裁，
     而不是 border 或实色背景，所以要在图上做遮罩 */
  -webkit-mask-image: linear-gradient(180deg, #000 55%, transparent 100%);
  mask-image: linear-gradient(180deg, #000 55%, transparent 100%);
}

.hero-overlay {
  position: absolute;
  inset: 0;
  background:
    radial-gradient(70% 60% at 50% 0%, rgba(168, 85, 247, 0.3), transparent 62%),
    linear-gradient(
      180deg,
      rgba(6, 6, 11, 0.55) 0%,
      rgba(6, 6, 11, 0.78) 55%,
      /* 85.4% = 首屏统计数据块的下沿，暗角在此之前保持全强度 */
      rgba(6, 6, 11, 0.8) 85.4%,
      /* 底边归零，与下方区块的 body 背景严丝合缝，消除横向色阶跳变 */
      rgba(6, 6, 11, 0) 100%
    );
}

.hero::after {
  content: "";
  position: absolute;
  inset: 0;
  background-image:
    linear-gradient(rgba(255, 255, 255, 0.04) 1px, transparent 1px),
    linear-gradient(90deg, rgba(255, 255, 255, 0.04) 1px, transparent 1px);
  background-size: 64px 64px;
  -webkit-mask-image: radial-gradient(60% 50% at 50% 30%, #000, transparent 78%);
  mask-image: radial-gradient(60% 50% at 50% 30%, #000, transparent 78%);
  pointer-events: none;
}

/* ---------- 极光与星尘 ---------- */
.hero-aurora {
  position: absolute;
  inset: -12% -6% auto;
  height: 118%;
  filter: blur(64px);
  opacity: 0.72;
  /*
   * 底部淡出：本容器比首屏高出 42px，模糊后的粉色光晕会在首屏底边被
   * overflow 硬切（实测 residual alpha ≈ 0.11）。首屏底边位于本容器
   * 94.9% 高度处，故在 76%→95% 之间把光晕收掉，出界前已归零。
   */
  -webkit-mask-image: linear-gradient(180deg, #000 76%, transparent 95%);
  mask-image: linear-gradient(180deg, #000 76%, transparent 95%);
  pointer-events: none;
}

.aurora {
  position: absolute;
  border-radius: 50%;
  mix-blend-mode: screen;
}

.aurora-a {
  top: 4%;
  left: 2%;
  width: 46vw;
  height: 46vw;
  background: radial-gradient(circle at 40% 40%, rgba(168, 85, 247, 0.72), transparent 68%);
  animation: aurora-drift-a 18s ease-in-out infinite;
}

.aurora-b {
  top: 10%;
  right: 4%;
  width: 38vw;
  height: 38vw;
  background: radial-gradient(circle at 60% 40%, rgba(34, 211, 238, 0.46), transparent 68%);
  animation: aurora-drift-b 23s ease-in-out infinite;
}

.aurora-c {
  bottom: -8%;
  left: 34%;
  width: 34vw;
  height: 34vw;
  background: radial-gradient(circle at 50% 50%, rgba(244, 114, 182, 0.34), transparent 70%);
  animation: aurora-drift-c 27s ease-in-out infinite;
}

@keyframes aurora-drift-a {
  0%,
  100% {
    transform: translate3d(0, 0, 0) scale(1);
  }
  50% {
    transform: translate3d(6%, 5%, 0) scale(1.14);
  }
}

@keyframes aurora-drift-b {
  0%,
  100% {
    transform: translate3d(0, 0, 0) scale(1.06);
  }
  50% {
    transform: translate3d(-7%, 6%, 0) scale(0.94);
  }
}

@keyframes aurora-drift-c {
  0%,
  100% {
    transform: translate3d(0, 0, 0) scale(1);
  }
  50% {
    transform: translate3d(4%, -6%, 0) scale(1.18);
  }
}

.hero-stars {
  position: absolute;
  inset: 0;
  overflow: hidden;
  pointer-events: none;
}

.star {
  position: absolute;
  width: var(--size, 2px);
  height: var(--size, 2px);
  border-radius: 50%;
  background: #f6efff;
  box-shadow: 0 0 8px 1px rgba(202, 168, 255, 0.9);
  opacity: 0;
  animation: star-twinkle var(--duration, 5s) ease-in-out infinite;
  animation-delay: var(--delay, 0s);
}

@keyframes star-twinkle {
  0%,
  100% {
    opacity: 0;
    transform: scale(0.6);
  }
  45% {
    opacity: 0.95;
    transform: scale(1);
  }
}

.hero-inner {
  position: relative;
  z-index: 1;
}

.eyebrow {
  display: inline-flex;
  align-items: center;
  padding: 7px 16px;
  border: 1px solid var(--border);
  border-radius: 999px;
  background: var(--primary-soft);
  color: #dcc9ff;
  font-size: 13px;
  letter-spacing: 0.1em;
}

.hero h1 {
  margin: 26px 0 22px;
  font-size: clamp(44px, 7vw, 78px);
  font-weight: 700;
  letter-spacing: 0.08em;
  background: linear-gradient(180deg, #ffffff, #c9b4f5);
  -webkit-background-clip: text;
  background-clip: text;
  color: transparent;
  /* 淡紫色双层发光，让标题“悬浮”在光晕中 */
  text-shadow:
    0 0 22px rgba(196, 150, 255, 0.55),
    0 0 55px rgba(140, 80, 255, 0.35);
  filter: drop-shadow(0 0 32px rgba(168, 85, 247, 0.45));
  animation: title-breathe 6s ease-in-out infinite;
}

@keyframes title-breathe {
  0%,
  100% {
    filter: drop-shadow(0 0 26px rgba(168, 85, 247, 0.38));
  }
  50% {
    filter: drop-shadow(0 0 48px rgba(168, 85, 247, 0.72));
  }
}

.hero-lead {
  max-width: 660px;
  margin-inline: auto;
  color: var(--text-muted);
  font-size: clamp(15px, 1.3vw, 17px);
  line-height: 1.9;
}

.hero-actions {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  gap: 14px;
  margin-top: 36px;
}

.hero-stats {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  gap: clamp(24px, 4vw, 60px);
  margin: clamp(40px, 5vw, 56px) 0 0;
  padding: 0;
  list-style: none;
}

.hero-stats li {
  display: flex;
  flex-direction: column;
  gap: 6px;
}

.hero-stats strong {
  color: #fff;
  font-size: clamp(17px, 1.6vw, 20px);
  font-weight: 600;
}

.hero-stats span {
  color: var(--text-muted);
  font-size: 13px;
  letter-spacing: 0.08em;
}

/* ---------- 通用区块 ---------- */
.section {
  padding: clamp(96px, 12vw, 168px) 0;
}

.section-alt {
  /* 去掉横贯整屏的两条切割线，改由上下渐隐的光带区分区块。
     紫光强度减半（0.06 → 0.03）：卡片改为「更暗的磨砂面板」后，
     区块这层紫光只需提供方向感，过浓会和卡片叠色显脏。 */
  background: linear-gradient(
    180deg,
    transparent,
    rgba(168, 85, 247, 0.03) 30%,
    rgba(168, 85, 247, 0.03) 70%,
    transparent
  );
}

.section-head {
  max-width: 720px;
  margin: 0 auto clamp(56px, 6.5vw, 88px);
  text-align: center;
}

.section-head h2,
.gameplay-copy h2,
.cta-inner h2 {
  margin: 22px 0 16px;
  color: #fff;
  font-size: clamp(26px, 3.2vw, 38px);
  font-weight: 650;
  letter-spacing: 0.02em;
}

/*
 * 「独特的魔法体系，等你来修行」共 13 个全角字符（每 em 文字宽 13.26），
 * 而双栏下文字列只有 496～504px，38px 字号需要 503.9px，正好卡在临界点上被折成两行。
 * 这里单独收敛这枚标题的字号（其余区块标题不受影响），
 * 桌面端单行余量 +32～38px；下限 24px 保证窄屏仍能优雅折行而非溢出。
 */
.gameplay-copy h2 {
  font-size: clamp(24px, 2.8vw, 35px);
}

.section-desc {
  color: var(--text-muted);
  font-size: 15.5px;
  line-height: 1.9;
}

/* ---------- 玩法特色 ---------- */
.feature-grid {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 20px;
}

.feature-card {
  padding: 32px 28px;
  /* 描边降到几乎不可见，只留一丝轮廓 */
  border: 1px solid rgba(255, 255, 255, 0.05);
  border-radius: 18px;
  /* 比区块「更深」：靠暗度分层，而不是把紫色叠在区块的紫色之上（会串色显脏） */
  background: rgba(5, 5, 12, 0.55);
  backdrop-filter: blur(8px);
  -webkit-backdrop-filter: blur(8px);
  /* 顶部内高光 + 大范围低透明度外发光 */
  box-shadow:
    inset 0 1px 0 rgba(255, 255, 255, 0.05),
    0 0 48px 4px rgba(138, 43, 226, 0.06);
  transition:
    transform 0.3s ease,
    background-color 0.3s ease,
    border-color 0.3s ease,
    box-shadow 0.3s ease;
}

/* 边界「按需出现」：静止态柔和，悬停时才点亮轮廓 */
.feature-card:hover {
  transform: translateY(-6px);
  border-color: rgba(196, 150, 255, 0.45);
  /* 悬停保持同一套「深色面板」语言，只微微提亮；若改成紫色会破坏深浅分层 */
  background-color: rgba(24, 20, 42, 0.6);
  box-shadow:
    inset 0 1px 0 rgba(255, 255, 255, 0.1),
    0 26px 54px -32px rgba(168, 85, 247, 0.9);
}

.feature-icon {
  display: grid;
  place-items: center;
  width: 44px;
  height: 44px;
  border: 1px solid var(--border);
  border-radius: 12px;
  background: var(--primary-soft);
  color: #d8b4fe;
  transition:
    transform 0.3s ease,
    box-shadow 0.3s ease,
    border-color 0.3s ease;
}

.feature-card:hover .feature-icon {
  transform: translateY(-2px) scale(1.06);
  border-color: var(--primary);
  box-shadow: 0 0 24px -6px rgba(168, 85, 247, 0.95);
}

.feature-icon svg {
  width: 22px;
  height: 22px;
}

.feature-card h3 {
  margin: 20px 0 10px;
  color: #fff;
  font-size: 17px;
  font-weight: 600;
}

.feature-card p {
  color: var(--text-muted);
  font-size: 14.5px;
  line-height: 1.8;
}

/* ---------- 魔法玩法 ---------- */
.gameplay-grid {
  display: grid;
  /* 加宽图片列，配合负边距让图片真正“长大” */
  grid-template-columns: 1.2fr 1fr;
  gap: clamp(40px, 6vw, 88px);
  align-items: center;
}

.gameplay-media {
  position: relative;
  z-index: 0;
  /*
   * 向左突破容器边界：
   * 可突破量 = 容器左侧留白 - 16px，上限 96px。
   * 用容器留白反推而非固定值，保证任何窗口宽度下都既有效果、
   * 又不会越过视口左边缘被裁切（窄屏至少保留 16px 边距）。
   */
  margin-left: calc(
    -1 * clamp(0px, calc((100vw - min(1180px, 100vw - 48px)) / 2 - 16px), 96px)
  );
}

/* 图片背后的淡魔法阵：旋转符文环 + 柔光底 */
.magic-circle {
  /* 客户要求关闭该装饰元素（模板中已注释），这里再兜底隐藏，
     同时让 52s 的旋转动画彻底停止，避免无谓的合成开销。
     恢复时删掉这一行即可，其余声明保持原样备用。 */
  display: none;
  position: absolute;
  top: 50%;
  left: 50%;
  width: 126%;
  aspect-ratio: 1;
  border-radius: 50%;
  background:
    repeating-conic-gradient(
      from 0deg,
      rgba(196, 150, 255, 0.55) 0deg 0.5deg,
      transparent 0.5deg 9deg
    ),
    radial-gradient(
      circle,
      rgba(168, 85, 247, 0.22),
      rgba(124, 58, 237, 0.08) 52%,
      transparent 70%
    );
  opacity: 0.42;
  -webkit-mask-image: radial-gradient(circle, transparent 30%, #000 42%, #000 72%, transparent 84%);
  mask-image: radial-gradient(circle, transparent 30%, #000 42%, #000 72%, transparent 84%);
  transform: translate(-50%, -50%) rotate(0deg);
  animation: magic-circle-spin 52s linear infinite;
  pointer-events: none;
}

@keyframes magic-circle-spin {
  to {
    transform: translate(-50%, -50%) rotate(360deg);
  }
}

.gameplay-visual {
  position: relative;
  z-index: 1;
  overflow: hidden;
  border: 1px solid var(--border);
  border-radius: 22px;
  background: var(--bg-1);
  box-shadow: 0 44px 84px -44px rgba(0, 0, 0, 0.95);
}

.gameplay-visual img {
  width: 100%;
  aspect-ratio: 4 / 3;
  object-fit: cover;
}

.gameplay-visual::after {
  content: "";
  position: absolute;
  inset: 0;
  background: linear-gradient(180deg, transparent 45%, rgba(6, 6, 11, 0.55));
  pointer-events: none;
}

.gameplay-list {
  display: grid;
  gap: 18px;
  margin: 30px 0 0;
  padding: 0;
  list-style: none;
}

.gameplay-list li {
  display: grid;
  grid-template-columns: 12px 1fr;
  gap: 14px;
}

.gameplay-list li::before {
  content: "";
  width: 10px;
  height: 10px;
  margin-top: 8px;
  background: linear-gradient(135deg, #c084fc, #7c3aed);
  clip-path: polygon(
    50% 0,
    62% 38%,
    100% 50%,
    62% 62%,
    50% 100%,
    38% 62%,
    0 50%,
    38% 38%
  );
  box-shadow: 0 0 12px rgba(168, 85, 247, 0.85);
}

.gameplay-list strong {
  display: block;
  margin-bottom: 5px;
  color: #fff;
  font-size: 15.5px;
  font-weight: 600;
}

.gameplay-list span {
  color: var(--text-muted);
  font-size: 14.5px;
  line-height: 1.8;
}

/* ---------- 服务器信息 ---------- */
.server-card {
  display: grid;
  gap: 28px;
  padding: clamp(32px, 4.5vw, 52px);
  /* 按先前的明确要求保持无描边，靠暗度与柔光界定范围 */
  border: none;
  border-radius: 24px;
  /* 与另两张卡片统一为「更暗的磨砂面板」 */
  background: rgba(5, 5, 12, 0.55);
  backdrop-filter: blur(8px);
  -webkit-backdrop-filter: blur(8px);
  box-shadow: 0 0 80px 20px rgba(138, 43, 226, 0.05);
}

.server-address {
  display: flex;
  align-items: center;
  flex-wrap: wrap;
  gap: 14px 20px;
  /* 内部横线已去掉，同时收掉原来为它留的下边距，避免留下空洞 */
}

.server-label {
  color: var(--text-muted);
  font-size: 14px;
  letter-spacing: 0.06em;
}

.server-pending {
  padding: 6px 20px;
  border: 1px dashed var(--border);
  border-radius: 999px;
  background: var(--primary-soft);
  color: #dcc9ff;
  font-size: clamp(16px, 2vw, 21px);
  letter-spacing: 0.16em;
  cursor: pointer;
  transition:
    border-color 0.25s ease,
    border-style 0.25s ease,
    background-color 0.25s ease,
    box-shadow 0.25s ease;
}

.server-pending:hover {
  border-style: solid;
  border-color: var(--primary);
  background: rgba(138, 43, 226, 0.26);
  box-shadow: 0 0 26px -6px rgba(168, 85, 247, 0.9);
}

.server-meta {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 22px;
  margin: 0;
  padding: 0;
  list-style: none;
}

.server-meta li {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.server-meta span {
  color: var(--text-muted);
  font-size: 13px;
  letter-spacing: 0.06em;
}

.server-meta strong {
  color: #fff;
  font-size: 16px;
  font-weight: 600;
}

.server-note {
  color: var(--text-muted);
  font-size: 14px;
}

/* ---------- 行动号召 ---------- */
.cta-inner {
  padding: clamp(48px, 7vw, 80px) clamp(24px, 4vw, 60px);
  border: 1px solid rgba(255, 255, 255, 0.05);
  border-radius: 28px;
  text-align: center;
  /* 与另两张卡片统一为「更暗的磨砂面板」 */
  background: rgba(5, 5, 12, 0.55);
  backdrop-filter: blur(8px);
  -webkit-backdrop-filter: blur(8px);
  box-shadow: 0 0 80px 10px rgba(138, 43, 226, 0.06);
}

.cta-inner .section-desc {
  max-width: 580px;
  margin-inline: auto;
}

/* ---------- 页脚 ---------- */
.footer {
  padding: 64px 0 40px;
  /* 去掉顶线与实色底：改为由上到下渐深的半透明遮罩，与页面光晕自然衔接 */
  background: linear-gradient(
    180deg,
    rgba(5, 5, 10, 0) 0%,
    rgba(5, 5, 10, 0.55) 40%,
    rgba(5, 5, 10, 0.9) 100%
  );
}

.footer-inner {
  display: grid;
  /* 品牌 + 站内导航 + 开服状态 三列 */
  grid-template-columns: 1.6fr 1fr 1fr;
  gap: clamp(28px, 4vw, 56px);
  align-items: start;
}

.footer-brand {
  display: flex;
  gap: 12px;
}

.footer-brand strong {
  display: block;
  color: #fff;
  font-size: 16px;
  letter-spacing: 0.06em;
}

.footer-brand p {
  margin-top: 6px;
  max-width: 260px;
  color: var(--text-muted);
  font-size: 14px;
}

.footer-title {
  margin-bottom: 16px;
  color: #fff;
  font-size: 14px;
  font-weight: 600;
  letter-spacing: 0.08em;
}

.footer-list {
  display: grid;
  gap: 10px;
  margin: 0;
  padding: 0;
  list-style: none;
  font-size: 14px;
}

.footer-list a {
  color: var(--text-muted);
  transition: color 0.2s ease;
}

.footer-list a:hover {
  color: #fff;
}

.footer-list--meta li {
  display: flex;
  justify-content: space-between;
  gap: 12px;
}

.footer-list--meta span {
  color: var(--text-muted);
}

.footer-list--meta strong {
  color: #fff;
  font-weight: 600;
}

.footer-bottom {
  display: flex;
  flex-wrap: wrap;
  justify-content: space-between;
  gap: 10px;
  margin-top: 40px;
  /* 内部横线已去掉，收掉为它留的上边距 */
  color: var(--text-muted);
  font-size: 13px;
}

/* ---------- 魔法弹窗 ---------- */
.modal {
  position: fixed;
  inset: 0;
  z-index: 100;
  display: grid;
  place-items: center;
  padding: 24px;
  background: rgba(6, 6, 11, 0.74);
  backdrop-filter: blur(6px);
}

.modal-card {
  position: relative;
  overflow: hidden;
  width: min(420px, 100%);
  padding: 40px 32px 34px;
  border: 1px solid var(--border);
  border-radius: 24px;
  text-align: center;
  background:
    radial-gradient(120% 110% at 50% 0%, rgba(168, 85, 247, 0.26), transparent 62%),
    rgba(138, 43, 226, 0.12);
  box-shadow:
    inset 0 1px 0 rgba(255, 255, 255, 0.1),
    0 40px 80px -30px rgba(168, 85, 247, 0.7);
}

/* 卡片内缓慢旋转的符文环 */
.modal-ring {
  position: absolute;
  top: 50%;
  left: 50%;
  width: 320px;
  height: 320px;
  border-radius: 50%;
  background: repeating-conic-gradient(
    from 0deg,
    rgba(196, 150, 255, 0.5) 0deg 0.5deg,
    transparent 0.5deg 12deg
  );
  opacity: 0.22;
  -webkit-mask-image: radial-gradient(circle, transparent 32%, #000 44%, #000 70%, transparent 82%);
  mask-image: radial-gradient(circle, transparent 32%, #000 44%, #000 70%, transparent 82%);
  transform: translate(-50%, -50%) rotate(0deg);
  animation: magic-circle-spin 40s linear infinite;
  pointer-events: none;
}

.modal-icon {
  position: relative;
  display: grid;
  place-items: center;
  width: 56px;
  height: 56px;
  margin: 0 auto 20px;
  border: 1px solid var(--border);
  border-radius: 50%;
  background: var(--primary-soft);
  color: #d8b4fe;
  box-shadow: 0 0 28px -6px rgba(168, 85, 247, 0.95);
}

.modal-icon svg {
  width: 26px;
  height: 26px;
}

.modal-title {
  position: relative;
  margin-bottom: 10px;
  color: #fff;
  font-size: clamp(19px, 2.4vw, 22px);
  font-weight: 650;
  letter-spacing: 0.04em;
}

.modal-text {
  position: relative;
  margin-bottom: 26px;
  color: var(--text-muted);
  font-size: 15px;
}

.modal-enter-active,
.modal-leave-active {
  transition: opacity 0.25s ease;
}

.modal-enter-active .modal-card,
.modal-leave-active .modal-card {
  transition:
    opacity 0.25s ease,
    transform 0.25s ease;
}

.modal-enter-from,
.modal-leave-to {
  opacity: 0;
}

.modal-enter-from .modal-card,
.modal-leave-to .modal-card {
  opacity: 0;
  transform: translateY(14px) scale(0.94);
}

/* ---------- 回到顶部 ---------- */
.to-top {
  position: fixed;
  right: clamp(16px, 3vw, 34px);
  bottom: clamp(18px, 3vw, 34px);
  z-index: 60;
  display: grid;
  place-items: center;
  width: 46px;
  height: 46px;
  border: 1px solid var(--border);
  border-radius: 50%;
  background: rgba(12, 10, 22, 0.82);
  backdrop-filter: blur(10px);
  color: #dcc9ff;
  cursor: pointer;
  box-shadow: 0 18px 40px -20px rgba(168, 85, 247, 0.95);
  transition:
    transform 0.25s ease,
    box-shadow 0.25s ease,
    border-color 0.25s ease,
    color 0.25s ease;
}

.to-top svg {
  width: 20px;
  height: 20px;
}

.to-top:hover {
  transform: translateY(-3px);
  border-color: var(--primary);
  color: #fff;
  box-shadow: 0 24px 48px -18px rgba(168, 85, 247, 1);
}

.to-top-enter-active,
.to-top-leave-active {
  transition:
    opacity 0.25s ease,
    transform 0.25s ease;
}

.to-top-enter-from,
.to-top-leave-to {
  opacity: 0;
  transform: translateY(10px) scale(0.9);
}

/* ---------- 响应式 ---------- */
@media (max-width: 1024px) {
  .feature-grid {
    grid-template-columns: repeat(2, 1fr);
  }

  .footer-inner {
    grid-template-columns: 1fr 1fr;
  }

  .footer-brand {
    grid-column: 1 / -1;
  }
}

@media (max-width: 900px) {
  .gameplay-grid {
    grid-template-columns: 1fr;
  }

  .brand-tag {
    display: none;
  }
}

@media (max-width: 860px) {
  .nav-links {
    display: none;
  }

  .nav-inner {
    justify-content: space-between;
  }
}

@media (max-width: 760px) {
  .server-meta {
    grid-template-columns: 1fr;
  }

  /* 小屏关掉卡片的 backdrop-filter：背景的粒子层在持续动画，
     6 个模糊区域要逐帧重算，手机上纯属白耗性能，而磨砂质感在小屏上肉眼也分辨不出。
     卡片底色不变，视觉上几乎无差别。 */
  .feature-card,
  .server-card,
  .cta-inner {
    backdrop-filter: none;
    -webkit-backdrop-filter: none;
  }
}

@media (max-width: 600px) {
  .container {
    width: min(1180px, 100% - 32px);
  }

  .feature-grid {
    grid-template-columns: 1fr;
  }

  .footer-inner {
    grid-template-columns: 1fr;
  }

  /* 小屏降低光效开销，避免糊成一片 */
  .hero-aurora {
    filter: blur(46px);
    opacity: 0.6;
  }

  .to-top {
    width: 42px;
    height: 42px;
  }
}
</style>