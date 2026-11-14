# Claude Code statusline mod 설치 프롬프트

아래 내용을 다른 PC의 Claude Code 대화창에 그대로 붙여 넣으면 됩니다. 이 PC에서 만든 `statusline-band` mod와 관련 설정을 재현합니다.

---

다음 작업을 순서대로 수행해 주세요. 파일 내용은 한 글자도 바꾸지 말고 그대로 작성해야 합니다. 각 단계가 끝날 때마다 결과를 한국어로 짧게 알려 주세요.

## 배경

입력창 위 밴드(`AbovePrompt`)에 상태 표시줄을 그리는 mod입니다. 표시 내용은 다음과 같습니다.

- 첫 줄: 모델 이모지와 모델명(예: Sonnet 5.5), effort, 폴더, git 브랜치(변경 파일 수 `*3`, 올릴 커밋 `↑1`, 받을 커밋 `↓2`는 0이 아닐 때만), 그리고 흐리게 계정 이메일과 구독 종류
- 둘째~넷째 줄: Context(자동 압축을 설정했다면 압축 기준 %도 함께, "⚠ 컴팩트 임박" 경고는 압축 기준보다 10%p 앞에서, 설정이 없으면 90%부터), 5시간 한도, 7일 한도(사용률이 기간 경과율보다 20%p 이상 앞서면 "⚠ 빠름")를 한 줄에 하나씩, 라벨·막대·%·부가 정보가 세로로 맞게 표시(막대 색이 반 칸 단위로 부드럽게 이어지고, % 숫자는 90% 이상이면 빨강)
- 테두리: 평소에는 파란색, 작업 중에는 파랑과 보라가 시계 방향으로 흐르며, 응답이 끝난 뒤 3초 동안 유지
- 마지막 줄: CPU·RAM 막대(10초마다 갱신), 비용, 세션 경과 시간

## 단계 0. 적용 범위 선택

다른 작업을 하기 전에 AskUserQuestion 도구로 아래 질문을 하고, 답을 받은 뒤에 진행해 주세요.

- 질문: "statusline-band를 어디에 적용할까요?"
- 선택지 1: **전역** — 모든 프로젝트에서 사용
- 선택지 2: **이 프로젝트만** — 현재 작업 폴더에서만 사용

선택에 따라 아래 두 값을 정하고, 이후 단계의 `<MOD_DIR>`과 `<SETTINGS_FILE>`을 이 값으로 바꿔 읽어 주세요.

| | `<MOD_DIR>` | `<SETTINGS_FILE>` |
|---|---|---|
| 전역 | `~/.claude/skills/statusline-band` | `~/.claude/settings.json` |
| 이 프로젝트만 | `<현재 작업 폴더>/.claude/skills/statusline-band` | `<현재 작업 폴더>/.claude/settings.local.json` |

- `skills` 폴더 아래의 plugin은 `CLAUDE_CODE_PLUGIN_DIRS` 없이 자동으로 인식되고, 재시작 없이 `/reload-plugins`로 불러올 수 있습니다.
- 프로젝트 범위는 이 프로젝트의 `.claude/skills`에만 설치되고 다른 프로젝트에는 영향이 없습니다. 설정은 개인 설정 파일(`settings.local.json`)에 넣어 저장소에 커밋되지 않게 합니다.

## 단계 0-2. 기존 상태 표시줄 정리

설치 전에 이 mod와 같은 일(상태 표시줄, 입력창 위 밴드)을 하는 기존 항목이 있는지 찾아 주세요. 찾기만 하고, 아직 아무것도 지우거나 고치지 마세요.

찾을 곳:
- **`/statusline`으로 만든 항목:** `~/.claude/settings.json`, `<현재 작업 폴더>/.claude/settings.json`, `<현재 작업 폴더>/.claude/settings.local.json`의 `statusLine` 항목과, 그 `command`가 가리키는 스크립트 파일(예: `~/.claude/statusline.sh`, `statusline.ps1`)
- **비슷한 mod·plugin:** `~/.claude/mods`, `~/.claude/skills`, `<현재 작업 폴더>/.claude/skills`, 두 settings 파일의 `enabledPlugins`에서, `hooks.json`의 `modules`에 `ui.render`(`AbovePrompt` 등)나 상태 표시줄 역할이 보이는 항목
- **`CLAUDE_CODE_PLUGIN_DIRS`:** 이전 버전 프롬프트로 설치해 `~/.claude/mods/statusline-band`를 가리키는 항목. 이 경우는 이전 설치본이므로 아래 선택지에 함께 넣어 주세요.

찾은 항목이 있으면 AskUserQuestion으로 항목마다 위치와 내용을 한 줄로 보여 주고 묻습니다.

- 질문: "기존 `<항목>`을 삭제하고 statusline-band로 대체할까요?"
- 선택지 1: **삭제하고 대체** — 해당 설정 항목과 스크립트 파일(그 항목만 쓰는 것에 한함)을 지웁니다
- 선택지 2: **그대로 두고 함께 사용** — 아무것도 지우지 않습니다. 화면에 겹쳐 보일 수 있다고 알려 주세요

규칙:
- 사용자가 삭제를 고른 항목만 처리하세요. 고르지 않았거나 답이 없으면 건드리지 마세요.
- 삭제하기 전에 지울 내용(설정 항목 원문, 파일 경로)을 보여 주세요. 설정 파일은 해당 항목만 제거하고 나머지는 그대로 두며, 수정 후 올바른 JSON인지 확인하세요.
- 다른 곳에서도 쓰일 수 있는 스크립트나 plugin은 지우지 말고, 어디서 쓰이는지 알려 준 뒤 다시 확인하세요.
- 찾은 항목이 없으면 "기존 항목 없음"이라고만 알리고 넘어가세요.

## 단계 0-3. 자동 압축 설정 선택

컨텍스트가 일정 비율에 도달하면 자동 압축(compact)이 실행되게 할지 AskUserQuestion으로 묻습니다. 단계 0의 선택에 맞는 `<SETTINGS_FILE>`을 읽어 `CLAUDE_AUTOCOMPACT_PCT_OVERRIDE`가 이미 있으면, 현재 값을 질문에 함께 적어 주세요.

**질문 1**
- 질문: "컨텍스트가 일정 비율에 도달하면 자동 압축하도록 설정할까요?"
- 선택지 1: **설정함** — 다음 질문에서 비율을 숫자로 입력합니다
- 선택지 2: **설정 안 함** — 이 설정은 건드리지 않습니다 (기존 값이 있으면 그대로 유지)

**질문 2** (질문 1에서 "설정함"을 고른 경우에만)
- 질문: "컨텍스트가 몇 %일 때 자동 압축할까요? 1~99 사이 숫자를 입력해 주세요."
- 고정 선택지(40/60/80 등)는 두지 않습니다. 사용자가 숫자를 직접 입력하게 하세요. (선택지 도구를 쓴다면 "Other" 입력으로 받습니다.)
- 숫자를 받으면 아래처럼 20칸 프로그레스바로 보여 주고, 이 값이 맞는지 확인하세요. 아니라고 하면 다시 입력받습니다.
  `[██████████░░░░░░░░░░] 50%` (채운 칸 수 = 값 ÷ 5, 반올림)

규칙:
- 입력값은 1~99 사이의 정수여야 합니다. 벗어나거나 숫자가 아니면 다시 물어보세요.
- 이 값은 기본 자동 압축 기준보다 **낮출 때만** 효과가 있습니다. 높은 값을 고르면 효과가 없을 수 있다고 한 줄로 알려 주세요.
- 고른 값을 이후 단계의 `<N>`으로 사용합니다. "설정 안 함"이면 단계 2와 단계 4의 자동 압축 관련 내용을 건너뜁니다.

## 단계 1. mod 폴더 만들기

`<MOD_DIR>` 폴더를 만들고(`~`는 사용자 홈 디렉터리), 그 안에 아래 파일 5개를 같은 상대 경로로 작성해 주세요. 폴더가 이미 있으면 덮어쓰기 전에 기존 내용을 알려 주고 저와 확인해 주세요.

### 파일 1: `.claude-plugin/plugin.json`

````json
{
  "name": "statusline-band",
  "version": "0.1.0",
  "description": "프롬프트 위 밴드에 모델, 컨텍스트, 5시간·7일 한도, 비용을 그라데이션 막대로 표시한다",
  "types": "./types/index.d.ts"
}
````

### 파일 2: `hooks/hooks.json`

````json
{ "modules": ["./register.tsx"] }
````

### 파일 3: `types/index.d.ts`

````ts
export type Limit = { kind: string; pct: number; resetAt: number | null }

// 작업 중 테두리 애니메이션 여부: 처리 중이거나 응답이 끝난 직후 유지 시간 동안 true다.
export type Flow = { animating: boolean }

export type Machine = { cpuPct: number; ramUsedGb: number; ramTotalGb: number }

export type Snap = {
  email: string | null
  plan: string | null
  model: string
  effort: string | null
  dir: string
  branch: string
  dirty: number
  ahead: number
  behind: number
  ctxPct: number | null
  ctxTokens: number | null
  ctxWindow: number
  compactPct: number | null
  limits: Limit[]
  usd: number | null
  elapsedMs: number
  at: number
}

declare module 'claude-code' {
  interface PluginState {
    'statusline-band': { snap: Snap | null; machine: Machine | null; frame: number; flow: Flow }
  }
}
````

### 파일 4: `hooks/register.tsx`

````tsx
import { atom, read, update } from 'claude-code'
import type { EngineInterface, Register } from 'claude-code'

import type { Flow, Limit, Snap } from '../types'

const snapAtom = atom({ plugin: 'statusline-band', key: 'snap' } as const, null)
const machineAtom = atom({ plugin: 'statusline-band', key: 'machine' } as const, null)
const frameAtom = atom({ plugin: 'statusline-band', key: 'frame' } as const, 0)

const flowAtom = atom({ plugin: 'statusline-band', key: 'flow' } as const, { animating: false })

const FRAME_INTERVAL_MS = 200
const FLOW_EVERY_TICKS = 4 // 테두리 상태는 1초마다 갱신한다
const LINGER_MS = 3000 // 응답이 끝난 뒤에도 작업 상태를 유지하는 시간
const STUCK_TICKS = 10 // 엔진은 처리 중이 아닌데 내 표시만 이만큼 켜져 있으면 끝난 것으로 본다(2초)
const BORDER_SEG = 6 // 테두리를 이 글자 수 단위로 묶어 한 색으로 칠해 요소 수를 줄인다
const LAP_FRAMES = 30 // 색이 테두리를 한 바퀴 도는 프레임 수(0.2초 × 30 = 6초)
const BORDER_A = [0x5b, 0x9b, 0xf0] // 파랑
const BORDER_B = [0xc6, 0x78, 0xdd] // 보라

// 작업 상태. 타이머와 훅이 함께 쓰므로 모듈 변수로 둔다.
let isWorking = false
let lingerTicks = 0 // 응답이 끝난 뒤 작업 상태를 더 유지할 프레임 수
let engineWorking = false // 마지막으로 그릴 때 엔진이 알려 준 처리 중 여부
let offTicks = 0
let flowShown = false
let tickN = 0

// 테두리가 도는 상태(처리 중이거나 응답 직후 유지 시간)를 화면용 값으로 내보낸다.
async function publishFlow($: EngineInterface) {
  const animating = isWorking || lingerTicks > 0
  flowShown = animating
  await update($, flowAtom, () => ({ animating }))
}

// 처리 중이거나 응답 직후 유지 시간인 동안에만 일한다. 둘 다 끝나면 테두리 상태를 한 번 지우고 멈춘다.
async function tickFrame($: EngineInterface) {
  tickN += 1
  // 엔진은 처리 중이 아니라는데 내 표시만 켜져 있으면(종료 이벤트를 놓친 경우) 잠시 뒤 끝난 것으로 본다.
  offTicks = isWorking && !engineWorking ? offTicks + 1 : 0
  if (offTicks >= STUCK_TICKS) {
    isWorking = false
    lingerTicks = Math.ceil(LINGER_MS / FRAME_INTERVAL_MS)
    offTicks = 0
  }
  const animating = isWorking || lingerTicks > 0
  if (!animating && !flowShown) return
  if (animating) await update($, frameAtom, n => n + 1)
  if (lingerTicks > 0) lingerTicks -= 1
  if (tickN % FLOW_EVERY_TICKS === 0 || !animating) await publishFlow($)
}

// 테두리 상자의 크기(칸 수). 위치는 둘레를 따라 시계 방향으로 잰 거리로 나타낸다.
type Frame2D = { w: number; h: number }

const perimeter = (b: Frame2D) => 2 * (b.w - 1) + 2 * (b.h - 1)

// 왼쪽 위 모서리에서 시작해 위(→), 오른쪽(↓), 아래(←), 왼쪽(↑) 순서로 거리가 늘어난다.
const distTop = (b: Frame2D, x: number) => x
const distRight = (b: Frame2D, y: number) => b.w - 1 + y
const distBottom = (b: Frame2D, x: number) => b.w - 1 + (b.h - 1) + (b.w - 1 - x)
const distLeft = (b: Frame2D, y: number) => 2 * (b.w - 1) + (b.h - 1) + (b.h - 1 - y)

// 거리 s 칸의 색. 파랑과 보라가 둘레를 한 바퀴에 한 번 오가며, 프레임이 오를수록 시계 방향으로 흐른다.
function flowColor(s: number, frame: number, b: Frame2D): string {
  const len = perimeter(b)
  const phase = (s - (frame * len) / LAP_FRAMES) / len
  const t = (1 + Math.cos(2 * Math.PI * phase)) / 2
  return `#${BORDER_A.map((a, i) => hex(a + (BORDER_B[i]! - a) * t)).join('')}`
}

// 가로 테두리 한 줄을 색이 조금씩 다른 조각으로 나눈다. distOf는 x칸의 시계 방향 거리를 돌려준다.
function borderSegments(
  left: string,
  mid: string,
  right: string,
  b: Frame2D,
  frame: number,
  distOf: (x: number) => number,
) {
  const line = left + mid.repeat(Math.max(0, b.w - 2)) + right
  const out: { text: string; color: string }[] = []
  for (let i = 0; i < line.length; i += BORDER_SEG) {
    out.push({ text: line.slice(i, i + BORDER_SEG), color: flowColor(distOf(i + BORDER_SEG / 2), frame, b) })
  }
  return out
}

// 한글 같은 전각 글자는 2칸으로 세어, 게이지 라벨을 같은 칸 수에 맞춘다.
const LABEL_CELLS = 7
const cellWidth = (s: string) => [...s].reduce((n, ch) => n + (ch.charCodeAt(0) > 0x2e7f ? 2 : 1), 0)
const padCells = (s: string, n: number) => s + ' '.repeat(Math.max(0, n - cellWidth(s)))
const FOOT_BAR = 5 // 하단 CPU·RAM 막대 칸 수(윗줄 게이지보다 작게)
const WARN_DEFAULT = 90 // 자동 압축 기준을 설정하지 않았을 때 "컴팩트 임박"을 알리는 Context %
const WARN_MARGIN = 10 // 자동 압축 기준을 설정했을 때, 기준보다 이만큼(%p) 앞에서 알린다

const MACHINE_INTERVAL_MS = 10_000
const HEAD_INTERVAL_MS = 2_000
const MACHINE_SCRIPT =
  '$o=Get-CimInstance Win32_OperatingSystem;' +
  '[pscustomobject]@{cpu=(Get-CimInstance Win32_Processor|Measure-Object LoadPercentage -Average).Average;' +
  'total=$o.TotalVisibleMemorySize;free=$o.FreePhysicalMemory}|ConvertTo-Json -Compress'

// CPU 부하와 메모리 사용량(시스템 전체)을 PowerShell로 읽는다. 실패하면 이전 값을 유지한다.
async function sampleMachine($: EngineInterface) {
  try {
    const r = await $.process.run(['powershell', '-NoProfile', '-Command', MACHINE_SCRIPT], { timeoutMs: 8000 })
    if (r.exitCode !== 0) return
    const v = JSON.parse(r.stdout.trim()) as { cpu: number | null; total: number; free: number }
    const gb = (kb: number) => kb / 1024 / 1024
    await update($, machineAtom, () => ({
      cpuPct: v.cpu ?? 0,
      ramUsedGb: gb(v.total - v.free),
      ramTotalGb: gb(v.total),
    }))
  } catch {
    /* 측정 실패는 표시에만 영향을 준다 */
  }
}

// 막대 색 구간: 민트 초록 → 연한 노랑 → 연한 코랄. 채도를 낮춰 구간 사이의 대비를 줄였다.
const GRADIENT = [
  { t: 0, r: 76, g: 195, b: 138 },
  { t: 0.5, r: 227, g: 197, b: 107 },
  { t: 1, r: 232, g: 130, b: 111 },
]
const TRACK = '#373a48'
const GRAY = '#8a8f9c'
const WIDTH = 8
const EIGHTHS = [' ', '▏', '▎', '▍', '▌', '▋', '▊', '▉']

const hex = (n: number) => Math.round(n).toString(16).padStart(2, '0')

function gradient(t: number): string {
  const x = Math.max(0, Math.min(1, t))
  for (let i = 0; i < GRADIENT.length - 1; i++) {
    const a = GRADIENT[i]!
    const b = GRADIENT[i + 1]!
    if (x >= a.t && x <= b.t) {
      const k = (x - a.t) / (b.t - a.t)
      return `#${hex(a.r + (b.r - a.r) * k)}${hex(a.g + (b.g - a.g) * k)}${hex(a.b + (b.b - a.b) * k)}`
    }
  }
  return '#e04444'
}

function formatTokens(n: number): string {
  if (n >= 1_000_000) {
    const m = n / 1_000_000
    return Number.isInteger(m) ? `${m}M` : `${m.toFixed(1)}M`
  }
  return `${Math.round(n / 1000)}k`
}

function resetIn(resetAt: number | null, now: number): string {
  if (resetAt === null) return ''
  const diff = Math.floor((resetAt - now) / 1000)
  if (diff <= 0) return '곧'
  const d = Math.floor(diff / 86400)
  const h = Math.floor((diff % 86400) / 3600)
  const m = Math.floor((diff % 3600) / 60)
  if (d > 0) return `${d}일 ${h}시간 후`
  if (h > 0) return `${h}시간 ${m}분 후`
  return `${m}분 후`
}

// 내부 모델 이름에서 계열과 버전을 읽는다. 계열(opus, sonnet 등) 목록을 따로 두지 않으므로
// 새 계열이 나와도 그대로 처리된다. claude-sonnet-5-5 → sonnet / 5.5, claude-nova-6-1 → nova / 6.1.
function parseModel(raw: string): { family: string; version: string } | null {
  const version = (major: string, minor?: string) => (minor ? `${major}.${minor}` : major)

  // claude-sonnet-5-5, claude-opus-4-1-20250805, us.anthropic.claude-sonnet-5-5-v1:0 (날짜·버전 꼬리는 버린다)
  const modern = raw.match(/claude[-_ ]([a-z][a-z0-9]*)[-_ ](\d+)(?:[-.](\d{1,2})(?!\d))?/i)
  if (modern) return { family: modern[1]!.toLowerCase(), version: version(modern[2]!, modern[3]) }

  // claude-3-5-sonnet 처럼 버전이 앞에 오는 옛 이름
  const legacy = raw.match(/claude[-_ ](\d+)(?:[-.](\d{1,2}))?[-_ ]([a-z][a-z0-9]*)/i)
  if (legacy) return { family: legacy[3]!.toLowerCase(), version: version(legacy[1]!, legacy[2]) }

  return null
}

const cap = (f: string) => f.charAt(0).toUpperCase() + f.slice(1)

// 화면용 이름: Sonnet 5.5, Fable 5.2. 버전 없는 별칭(opus, sonnet[1m])은 계열만, 못 알아보는 이름은 그대로 보여 준다.
function modelLabel(raw: string): string {
  const p = parseModel(raw)
  if (p) return `${cap(p.family)} ${p.version}`
  const alias = raw.match(/^([a-z]+)(\[[^\]]*\])?$/i)
  return alias ? cap(alias[1]!.toLowerCase()) : raw
}

// 계열별 이모지. 아는 계열은 고정 이모지를, 새 계열은 이름에서 정해지는 이모지를 쓴다(같은 이름은 항상 같은 이모지).
const FAMILY_EMOJI: Record<string, string> = { opus: '👑', sonnet: '🎻', haiku: '🌸', fable: '📖' }
const EMOJI_POOL = ['🌟', '🔮', '🧩', '🌙', '🍀', '🎯', '🪐', '🦋']

function modelEmoji(raw: string): string {
  const family = parseModel(raw)?.family ?? raw.match(/^([a-z]+)/i)?.[1]?.toLowerCase()
  if (!family) return '🤖'
  const known = FAMILY_EMOJI[family]
  if (known) return known
  let h = 0
  for (const ch of family) h = (h * 31 + ch.charCodeAt(0)) % EMOJI_POOL.length
  return EMOJI_POOL[h]!
}

const EFFORT_COLOR: Record<string, string> = {
  low: GRAY,
  medium: '#4fc3d9',
  high: '#e6b83e',
  xhigh: '#c678dd',
  max: '#e04444',
}

let account: { email: string | null; plan: string | null } | null = null

async function readJson($: EngineInterface, path: string): Promise<any> {
  try {
    return JSON.parse(await $.fs.read(path))
  } catch {
    return null
  }
}

// 계정 정보는 바뀌는 일이 드물어 한 번만 읽어 둔다.
async function loadAccount($: EngineInterface) {
  if (account !== null) return account
  const home = (await $.env.get('USERPROFILE')) ?? (await $.env.get('HOME')) ?? ''
  const cred = await readJson($, `${home}/.claude/.credentials.json`)
  const cfg = await readJson($, `${home}/.claude.json`)
  account = {
    email: cfg?.oauthAccount?.emailAddress ?? null,
    plan: cred?.claudeAiOauth?.subscriptionType ?? null,
  }
  return account
}

// 현재 effort는 CLAUDE_EFFORT 환경 변수에 바로 반영된다. 없을 때만 설정값으로 추정한다.
async function currentEffort($: EngineInterface, model: string): Promise<string | null> {
  const fromEnv = await $.env.get('CLAUDE_EFFORT')
  if (fromEnv) return fromEnv
  try {
    const s = (await $.settings.read()) as {
      effortLevel?: string
      modelSettings?: Record<string, { effortLevel?: string }>
    }
    const byModel = Object.entries(s.modelSettings ?? {})
      .filter(([id]) => model.includes(id))
      .sort((a, b) => b[0].length - a[0].length)[0]
    return byModel?.[1].effortLevel ?? s.effortLevel ?? null
  } catch {
    return null
  }
}

// 자동 압축 기준(%). 환경 변수가 없거나 1~99 정수가 아니면 null이라 표시하지 않는다.
async function compactThreshold($: EngineInterface): Promise<number | null> {
  const v = Number(await $.env.get('CLAUDE_AUTOCOMPACT_PCT_OVERRIDE'))
  return Number.isInteger(v) && v >= 1 && v <= 99 ? v : null
}

// `git status --porcelain -b` 출력에서 브랜치, 변경 파일 수, 올릴·받을 커밋 수를 읽는다.
// 첫 줄은 "## main...origin/main [ahead 1, behind 2]" 꼴이고, 나머지 줄은 변경된 파일(추적 안 되는 파일 포함) 하나씩이다.
function parseGit(out: string): { branch: string; dirty: number; ahead: number; behind: number } {
  const lines = out.split(/\r?\n/).filter(Boolean)
  const head = lines[0]?.startsWith('## ') ? lines[0].slice(3) : ''
  const name = head.split('...')[0]!.replace(/^(No commits yet on|Initial commit on) /, '').trim()
  return {
    branch: name.startsWith('HEAD (no branch)') ? '' : name,
    dirty: lines.filter(l => !l.startsWith('## ')).length,
    ahead: Number(head.match(/ahead (\d+)/)?.[1] ?? 0),
    behind: Number(head.match(/behind (\d+)/)?.[1] ?? 0),
  }
}

const WINDOW_MS: Record<string, number> = { five_hour: 5 * 3600_000, seven_day: 7 * 86400_000 }
const PACE_MARGIN = 20 // 한도 사용률이 기간 경과율보다 이만큼(%p) 앞서면 "빠름"으로 알린다

// 한도 기간이 지난 비율(경과율)보다 사용률이 많이 앞서면, 이 속도로는 기간이 끝나기 전에 한도가 찬다는 뜻이다.
function paceWarn(kind: string, pct: number, resetAt: number | null, now: number): string {
  const win = WINDOW_MS[kind]
  if (win === undefined || resetAt === null) return ''
  const elapsedPct = Math.max(0, Math.min(100, (1 - (resetAt - now) / win) * 100))
  return pct - elapsedPct >= PACE_MARGIN ? ' ⚠ 빠름' : ''
}

// /model, /effort는 응답 없이 바뀌므로 짧은 주기로 두 값만 확인해 바뀌었을 때만 고친다.
async function refreshHead($: EngineInterface) {
  const snap = await read($, snapAtom)
  if (snap === null) return
  const model = await $.session.model()
  const effort = await currentEffort($, model)
  if (model === snap.model && effort === snap.effort) return
  await update($, snapAtom, s => (s === null ? s : { ...s, model, effort }))
}

async function refresh($: EngineInterface) {
  const usage = await $.session.usage()
  const model = await $.session.model()
  const cwd = await $.session.cwd()
  const now = await $.clock.now()
  const acc = await loadAccount($)
  const effort = await currentEffort($, model)

  let git = { branch: '', dirty: 0, ahead: 0, behind: 0 }
  try {
    const r = await $.process.run(['git', 'status', '--porcelain', '-b'], { cwd, timeoutMs: 3000 })
    if (r.exitCode === 0) git = parseGit(r.stdout)
  } catch {
    /* git 저장소가 아니거나 git이 없는 경우 */
  }

  const limits: Limit[] = usage.rateLimits.map(l => ({
    kind: l.kind,
    pct: l.percentUsed,
    resetAt: l.resetsAt ? Date.parse(l.resetsAt) : null,
  }))

  const snap: Snap = {
    email: acc.email,
    plan: acc.plan,
    model,
    effort,
    dir: cwd.split(/[\\/]/).filter(Boolean).pop() ?? cwd,
    ...git,
    ctxPct: usage.context.percent ?? null,
    ctxTokens: usage.context.tokens ?? null,
    ctxWindow: usage.context.window,
    compactPct: await compactThreshold($),
    limits,
    usd: usage.cost?.usd ?? null,
    elapsedMs: now - usage.startedAt,
    at: now,
  }
  await update($, snapAtom, () => snap)
}

export const register: Register = on => {
  let stopTimers: (() => void) | null = null

  on('session.start', async ($, e, next) => {
    await refresh($)
    await sampleMachine($)
    stopTimers?.()
    const machineTimer = $.clock.every(MACHINE_INTERVAL_MS, () => sampleMachine($))
    const frameTimer = $.clock.every(FRAME_INTERVAL_MS, () => tickFrame($))
    const headTimer = $.clock.every(HEAD_INTERVAL_MS, () => refreshHead($))
    stopTimers = () => {
      machineTimer.cancel()
      frameTimer.cancel()
      headTimer.cancel()
    }
    return next(e)
  })

  on('session.measure', async ($, e, next) => {
    await refresh($)
    return next(e)
  })

  // 메인 응답의 시작과 끝으로 처리 중 여부를 정한다. prompt.submit은 시스템 메타 프롬프트나
  // 대기열 입력에도 발생해 끝 이벤트 없이 켜진 채 남을 수 있어 쓰지 않는다.
  on('turn.start', async ($, e, next) => {
    if (e.agentId === undefined) {
      lingerTicks = 0
      offTicks = 0
      engineWorking = true // 시작 직후 엔진 표시가 따라올 때까지 잠시 기다린다
      isWorking = true
      await publishFlow($)
    }
    return next(e)
  })

  on('turn.complete', async ($, e, next) => {
    if (e.agentId === undefined) {
      isWorking = false
      lingerTicks = Math.ceil(LINGER_MS / FRAME_INTERVAL_MS)
    }
    await refresh($)
    await publishFlow($)
    return next(e)
  })

  on('ui.render', { component: 'AbovePrompt' }, async ($, e, next) => {
    const snap = await read($, snapAtom)
    const machine = await read($, machineAtom)
    const frame = await read($, frameAtom)
    const flow = await read($, flowAtom)
    engineWorking = e.props.isWorking
    if (e.props.hasSurvey || snap === null) return next(e)

    const { Box, Text } = $.ui.resolve(e)

    const bar = (pct: number, width = WIDTH) => {
      const p = Math.max(0, Math.min(100, pct))
      // 8분의 1칸 단위로 먼저 반올림해야 7.96칸 같은 값이 한 칸을 잃지 않는다.
      const totalEighths = Math.round((p * width * 8) / 100)
      const filled = Math.floor(totalEighths / 8)
      const eighth = totalEighths % 8
      const hasPartial = eighth > 0
      const empty = width - filled - (hasPartial ? 1 : 0)
      const cells = []
      // 채운 칸마다 '▌'의 글자색(왼쪽 반 칸)과 배경색(오른쪽 반 칸)을 따로 칠해, 8칸 막대에서도
      // 색이 16단계로 이어지게 한다. 색 경계가 뚜렷하게 끊겨 보이지 않는다.
      const at = (k: number) => gradient((k + 0.5) / (2 * width))
      for (let i = 0; i < filled; i++) {
        cells.push(<Text key={`f${i}`} color={at(2 * i)} backgroundColor={at(2 * i + 1)}>▌</Text>)
      }
      if (hasPartial) {
        cells.push(<Text key="p" color={at(2 * filled)}>{EIGHTHS[eighth]}</Text>)
      }
      if (empty > 0) cells.push(<Text key="e" color={TRACK}>{'░'.repeat(empty)}</Text>)
      return cells
    }

    // 90% 이상은 빨강, 그 아래는 초록 → 노랑 → 코랄 그라데이션
    const pctColor = (p: number) => (p >= 90 ? '#e04444' : gradient(p / 100))

    // 게이지 한 줄: 라벨 칸과 % 칸을 고정해, 위아래 줄의 막대와 숫자가 세로로 맞게 한다.
    const gauge = (key: string, icon: string, label: string, pct: number, tail: string, warn = '') => (
      <Text key={key}>
        {icon} <Text color={GRAY}>{padCells(label, LABEL_CELLS)}</Text> {bar(pct)}{'  '}
        <Text bold color={pctColor(pct)}>{`${Math.round(pct)}%`.padEnd(5)}</Text>
        <Text dimColor>{tail}</Text>
        <Text color="#e04444" bold>{warn}</Text>
      </Text>
    )

    // 한 줄에 여러 항목을 놓기 위한 조각: 아이콘 라벨 막대 백분율 (부가 정보)
    const seg = (key: string, icon: string, label: string, pct: number, tail: string) => (
      <Text key={key}>
        {icon} <Text color={GRAY}>{label}</Text> {bar(pct, FOOT_BAR)} <Text bold color={pctColor(pct)}>{Math.round(pct)}%</Text>
        <Text dimColor>{tail}</Text>
      </Text>
    )

    const rows: any[] = []

    // 첫 줄: 모델이 가장 눈에 띄고, 이메일·플랜은 흐리게 맨 끝에 둔다.
    rows.push(
      <Box key="head">
        <Text>
          <Text>{modelEmoji(snap.model)} </Text>
          <Text bold color="#5b9bf0">{modelLabel(snap.model)}</Text>
          {snap.effort ? <Text color={EFFORT_COLOR[snap.effort] ?? GRAY}> ⚡{snap.effort.toUpperCase()}</Text> : null}
          <Text color={GRAY}> │ </Text>
          <Text>📁 {snap.dir}</Text>
          {snap.branch ? <Text color="#2bbf6a"> │ 🌿 {snap.branch}</Text> : null}
          {snap.dirty > 0 ? <Text color="#e6b83e">{` *${snap.dirty}`}</Text> : null}
          {snap.ahead > 0 ? <Text color="#4fc3d9">{` ↑${snap.ahead}`}</Text> : null}
          {snap.behind > 0 ? <Text color="#e6b83e">{` ↓${snap.behind}`}</Text> : null}
          {snap.email || snap.plan ? <Text color={GRAY}> │ </Text> : null}
          {snap.email ? <Text color={GRAY}>👤 {snap.email}</Text> : null}
          {snap.plan ? <Text color={GRAY}> ✦ {snap.plan.toUpperCase()}</Text> : null}
        </Text>
      </Box>,
    )

    if (snap.ctxPct !== null) {
      const parts = [
        snap.ctxTokens !== null ? `${formatTokens(snap.ctxTokens)}/${formatTokens(snap.ctxWindow)}` : '',
        snap.compactPct !== null ? `${snap.compactPct}%에서 압축` : '',
      ].filter(Boolean)
      const warnAt = snap.compactPct !== null ? Math.max(1, snap.compactPct - WARN_MARGIN) : WARN_DEFAULT
      const warn = snap.ctxPct >= warnAt ? ' ⚠ 컴팩트 임박' : ''
      rows.push(gauge('ctx', '🧠', 'Context', snap.ctxPct, parts.join(' · '), warn))
    }

    for (const l of snap.limits) {
      const spec =
        l.kind === 'five_hour' ? { icon: '⏰', label: '5h 한도' } : l.kind === 'seven_day' ? { icon: '📅', label: '7d 한도' } : null
      if (spec === null) continue
      const r = resetIn(l.resetAt, snap.at)
      rows.push(gauge(l.kind, spec.icon, spec.label, l.pct, r ? `${r} 초기화` : '', paceWarn(l.kind, l.pct, l.resetAt, snap.at)))
    }

    // 맨 아래 줄: CPU·RAM 게이지와 비용·경과 시간을 한 줄에 모은다.
    const foot: any[] = []
    if (machine !== null) {
      const ramPct = (machine.ramUsedGb / machine.ramTotalGb) * 100
      foot.push(seg('cpu', '💻', 'CPU', machine.cpuPct, ''))
      foot.push(seg('ram', '💾', 'RAM', ramPct, ` ${machine.ramUsedGb.toFixed(1)}G`))
    }
    const isSubscription = snap.limits.length > 0
    if (snap.usd !== null) {
      foot.push(
        <Text key="usd" color={GRAY}>
          {isSubscription ? `💎 $${snap.usd.toFixed(3)} 상당` : `💰 $${snap.usd.toFixed(3)}`}
        </Text>,
      )
    }
    const mins = Math.floor(snap.elapsedMs / 60000)
    const secs = Math.floor((snap.elapsedMs % 60000) / 1000)
    foot.push(<Text key="time" color={GRAY}>{`⏳ ${mins}m ${secs}s`}</Text>)
    const footJoined: any[] = []
    foot.forEach((f, i) => {
      if (i > 0) footJoined.push(<Text key={`fsep${i}`} color={GRAY}> │ </Text>)
      footJoined.push(f)
    })
    rows.push(<Text key="foot">{footJoined}</Text>)

    // 대기 중: 정지한 파란 테두리. 처리 중이 아니어도 응답 직후 유지 시간(flow.animating) 중이면 처리 중 모양을 쓴다.
    if (!e.props.isWorking && !flow.animating) {
      return (
        <Box flexDirection="column" borderStyle="round" borderColor="#5b9bf0" paddingX={1}>
          {rows}
        </Box>
      )
    }

    // 처리 중: 기본 테두리로는 색 하나만 쓸 수 있어, 테두리를 직접 그려 파랑과 보라가
    // 왼쪽 위 → 오른쪽 위 → 오른쪽 아래 → 왼쪽 아래 → 왼쪽 위 순서로 흐르게 한다.
    const b: Frame2D = { w: e.props.bodyColumns, h: rows.length + 2 }
    const side = (kind: 'l' | 'r') => (
      <Box flexDirection="column">
        {rows.map((_, i) => (
          <Text key={`${kind}${i}`} color={flowColor(kind === 'l' ? distLeft(b, i + 1) : distRight(b, i + 1), frame, b)}>
            │
          </Text>
        ))}
      </Box>
    )
    const edge = (l: string, r: string, tag: string, distOf: (x: number) => number) => (
      <Text>
        {borderSegments(l, '─', r, b, frame, distOf).map((s, i) => (
          <Text key={`${tag}${i}`} color={s.color}>{s.text}</Text>
        ))}
      </Text>
    )

    return (
      <Box flexDirection="column">
        {edge('╭', '╮', 't', x => distTop(b, x))}
        <Box>
          {side('l')}
          <Box flexDirection="column" flexGrow={1} paddingX={1}>
            {rows}
          </Box>
          {side('r')}
        </Box>
        {edge('╰', '╯', 'b', x => distBottom(b, x))}
      </Box>
    )
  })
}
````

### 파일 5: `tests/flow.test.tsx`

````tsx
import { expect, test } from 'claude-code/testing'

const BAND = (isWorking: boolean, bodyColumns = 140) =>
  ({
    plugin: 'statusline-band',
    component: 'AbovePrompt',
    surface: 'terminal',
    props: {
      hasSurvey: false,
      isCollapsed: false,
      isWorking,
      maxRows: 30,
      bodyColumns,
      scroll: { offset: 0, bodyRows: 30 },
      view: {},
    },
  }) as const

// 엔진 아래층을 흉내 낸다: 상태는 메모리에, 나머지 호출은 고정 값으로 답한다.
function stubEngine(
  on: (...args: any[]) => void,
  opts: { pct: number; frame: number; model?: string; compact?: string; git?: string; five?: number; seven?: number } = {
    pct: 42,
    frame: 0,
  },
) {
  const store: Record<string, unknown> = {}
  let version = 1
  let clock = 1_000_000

  on('state.get', (_: unknown, e: any) => ({
    value: { value: e.key === 'frame' ? opts.frame : store[e.key], version },
  }))
  on('state.set', (_: unknown, e: any) => {
    store[e.key] = e.value
    version += 1
    return { value: { isSet: true, version } }
  })
  on('clock.now', () => ({ value: (clock += 1000) }))
  on('clock.every', () => ({ value: undefined }))
  on('session.start', (_: unknown, e: any) => ({ cwd: e.cwd }))
  on('session.model', () => ({ value: opts.model ?? 'claude-sonnet-5-5' }))
  on('session.cwd', () => ({ value: '/proj' }))
  on('session.usage', () => ({
    value: {
      startedAt: 1_000_000,
      context: { window: 200000, tokens: 84000, percent: opts.pct },
      rateLimits: [
        { kind: 'five_hour', percentUsed: opts.five ?? 12, resetsAt: new Date(1_000_000 + 3 * 3600_000).toISOString() },
        { kind: 'seven_day', percentUsed: opts.seven ?? 30, resetsAt: new Date(1_000_000 + 4 * 86400_000).toISOString() },
      ],
      cost: { usd: 1.234 },
    },
  }))
  on('env.get', (_: unknown, e: any) => {
    const name = e.name ?? e.key
    return { value: name === 'CLAUDE_EFFORT' ? 'high' : name === 'CLAUDE_AUTOCOMPACT_PCT_OVERRIDE' ? opts.compact : '/home' }
  })
  on('fs.read', () => ({ value: '{}' }))
  on('settings.read', () => ({ value: {} }))
  on('process.run', (_: unknown, e: any) => ({
    value: {
      exitCode: 0,
      stdout: e.argv[0] === 'git' ? (opts.git ?? '## main...origin/main\n') : '{"cpu":16,"total":16000000,"free":6000000}',
      stderr: '',
    },
  }))
  on('turn.start', () => ({ turnId: 't1' }))
  on('turn.complete', () => ({ text: '' }))
}

test('처리 중 밴드에 헤더, 한도, 도는 테두리가 그려지고 작업 흐름은 나오지 않는다', async ($, on) => {
  stubEngine(on)

  await $.session.start({ cwd: '/', surface: 'terminal', isInteractive: true })
  await $.turn.start({ text: '작업 시작', turnId: 't1' } as any)
  await $.turn.complete({} as any)

  const ui = await $.ui.mount(BAND(true))
  const texts = (await ui.findAll({ type: 'Text' })).map(t => t.text)
  const all = texts.join('|')

  expect(all).toContain('Sonnet 5.5')
  expect(all).not.toContain('claude-sonnet')
  expect(all).not.toContain('🧭 메인')
  expect(all).toContain('╭')
  expect(all).toContain('╯')
  await ui.unmount()
})

test('대기 중에는 작업 흐름 없이 정지한 기본 테두리만 쓴다', async ($, on) => {
  stubEngine(on)

  await $.session.start({ cwd: '/', surface: 'terminal', isInteractive: true })

  const ui = await $.ui.mount(BAND(false))
  const all = (await ui.findAll({ type: 'Text' })).map(t => t.text).join('|')

  expect(all).not.toContain('🧭 메인')
  expect(all).toContain('Context')
  await ui.unmount()
})

test('자동 압축 기준이 설정되어 있으면 Context 옆에 보여 준다', async ($, on) => {
  stubEngine(on, { pct: 42, frame: 0, compact: '50' })
  await $.session.start({ cwd: '/', surface: 'terminal', isInteractive: true })

  const ui = await $.ui.mount(BAND(false))
  const all = (await ui.findAll({ type: 'Text' })).map(t => t.text).join('|')

  expect(all).toContain('50%에서 압축')
  await ui.unmount()
})

for (const bad of ['abc', '0', '100', '', undefined]) {
  test(`자동 압축 기준이 ${JSON.stringify(bad)}이면(1~99 정수가 아니면) 보여 주지 않는다`, async ($, on) => {
    stubEngine(on, { pct: 42, frame: 0, compact: bad })
    await $.session.start({ cwd: '/', surface: 'terminal', isInteractive: true })

    const ui = await $.ui.mount(BAND(false))
    const all = (await ui.findAll({ type: 'Text' })).map(t => t.text).join('|')

    expect(all).toContain('Context')
    expect(all).not.toContain('에서 압축')
    await ui.unmount()
  })
}

// 압축 기준을 설정했으면 기준 10%p 전부터, 설정이 없으면 90%부터 "컴팩트 임박"을 알린다.
for (const c of [
  { compact: '50', pct: 39, show: false },
  { compact: '50', pct: 40, show: true },
  { compact: undefined, pct: 89, show: false },
  { compact: undefined, pct: 90, show: true },
  // 기준 − 10이 0 이하가 되는 낮은 기준에서도 경고 시작점은 1%로 묶인다.
  { compact: '5', pct: 0, show: false },
  { compact: '5', pct: 1, show: true },
]) {
  test(`컴팩트 임박 경고: 압축 기준 ${c.compact ?? '없음'}, Context ${c.pct}% → ${c.show ? '표시' : '숨김'}`, async ($, on) => {
    stubEngine(on, { pct: c.pct, frame: 0, compact: c.compact })
    await $.session.start({ cwd: '/', surface: 'terminal', isInteractive: true })

    const ui = await $.ui.mount(BAND(false))
    const all = (await ui.findAll({ type: 'Text' })).map(t => t.text).join('|')

    if (c.show) expect(all).toContain('컴팩트 임박')
    else expect(all).not.toContain('컴팩트 임박')
    await ui.unmount()
  })
}

test('맨 아래 줄의 CPU·RAM 막대는 5칸이다', async ($, on) => {
  stubEngine(on, { pct: 42, frame: 0 })
  await $.session.start({ cwd: '/', surface: 'terminal', isInteractive: true })

  const ui = await $.ui.mount(BAND(false))
  const foot = (await ui.findAll({ type: 'Text' })).map(t => t.text).find(t => t.startsWith('💻 CPU'))!
  const barOf = (label: string) => foot.slice(foot.indexOf(label) + label.length).split(' ')[0]!

  expect([...barOf('💻 CPU ')].length).toBe(5)
  expect([...barOf('💾 RAM ')].length).toBe(5)
  await ui.unmount()
})

test('게이지는 한 줄에 하나씩, 라벨 칸이 맞고 CPU·RAM·비용·시간은 맨 아래 한 줄에 모인다', async ($, on) => {
  stubEngine(on, { pct: 42, frame: 0 })
  await $.session.start({ cwd: '/', surface: 'terminal', isInteractive: true })

  const ui = await $.ui.mount(BAND(false))
  const texts = (await ui.findAll({ type: 'Text' })).map(t => t.text)

  // 세 게이지 줄은 라벨 시작부터 막대까지 같은 칸 수다(한글은 2칸으로 센다).
  const barStart = (prefix: string) => {
    const t = texts.find(x => x.startsWith(prefix))!
    const label = [...t.slice(t.indexOf(' ') + 1)]
    const i = label.findIndex(ch => ch === '▌' || ch === '░')
    return label.slice(0, i).reduce((n, ch) => n + (ch.charCodeAt(0) > 0x2e7f ? 2 : 1), 0)
  }
  const starts = ['🧠 Context', '⏰ 5h 한도', '📅 7d 한도'].map(barStart)
  expect(new Set(starts).size).toBe(1)

  const foot = texts.find(t => t.startsWith('💻 CPU'))!
  expect(foot).toContain('💾 RAM')
  expect(foot).toContain('$1.234')
  expect(foot).toContain('⏳')
  await ui.unmount()
})

test('모든 막대는 어떤 백분율에서도 8칸이다', async ($, on) => {
  const opts = { pct: 0, frame: 0 }
  stubEngine(on, opts)
  await $.session.start({ cwd: '/', surface: 'terminal', isInteractive: true })

  for (const pct of [0, 1, 12, 49, 50, 99, 100]) {
    opts.pct = pct
    await $.session.measure?.({} as any).catch?.(() => {})
    const ui = await $.ui.mount(BAND(false))
    const ctx = (await ui.findAll({ type: 'Text' })).map(t => t.text).find(t => t.startsWith('🧠 Context '))!
    const bar = ctx.slice('🧠 Context '.length).split(' ')[0]!
    expect([...bar].length).toBe(8)
    await ui.unmount()
  }
})

// 처리 중 밴드를 주어진 프레임으로 그려, 테두리 각 변의 칸별 빨강 채널 값을 읽는다.
// 기본 파랑은 0x5b, 빛줄기가 보라에 가까울수록 값이 커진다.
const BASE_RED = 0x5b
async function borderCells($: any, on: any, frames: number[], width = 140) {
  const opts = { pct: 42, frame: 0 }
  stubEngine(on, opts)
  await $.session.start({ cwd: '/', surface: 'terminal', isInteractive: true })
  await $.turn.start({ text: '시작', turnId: 't1' } as any)
  await $.turn.complete({ answer: '', text: '', reason: 'end_turn' } as any)

  const red = (c: unknown) => parseInt(String(c).slice(1, 3), 16)
  const out: { top: number[]; bottom: number[]; right: number[]; left: number[] }[] = []
  for (const frame of frames) {
    opts.frame = frame
    const ui = await $.ui.mount(BAND(true, width))
    // 변 전체를 감싸는 바깥 Text는 색이 없으므로 빼고, 색이 있는 조각만 읽는다.
    const edges = ((await ui.findAll({ type: 'Text', text: /^[╭╮╰╯─]+$/ })) as any[]).filter(e => e.props?.color)
    const cells: number[] = []
    for (const e of edges) for (const _ of e.text) cells.push(red(e.props?.color))
    const bars = ((await ui.findAll({ type: 'Text', text: /^│$/ })) as any[]).map(e => red(e.props?.color))
    // 위쪽 선 조각들 다음에 아래쪽 선 조각들이 이어서 나오고, 왼쪽 세로선 다음에 오른쪽 세로선이 나온다.
    out.push({
      top: cells.slice(0, width),
      bottom: cells.slice(width, width * 2),
      left: bars.slice(0, bars.length / 2),
      right: bars.slice(bars.length / 2),
    })
    await ui.unmount()
  }
  return out
}

test('테두리 색은 파랑과 보라 사이에서 부드럽게 이어진다', async ($, on) => {
  const [f0] = await borderCells($, on, [0])
  // 6칸 조각이 한 색이므로, 조각 경계에서 이웃 칸의 빨강 채널 차이만 본다.
  const diffs = f0!.top.slice(1).map((r, i) => Math.abs(r - f0!.top[i]!))
  expect(Math.max(...diffs)).toBeLessThan(10)
  for (const r of [...f0!.top, ...f0!.bottom, ...f0!.left, ...f0!.right]) {
    expect(r).toBeGreaterThanOrEqual(BASE_RED)
    expect(r).toBeLessThanOrEqual(0xc6)
  }
})

test('색은 위쪽은 오른쪽으로, 오른쪽은 아래로, 아래쪽은 왼쪽으로 흐른다', async ($, on) => {
  // 폭 140, 줄 7개: 둘레 294칸을 30프레임에 한 바퀴. 보라 정점은 한 프레임에 9.8칸 나아간다.
  const fr = await borderCells($, on, [0, 2, 4, 13, 16, 24, 26])
  const head = (xs: number[]) => xs.indexOf(Math.max(...xs))

  // 위쪽: 정점이 왼쪽 위 모서리에서 시작해 오른쪽(큰 칸)으로 간다.
  expect(head(fr[0]!.top)).toBeLessThan(head(fr[1]!.top))
  expect(head(fr[1]!.top)).toBeLessThan(head(fr[2]!.top))

  // 오른쪽: 정점이 세로선 위쪽을 지나기 전에는 위쪽이, 지난 뒤에는 아래쪽이 더 보랏빛이다.
  const before = fr[3]!.right
  const after = fr[4]!.right
  expect(before[0]!).toBeGreaterThan(before[before.length - 1]!)
  expect(after[after.length - 1]!).toBeGreaterThan(after[0]!)

  // 아래쪽: 정점이 오른쪽(큰 칸)에서 왼쪽(작은 칸)으로 간다.
  expect(head(fr[6]!.bottom)).toBeLessThan(head(fr[5]!.bottom))
})

test('응답이 끝나도 유지 시간 동안은 도는 테두리가 남는다', async ($, on) => {
  // 엔진은 이미 처리 중이 아니라고(isWorking=false) 알린다.
  stubEngine(on)
  await $.session.start({ cwd: '/', surface: 'terminal', isInteractive: true })
  await $.turn.start({ text: '시작', turnId: 't1' } as any)
  await $.turn.complete({ answer: '', text: '', reason: 'end_turn' } as any)

  const ui = await $.ui.mount(BAND(false))
  const texts = (await ui.findAll({ type: 'Text' })).map(e => e.text).join('|')
  expect(texts).not.toContain('🧭 메인')
  expect(texts).toContain('╭')
  const edges = ((await ui.findAll({ type: 'Text', text: /^[╭╮╰╯─]+$/ })) as any[]).filter(e => e.props?.color)
  expect(edges.some(e => String(e.props.color) !== '#5b9bf0')).toBe(true)
  await ui.unmount()
})

test('가득 찬 막대의 색은 반 칸 단위로 부드럽게 이어진다', async ($, on) => {
  const opts = { pct: 100, frame: 0 }
  stubEngine(on, opts)
  await $.session.start({ cwd: '/', surface: 'terminal', isInteractive: true })

  const ui = await $.ui.mount(BAND(false))
  // 화면 순서상 첫 막대가 Context이고 8칸이 모두 채워져 있다.
  const cells = ((await ui.findAll({ type: 'Text', text: /^▌$/ })) as any[]).slice(0, 8)
  expect(cells.length).toBe(8)

  const rgb = (c: string) => [1, 3, 5].map(i => parseInt(c.slice(i, i + 2), 16))
  const samples = cells.flatMap(c => [rgb(String(c.props.color)), rgb(String(c.props.backgroundColor))])
  let worst = 0
  for (let i = 1; i < samples.length; i++) {
    for (let ch = 0; ch < 3; ch++) worst = Math.max(worst, Math.abs(samples[i]![ch]! - samples[i - 1]![ch]!))
  }
  // 한 칸을 한 색으로 칠하던 때는 이웃 단계의 차이가 50을 넘었다.
  expect(worst).toBeLessThan(30)
  await ui.unmount()
})

// 내부 모델 이름이 화면용 이름으로 바뀌어 나온다.
for (const [raw, label] of [
  ['claude-sonnet-5-5', 'Sonnet 5.5'],
  ['claude-fable-5-2', 'Fable 5.2'],
  ['claude-opus-5-5', 'Opus 5.5'],
  ['claude-haiku-5-5', 'Haiku 5.5'],
  ['claude-opus-5', 'Opus 5'],
  ['claude-sonnet-4-5-20250929', 'Sonnet 4.5'],
  ['claude-sonnet-5-5[1m]', 'Sonnet 5.5'],
  ['claude-3-5-sonnet', 'Sonnet 3.5'],
  ['opus', 'Opus'],
  ['sonnet[1m]', 'Sonnet'],
  // 목록에 없는 새 계열도 그대로 읽는다.
  ['claude-nova-6-1', 'Nova 6.1'],
  ['claude-zephyr-7', 'Zephyr 7'],
  ['us.anthropic.claude-sonnet-5-5-v1:0', 'Sonnet 5.5'],
  // Claude 이름이 아니면 건드리지 않는다.
  ['gpt-5', 'gpt-5'],
] as const) {
  test(`모델 이름 ${raw} → ${label}`, async ($, on) => {
    stubEngine(on, { pct: 42, frame: 0, model: raw })
    await $.session.start({ cwd: '/', surface: 'terminal', isInteractive: true })

    const ui = await $.ui.mount(BAND(false))
    const names = (await ui.findAll({ type: 'Text' })).map(e => e.text)
    expect(names).toContain(label)
    await ui.unmount()
  })
}

test('새 계열의 이모지는 로봇이 아니고, 같은 이름이면 늘 같다', async ($, on) => {
  const emojiOf = async (model: string) => {
    stubEngine(on, { pct: 42, frame: 0, model })
    await $.session.start({ cwd: '/', surface: 'terminal', isInteractive: true })
    const ui = await $.ui.mount(BAND(false))
    const names = (await ui.findAll({ type: 'Text' })).map(e => e.text)
    await ui.unmount()
    return names.find(n => n.endsWith(' ') && n.length <= 4 && n !== ' ')
  }
  const a = await emojiOf('claude-nova-6-1')
  expect(a).toBeDefined()
  expect(a).not.toContain('🤖')
  expect(a).not.toContain('🎻')
})

// 첫 줄의 git 표시: 브랜치 뒤에 변경 파일 수(*), 올릴 커밋(↑), 받을 커밋(↓)이 0이 아닐 때만 붙는다.
for (const c of [
  { name: '깨끗하고 서버와 같음', git: '## main...origin/main\n', has: ['🌿 main'], not: ['*', '↑', '↓'] },
  {
    name: '변경 3개, 올릴 커밋 1개',
    git: '## main...origin/main [ahead 1]\n M a.ts\n?? b.ts\n M c.ts\n',
    has: ['🌿 main', '*3', '↑1'],
    not: ['↓'],
  },
  { name: '올릴 1개, 받을 2개', git: '## main...origin/main [ahead 1, behind 2]\n', has: ['↑1', '↓2'], not: ['*'] },
  { name: '서버 브랜치 없음', git: '## feat\n', has: ['🌿 feat'], not: ['*', '↑', '↓'] },
]) {
  test(`git 표시: ${c.name}`, async ($, on) => {
    stubEngine(on, { pct: 42, frame: 0, git: c.git })
    await $.session.start({ cwd: '/', surface: 'terminal', isInteractive: true })

    const ui = await $.ui.mount(BAND(false))
    const all = (await ui.findAll({ type: 'Text' })).map(t => t.text).join('|')

    for (const s of c.has) expect(all).toContain(s)
    for (const s of c.not) expect(all).not.toContain(s)
    await ui.unmount()
  })
}

// 5시간 한도는 3시간 뒤 초기화(경과율 약 40%), 7일 한도는 4일 뒤 초기화(경과율 약 43%)로 고정되어 있다.
// 사용률이 경과율보다 20%p 이상 앞서면 "빠름"을 알린다.
for (const c of [
  { five: 12, seven: 30, show: false },
  { five: 55, seven: 30, show: false },
  { five: 65, seven: 30, show: true },
  { five: 12, seven: 70, show: true },
]) {
  test(`한도 페이스 경고: 5h ${c.five}%, 7d ${c.seven}% → ${c.show ? '표시' : '숨김'}`, async ($, on) => {
    stubEngine(on, { pct: 42, frame: 0, five: c.five, seven: c.seven })
    await $.session.start({ cwd: '/', surface: 'terminal', isInteractive: true })

    const ui = await $.ui.mount(BAND(false))
    const all = (await ui.findAll({ type: 'Text' })).map(t => t.text).join('|')

    if (c.show) expect(all).toContain('빠름')
    else expect(all).not.toContain('빠름')
    await ui.unmount()
  })
}
````

## 단계 2. 설정 파일 수정

단계 0-3에서 "설정 안 함"을 골랐다면 이 단계에서 할 일은 없으니 건너뛰고 단계 3으로 가세요.

"설정함"을 골랐다면 `<SETTINGS_FILE>`을 먼저 읽고(프로젝트 범위에서 파일이나 `.claude` 폴더가 없으면 새로 만들어 주세요), 아래 항목을 `env` 블록에 **추가**해 주세요. 이미 같은 키가 있으면 사용자가 고른 `<N>`으로 바꾸세요. `env` 블록이 없으면 새로 만들고, 다른 설정은 절대 건드리지 마세요.

```json
"env": {
  "CLAUDE_AUTOCOMPACT_PCT_OVERRIDE": "<N>"
}
```

- `CLAUDE_AUTOCOMPACT_PCT_OVERRIDE`: 컨텍스트가 약 `<N>`% 찼을 때 자동 압축이 실행되게 합니다. 값은 따옴표로 감싼 문자열로 적고, 낮추는 방향으로만 동작합니다.
- mod 폴더는 `skills` 아래에 있어 자동으로 인식되므로 `CLAUDE_CODE_PLUGIN_DIRS`는 필요 없습니다. 이미 이 값에 같은 폴더가 들어 있으면 중복 로드될 수 있으니 저에게 알려 주세요.

`statusLine` 항목은 단계 0-2에서 사용자가 삭제를 고른 것만 처리하고, 그 외에는 건드리지 마세요.

수정한 뒤 파일이 올바른 JSON인지 확인해 주세요.

## 단계 3. 검증

1. `claude plugin validate <MOD_DIR>`를 실행해 주세요. 타입 오류(`error TS`)나 `skipped`가 없어야 합니다. README와 LICENSE가 없다는 항목 2건은 공유용 제출 규칙이라 무시해도 됩니다. 이 2건은 출력에 "refuses a submission"이라는 문장과 함께 나오지만 같은 이유이므로 함께 무시하세요.
2. `claude plugin test <MOD_DIR>`를 실행해 주세요. 테스트 44건이 모두 통과해야 합니다.
3. 실패가 있으면 코드를 임의로 고치지 말고 출력 전체를 보여 주세요.

## 단계 4. 현재 세션에 바로 적용

Claude는 슬래시 명령을 직접 실행할 수 없으므로, 아래처럼 안내해 주세요.

1. 선택한 범위(전역 또는 이 프로젝트만)와 만든 파일 위치를 한 줄로 알려 주세요.
2. 사용자에게 지금 이 대화창에 `/reload-plugins`를 입력하라고 안내해 주세요. 재시작 없이 현재 세션에 mod가 로드됩니다.
3. 프로젝트 범위에서 이 폴더를 신뢰하는지 묻는 확인이 뜨면 수락하라고 알려 주세요.
4. 입력창 위에 파란 테두리의 상태 밴드가 나오는지 확인하라고 안내해 주세요. 나오지 않으면 `/plugin`의 Errors 탭이나 `/doctor`를 확인하고, 그 내용을 저에게 보여 달라고 해 주세요.
5. 자동 압축을 설정했다면(단계 0-3에서 "설정함"), 그 설정(`CLAUDE_AUTOCOMPACT_PCT_OVERRIDE`)은 세션에 바로 반영되지 않을 수 있으니, 반영이 안 되면 Claude Code를 다시 시작하고 `/context`로 기준 값이 `<N>`%에 맞는지 확인하라고 안내해 주세요. "설정 안 함"이었다면 이 항목은 말하지 마세요.

## 알아 둘 점

- **모델 이름**: 이름에서 계열과 버전을 직접 읽으므로(`claude-sonnet-5-5`는 Sonnet 5.5) 새 모델이나 새 계열이 나와도 코드를 고칠 필요가 없습니다. 이모지는 opus, sonnet, haiku, fable만 고정이고 새 계열은 이름에서 정해진 이모지를 씁니다. Claude 이름이 아닌 모델은 원래 이름 그대로 표시합니다.
- **CPU·RAM 줄**: Windows PowerShell(`Get-CimInstance`)로 측정하므로 Windows에서만 나옵니다. macOS와 Linux에서는 맨 아래 줄에서 CPU·RAM 조각만 빠지고, 비용과 경과 시간은 그대로 나옵니다.
- **계정 이메일 표시**: 첫 줄에 로그인한 계정의 이메일이 그대로 보입니다. 화면을 공유하거나 캡처할 때 주의해 주세요.
- **밴드 폭**: 터미널 폭이 약 110칸보다 좁으면 줄이 넘칠 수 있습니다.
- **구독이 아닌 세션**: 5시간·7일 한도 정보가 없으면 두 막대는 나오지 않고 Context만 표시됩니다.
- **git 표시**: `git status --porcelain -b`로 읽습니다. 추적되지 않는 새 파일도 변경 파일 수(`*`)에 들어가지만, git은 추적되지 않는 폴더를 한 줄로 묶어 보여 주므로 폴더 하나는 안에 파일이 여럿이어도 1로 셉니다. git 저장소가 아니면 브랜치와 함께 모두 빠지고, 서버(upstream) 브랜치가 없으면 `↑`·`↓`는 나오지 않습니다. 분리 HEAD 상태에서는 브랜치 이름이 없어 변경 파일 수(`*`)만 폴더 이름 뒤에 나옵니다. 아주 큰 저장소에서 `git status`가 3초 제한을 넘기면 브랜치 이름을 포함해 이 표시가 모두 사라집니다.
- **한도 페이스 경고**: 5시간·7일 한도에서 "기간이 지난 비율"보다 사용률이 20%p 이상 앞서면 `⚠ 빠름`이 붙습니다(예: 5시간 중 2시간이 지났는데 60% 사용). 초기화 시각 정보가 없으면 표시하지 않습니다.
- **낮은 압축 기준**: "⚠ 컴팩트 임박" 경고는 압축 기준보다 10%p 앞에서 뜹니다. 기준을 11% 이하로 두면 경고 시작점이 1%로 묶여서 Context 1%부터 경고가 계속 보입니다.
