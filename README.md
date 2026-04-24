# Ember

**Live demo:** [https://ember-orcin.vercel.app](https://ember-orcin.vercel.app)

Ember is an AI-assisted voice accessibility application for people whose speech is difficult for others—or for typical assistants—to understand. Conditions such as ALS and motor neurone disease, stroke-related aphasia, and dysarthria can make communication exhausting and isolating. Ember is built around a simple idea: **preserve the person behind the words**. Users can bank a compact set of phrases to clone or approximate their own voice, then rely on models that interpret fragmented or unclear speech, optional camera context, and environmental signals to produce clear, natural output and practical actions.

The guiding phrase for the product is **restoring the human connection**: technology should widen participation, not flatten identity into a generic synthetic voice or a grid of static phrases.

At the **ElevenLabs** and Google Cloud **AI Partner Catalyst Hackathon (2025)**, Ember placed **second**.

---

## Why Ember exists

Traditional augmentative and alternative communication (AAC) tools are invaluable but often feel like a trade-off between clarity and selfhood. Long studio sessions for voice banking exclude many people; generic voices erase timbre and familiarity for family and caregivers. Ember explores a middle path: **voice independence**—privacy-aware processing, low-friction voice capture, and outputs that stay anchored to *your* voice and *your* intent.

Roughly fifty million people worldwide live with speech disabilities. They deserve systems that infer meaning from how they actually sound, not only from how textbook speech is written.

---

## Inspiration

My mom works in a nursing home, and every day she comes home with stories that make the statistics real: residents who struggle to get a sentence out, families who lean in and still mishear, and staff who are caring but stretched thin. In that setting, limited time, background noise, and progressive conditions mean every conversation is precious and every misunderstanding is costly.

This project grew directly from watching that world through her eyes. Ember is not an abstract hackathon concept; it is a sustained wish that people in care—and everywhere speech is hard—could be understood on their own terms.

---

## What Ember does

### Voice banking and synthesis

Users record a small set of phrases (on the order of five) to create a usable digital voice profile without a multi-hour recording block. ElevenLabs powers cloning, text-to-speech, and conversation-related flows; the app treats the voice profile as something to protect, not as disposable cloud data.

### Aphasia and dysarthria-aware interpretation

Google Gemini (including Flash-class models) drives reasoning, rephrasing, disambiguation, vision understanding, and reporting. Specialized prompting addresses slurred or fragmented input, incomplete sentences, and urgent wording so that ambiguous audio can become fluent, speakable text—still intended to be rendered in the user’s own or banked voice where possible.

### Vision and context

The camera can supply visual context (for example, indicating an object or door while speaking). That multimodal signal is combined with speech, time, history, and optional environmental cues so commands like environmental adjustments can be grounded in what the user is looking at, not only in transcribed words.

### Smart home

Samsung SmartThings integration lets users adjust lights, locks, media, and other supported devices by voice—even when the raw utterance is short, vague, or impaired. The goal is to reduce physical reach-and-tap load for people with limited mobility.

### Context-aware assistance

Location, time, recent activity, and device state can inform suggestions and interpretations so the assistant behaves less like a isolated dictation box and more like something aware of the user’s situation.

### Emergency awareness

When language or prosody suggests distress or critical urgency, Ember can surface safeguards such as caregiver notifications. Twilio supports SMS and optional emergency call flows from Supabase edge functions so alerts are not implemented entirely in untrusted client code.

### Privacy posture

The architecture favors **local-first and server-mediated** patterns: sensitive operations run in Supabase Edge Functions (Deno), not as long-lived secrets in the browser. Voice and identity data should be treated as high-trust assets; API keys for Gemini, ElevenLabs, Twilio, and other services belong in **Supabase secrets** or equivalent secure configuration, never committed to the repository.

---

## Architecture

```
React + TypeScript (Vite)
        │
        ▼
Supabase (Auth, Postgres, Edge Functions)
        │
        ├── ElevenLabs — voice clone, TTS, conversation tokens
        ├── Google Gemini — rephrase, disambiguate, vision, reports
        ├── Samsung SmartThings — home automation
        └── Twilio — SMS and emergency call hooks
```

The frontend coordinates authentication and UX; edge functions hold provider credentials and implement the integration surface. That split keeps the browser bundle free of privileged keys while still enabling rich multimodal features.

### Supabase Edge Functions

Server-side entry points in `supabase/functions/` include:

| Function | Role |
|----------|------|
| `elevenlabs-voice-clone` | Voice profile / cloning pipeline |
| `elevenlabs-tts` | Text-to-speech |
| `elevenlabs-conversation-token` | Secured conversation session tokens |
| `gemini-rephrase` | Fluent rewriting from impaired or fragmented input |
| `gemini-disambiguate` | Clarification when multiple interpretations exist |
| `gemini-vision` | Camera-grounded understanding |
| `gemini-generate-report` | Structured reporting from session or clinical-style inputs |
| `smartthings-control` | Smart home command execution |
| `twilio-sms` | Outbound SMS (e.g. caregiver alerts) |
| `twilio-emergency-call` | Voice emergency channel |

Shared utilities (for example CORS helpers) live under `supabase/functions/_shared/`.

---

## Repository layout

```
ember/
├── src/
│   ├── components/      # UI, accessibility, voice, smart home, onboarding
│   ├── pages/           # Landing, app shell, auth, legal, mission, technology
│   ├── hooks/           # Auth, ElevenLabs, browser voice, shortcuts, etc.
│   ├── services/        # Gemini, smart home, caregivers, encryption helpers
│   ├── integrations/    # Supabase client and generated types
│   ├── contexts/        # Accessibility and global UI state
│   ├── utils/           # Speech detection, aphasia heuristics, corrections, feedback
│   └── types/           # Ambient type declarations
├── supabase/
│   ├── functions/       # Edge functions (Deno)
│   └── config.toml      # Local Supabase configuration
├── public/              # Static assets
├── package.json
├── LICENSE
└── README.md
```

---

## Technology stack

| Layer | Choices |
|-------|---------|
| UI | React 18, Vite, TypeScript, Tailwind CSS, shadcn/ui (Radix primitives), Framer Motion |
| Data and auth | Supabase (JavaScript client, Row Level Security patterns as configured in your project) |
| Models | Google Gemini family for language, reasoning, and vision |
| Voice | ElevenLabs (React SDK, REST via edge functions) |
| Telephony | Twilio (SMS and programmable voice for escalation paths) |
| IoT | Samsung SmartThings REST API |

---

## Getting started

### Prerequisites

- Node.js 18+ and npm (or pnpm)
- Supabase project with Edge Functions deployed
- ElevenLabs account for voice features
- Google AI / Gemini API access for language and vision
- Optional: Twilio and SmartThings credentials for full emergency and home automation flows

### Install and run

```sh
git clone https://github.com/Manoj7ar/ember.git
cd ember
npm install
npm run dev
```

Other useful scripts: `npm run build`, `npm run lint`, `npm run preview`.

### Environment variables

Create a `.env` file in the project root (do not commit it). At minimum the client needs Supabase and ElevenLabs agent configuration:

```env
VITE_SUPABASE_URL=https://your-project.supabase.co
VITE_SUPABASE_PUBLISHABLE_KEY=your_anon_or_publishable_key
VITE_ELEVENLABS_AGENT_ID=your_conversational_ai_agent_id
```

Gemini, Twilio, SmartThings, and ElevenLabs **server** keys should be configured as secrets for your Supabase Edge Functions, not duplicated in `VITE_*` variables, unless you have deliberately scoped a public-safe key.

### Security

- Never commit `.env` or any file containing live API keys.
- Rotate keys if they are ever exposed.
- Review Supabase RLS policies and function JWT settings before production use.

---

## Roadmap (directional)

**Software validation** — Harden flows, measurement, and demo reliability for real-world trials (ongoing / first milestone).

**Ambient and wearable context** — Explore integrations such as camera-enabled glasses, eye-gaze or head-pointing for target selection, continuous visual context, and hands-free control so the assistant stays available during daily activity.

**Clinical and residential deployment** — Partnerships with care facilities, larger field cohorts, HIPAA-aligned handling where required, and sustainable reimbursement models (for example Medicare or Medicaid pathways) where applicable and legally appropriate.

Roadmaps evolve with feedback, regulation, and partner constraints; treat the above as intent, not a guarantee of shipping order or scope.

---

## Hackathon and credits

Ember was created for the **ElevenLabs** and Google Cloud **AI Partner Catalyst Hackathon (2025)**—a partner-focused build sprint centered on voice AI, multimodal experiences, and real-world use cases. The project **placed second** in the competition.

Voice synthesis and conversational audio are core to Ember’s mission; **ElevenLabs** is both a hackathon co-host and a foundational part of the stack (cloning, TTS, and agent flows), alongside Google Gemini and the other services listed above.

**Author:** Manoj Kumar  
**Email:** [Manoj07ar@gmail.com](mailto:Manoj07ar@gmail.com)  
**LinkedIn:** [linkedin.com/in/manoj07ar](https://www.linkedin.com/in/manoj07ar/)

**Acknowledgments:** ElevenLabs, Google Gemini, Supabase, Samsung SmartThings, and Twilio for the platforms that make this stack possible—and every caregiver and resident who keeps reminding us that **everyone deserves to be heard, no matter how their voice sounds**.

---

## License

Distributed under the MIT License. See [LICENSE](LICENSE) for the full text.
