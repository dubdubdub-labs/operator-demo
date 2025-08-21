# IMPLEMENTATION — Shared Infrastructure for UI Experiments

A lightweight implementation plan focused on rapid experimentation with shared mocks and contracts.

---

## Core Philosophy
- **Mock Everything**: No real APIs, databases, or external services
- **Share Liberally**: Common components and data across all experiments
- **Stream Fake**: All "AI responses" are pre-written strings with simulated streaming
- **Focus on Feel**: Optimize for interaction quality, not functionality

---

## Shared Data Contract (Zustand Store)

```typescript
// stores/experimentStore.ts
interface ExperimentStore {
  // User Context
  user: {
    email: string
    linkedinUrl?: string
    role?: string
    company?: string
    preferences: {
      uiStyle: 'notion' | 'linear' | 'airtable' | null
      complexity: 'minimal' | 'balanced' | 'power'
      aiAssistance: 'aggressive' | 'balanced' | 'minimal'
    }
  }

  // Survey/Discovery State
  discovery: {
    currentQuestion: number
    answers: Record<string, any>
    confidence: number // 0-100
    spec: AppSpec
    knowledgeGraph: Node[]
  }

  // App Specification (Living Document)
  spec: {
    id: string
    title: string
    purpose: string
    workflows: Workflow[]
    uiPatterns: string[]
    dataContract: DataSchema
    nonGoals: string[]
    openQuestions: string[]
    version: number
    history: SpecVersion[]
  }

  // Data Schema Contract
  schema: {
    tables: Table[]
    relationships: Relationship[]
    version: number
    migrations: Migration[]
  }

  // Generated Variants
  variants: {
    id: string
    name: string
    status: 'queued' | 'building' | 'ready' | 'error'
    preview?: string // base64 or URL
    logs: string[]
    agent: AgentPersonality
    features: string[]
    userRating?: 'love' | 'like' | 'dislike' | 'needs-work'
  }[]

  // Agent Orchestration
  agents: {
    id: string
    type: AgentPersonality
    status: 'idle' | 'thinking' | 'building' | 'complete'
    currentTask: string
    messages: AgentMessage[]
  }[]

  // Telemetry & Context
  telemetry: {
    actions: UserAction[]
    patterns: Pattern[]
    suggestions: Suggestion[]
    privacySettings: PrivacySettings
  }

  // Active Workspace
  workspace: {
    blocks: Block[]
    backgroundJobs: Job[]
    notifications: Notification[]
  }

  // Actions
  actions: {
    // Discovery
    answerQuestion: (answer: any) => void
    generateNextQuestion: () => void
    
    // Variants
    generateVariants: (count: number) => void
    rateVariant: (id: string, rating: Rating) => void
    synthesizeVariant: (features: string[]) => void
    
    // Spec/Schema
    updateSpec: (changes: Partial<AppSpec>) => void
    updateSchema: (changes: Partial<DataSchema>) => void
    
    // Workspace
    addBlock: (block: Block) => void
    runBackgroundJob: (job: Job) => void
    
    // Streaming
    streamAgentMessage: (agentId: string, chunk: string) => void
    streamBuildLog: (variantId: string, chunk: string) => void
  }
}
```

---

## File System Structure

```
demo/
├── SUMMARY.md                    # Experiment descriptions
├── IMPLEMENTATION.md             # This file
├── README.md                     # How to run experiments
│
├── shared/                       # Shared infrastructure
│   ├── stores/
│   │   └── experimentStore.ts    # Zustand store
│   │
│   ├── components/               # Reusable UI components
│   │   ├── StreamingText.tsx    # Simulates typing/streaming
│   │   ├── AgentBubble.tsx      # Agent status/thinking
│   │   ├── SpecEditor.tsx       # Spec document editor
│   │   ├── SchemaVisualizer.tsx # ER diagram component
│   │   ├── VariantCard.tsx      # Variant preview card
│   │   ├── ConfidenceMeter.tsx  # Visual confidence
│   │   ├── PrivacyToggle.tsx    # Privacy controls
│   │   ├── MetaAgentDock.tsx    # Bottom-right dock
│   │   └── WorkspaceBlock.tsx   # Notion-style blocks
│   │
│   ├── mocks/                    # Mock data & services
│   │   ├── fakeStream.ts        # Streaming text simulator
│   │   ├── sampleData.ts        # Companies, deals, etc.
│   │   ├── agentResponses.ts    # Pre-written agent dialogue
│   │   ├── variantPreviews.ts   # Base64 preview images
│   │   └── telemetryEvents.ts   # Fake user actions
│   │
│   ├── hooks/                    # Shared React hooks
│   │   ├── useStreaming.ts      # Simulate streaming
│   │   ├── useFakeDelay.ts      # Random delays
│   │   └── useKeyboardShortcuts.ts
│   │
│   └── styles/
│       ├── globals.css          # Shared styles
│       └── themes.ts            # UI theme variants
│
├── experiments/                  # Individual experiments
│   ├── 01-adaptive-discovery/
│   │   ├── page.tsx
│   │   └── components/
│   │       ├── DynamicSurvey.tsx
│   │       └── KnowledgeGraph.tsx
│   │
│   ├── 02-variant-factory/
│   │   ├── page.tsx
│   │   └── components/
│   │       ├── FactoryLanes.tsx
│   │       └── VMSwitcher.tsx
│   │
│   ├── 03-living-spec/
│   │   ├── page.tsx
│   │   └── components/
│   │       └── DualPane.tsx
│   │
│   ├── 04-context-lens/
│   │   ├── page.tsx
│   │   └── components/
│   │       └── TelemetryOverlay.tsx
│   │
│   ├── 05-variant-carousel/
│   │   ├── page.tsx
│   │   └── components/
│   │       └── SwipeInterface.tsx
│   │
│   ├── 06-ai-components/
│   │   ├── page.tsx
│   │   └── components/
│   │       ├── GhostText.tsx
│   │       ├── OracleTable.tsx
│   │       └── ProphetEditor.tsx
│   │
│   ├── 07-workspace-canvas/
│   │   ├── page.tsx
│   │   └── components/
│   │       └── BackgroundJobs.tsx
│   │
│   ├── 08-agent-timeline/
│   │   ├── page.tsx
│   │   └── components/
│   │       └── GanttView.tsx
│   │
│   ├── 09-progressive-disclosure/
│   │   ├── page.tsx
│   │   └── components/
│   │       └── ComplexitySlider.tsx
│   │
│   ├── 10-fork-share/
│   │   ├── page.tsx
│   │   └── components/
│   │       └── SharingModal.tsx
│   │
│   ├── 11-deal-memo-workspace/
│   │   ├── page.tsx
│   │   └── components/
│   │       └── DealMemoBlocks.tsx
│   │
│   └── 12-workflow-recording/
│       ├── page.tsx
│       └── components/
│           └── WorkflowVisualizer.tsx
│
└── app/                          # Next.js app directory
    ├── layout.tsx
    ├── page.tsx                  # Experiment selector
    └── experiments/
        └── [id]/
            └── page.tsx          # Dynamic experiment loader
```

---

## Shared Components

### Core UI Components

```typescript
// StreamingText: Simulates AI typing
<StreamingText 
  text="Here's what I found about Airbnb..." 
  speed={50} 
  onComplete={() => {}}
/>

// AgentBubble: Shows agent status
<AgentBubble
  agent={{ type: 'creative', status: 'thinking' }}
  message="Exploring UI patterns..."
/>

// VariantCard: Preview with status
<VariantCard
  variant={variant}
  onClick={() => {}}
  showLogs={false}
/>

// ConfidenceMeter: Visual progress
<ConfidenceMeter
  value={75}
  label="Context confidence"
/>
```

### Mock Services

```typescript
// fakeStream.ts
export async function* fakeStream(text: string, chunkSize = 5) {
  for (let i = 0; i < text.length; i += chunkSize) {
    await delay(random(30, 100))
    yield text.slice(i, i + chunkSize)
  }
}

// agentResponses.ts
export const agentResponses = {
  discovery: [
    "I see you're building a deal memo app. What stage deals do you typically evaluate?",
    "Do you prefer detailed financial models or high-level metrics?",
    "Would you like automated competitor analysis?"
  ],
  factory: [
    "Spinning up 5 variants with different approaches...",
    "Variant 1: Focusing on visual dashboards",
    "Variant 2: Optimizing for collaboration"
  ]
}

// sampleData.ts
export const companies = {
  airbnb: {
    name: "Airbnb",
    ticker: "ABNB",
    metrics: { revenue: "9.5B", growth: "18%" },
    description: "Global travel marketplace...",
    competitors: ["Booking", "Expedia", "Vrbo"]
  }
}
```

---

## Mock Data Patterns

### Streaming Simulation
```typescript
// All "AI" responses use pre-written content with fake delays
const streamResponse = async (text: string) => {
  for await (const chunk of fakeStream(text)) {
    store.streamAgentMessage(agentId, chunk)
  }
}
```

### Variant Generation
```typescript
// Pre-built variant screenshots as base64 or placeholder images
const variants = [
  { preview: "/mockups/variant-notion-style.png" },
  { preview: "/mockups/variant-airtable-style.png" },
  { preview: "/mockups/variant-linear-style.png" }
]
```

### Telemetry Events
```typescript
// Pre-recorded user action sequences
const mockSession = [
  { type: 'click', target: 'addBlock', timestamp: 0 },
  { type: 'type', value: 'Airbnb revenue', timestamp: 1000 },
  { type: 'scroll', delta: 200, timestamp: 2000 }
]
```

---

## Development Workflow

### Setup
```bash
# Install dependencies
npm install zustand framer-motion lucide-react cmdk

# Run all experiments
npm run dev

# Visit experiment selector
http://localhost:3000
```

### Creating New Experiment
1. Copy experiment template from `experiments/template/`
2. Import shared components and store
3. Mock any specific data needed
4. Focus on interaction and animation

### Shared Hooks
```typescript
// useStreaming: Handle fake streaming
const { text, isStreaming } = useStreaming(
  agentResponses.discovery[0], 
  { speed: 50 }
)

// useFakeDelay: Simulate loading
const { isLoading } = useFakeDelay(1500)

// useKeyboardShortcuts: Common shortcuts
useKeyboardShortcuts({
  'cmd+k': () => openCommandPalette(),
  'cmd+/': () => toggleAI()
})
```

---

## Animation & Polish

### Framer Motion Presets
```typescript
export const animations = {
  fadeIn: {
    initial: { opacity: 0 },
    animate: { opacity: 1 },
    transition: { duration: 0.3 }
  },
  slideUp: {
    initial: { y: 20, opacity: 0 },
    animate: { y: 0, opacity: 1 }
  },
  pulse: {
    animate: { 
      scale: [1, 1.05, 1],
      transition: { repeat: Infinity, duration: 2 }
    }
  }
}
```

### Loading States
```typescript
// Skeleton screens for all major components
<VariantCard.Skeleton />
<SpecEditor.Skeleton />
<WorkspaceBlock.Skeleton />
```

---

## Testing Strategy

### User Testing Setup
1. Each experiment has standalone URL
2. No login required
3. Reset button to clear state
4. Feedback widget in corner

### Metrics Collection
```typescript
// Simple analytics events (using PostHog or similar)
track('experiment_viewed', { id: '01-adaptive-discovery' })
track('variant_rated', { rating: 'love' })
track('feature_discovered', { feature: 'ghost-text' })
```

---

## Key Decisions

1. **No Backend**: Everything runs client-side with localStorage
2. **No Real AI**: All responses are pre-written with fake streaming
3. **No Auth**: Jump straight into experiments
4. **No Database**: Zustand + localStorage for persistence
5. **Static Assets**: Mock images and data checked into repo
6. **Fast Iteration**: Hot reload, no build step for experiments

---

Remember: We're optimizing for learning about the experience, not building production code. Keep it simple, make it feel magical.