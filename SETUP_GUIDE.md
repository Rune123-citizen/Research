# RewardIQ — Person 1 Frontend Setup Guide

## 1. WSL Terminal Commands (run in order)

```bash
cd ~
npx create-next-app@latest rewardiq-p1 --typescript --tailwind --eslint --app --src-dir --import-alias "@/*"
cd rewardiq-p1
npm install lucide-react clsx
code .
```

When create-next-app asks questions:
- TypeScript → Yes
- ESLint → Yes
- Tailwind → Yes
- src/ directory → Yes
- App Router → Yes
- Import alias → Yes (keep @/*)

## 2. Replace files with the ones provided in this folder

Copy each file from this folder into your project at the same path:

| File in this folder                          | Goes into your project at               |
|----------------------------------------------|-----------------------------------------|
| tailwind.config.ts                           | tailwind.config.ts                      |
| src/styles/globals.css                       | src/styles/globals.css                  |
| src/app/layout.tsx                           | src/app/layout.tsx                      |
| src/app/page.tsx                             | src/app/page.tsx                        |
| src/app/auth/page.tsx                        | src/app/auth/page.tsx                   |
| src/app/onboarding/page.tsx                  | src/app/onboarding/page.tsx             |
| src/app/dashboard/page.tsx                   | src/app/dashboard/page.tsx              |
| src/data/mockData.ts                         | src/data/mockData.ts                    |
| src/components/layout/Navbar.tsx             | src/components/layout/Navbar.tsx        |
| src/components/layout/Footer.tsx             | src/components/layout/Footer.tsx        |
| src/components/sections/HeroSection.tsx      | src/components/sections/HeroSection.tsx |
| src/components/sections/TrustStrip.tsx       | src/components/sections/TrustStrip.tsx  |
| src/components/sections/HowItWorks.tsx       | src/components/sections/HowItWorks.tsx  |
| src/components/sections/FeaturedCards.tsx    | src/components/sections/FeaturedCards.tsx|
| src/components/sections/HiddenTerms.tsx      | src/components/sections/HiddenTerms.tsx |
| src/components/sections/BottomCTA.tsx        | src/components/sections/BottomCTA.tsx   |

## 3. Start dev server

```bash
npm run dev
```

Open http://localhost:3000

## 4. Pages & Routes

| URL                    | What you see                          |
|------------------------|---------------------------------------|
| /                      | Landing page (full)                   |
| /auth                  | Sign up / Login (all 3 states)        |
| /onboarding            | 7-step quiz                           |
| /dashboard             | Placeholder (Person 3 builds this)    |

## 5. Auth page states

The auth page has 3 states you can test:
1. Email entry → type email → Send OTP
2. OTP entry → type any 6 digits → Verify code
3. Success → auto-redirects to /onboarding

## 6. Folder structure

```
src/
  app/
    layout.tsx          ← root layout, fonts, metadata
    page.tsx            ← landing page
    auth/page.tsx       ← login/signup (3 states)
    onboarding/page.tsx ← 7-step quiz
    dashboard/page.tsx  ← placeholder
  components/
    layout/
      Navbar.tsx        ← sticky nav with scroll effect
      Footer.tsx        ← 4-column footer
    sections/
      HeroSection.tsx   ← mesh gradient hero
      TrustStrip.tsx    ← 4-stat trust bar
      HowItWorks.tsx    ← 3-step section
      FeaturedCards.tsx ← 4 card tiles
      HiddenTerms.tsx   ← 3 hidden term examples
      BottomCTA.tsx     ← final CTA before footer
  data/
    mockData.ts         ← all mock data for all sections
  styles/
    globals.css         ← design tokens, component classes
```
