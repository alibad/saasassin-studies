# SaaSassin Studies

**Free, open-source Maze alternative.** Run rigorous product studies with your own participants and portable response data.

Product research testing for product teams and researchers testing prototypes, concepts, and information architecture.

- **Try it:** [studies.saasassin.net](https://studies.saasassin.net)
- **Research brief and product decision:** [saasassin.net/targets/maze](https://saasassin.net/targets/maze)

[![Deploy with Vercel](https://vercel.com/button)](https://vercel.com/new/clone?repository-url=https%3A%2F%2Fgithub.com%2Falibad%2Fsaasassin-studies&project-name=saasassin-studies&repository-name=saasassin-studies&env=NEXT_PUBLIC_FIREBASE_API_KEY%2CNEXT_PUBLIC_FIREBASE_AUTH_DOMAIN%2CNEXT_PUBLIC_FIREBASE_PROJECT_ID%2CNEXT_PUBLIC_FIREBASE_APP_ID%2CFIREBASE_PROJECT_ID%2CFIREBASE_CLIENT_EMAIL%2CFIREBASE_PRIVATE_KEY&envDescription=Firebase+web+app+config+and+service-account+credentials+%28see+the+README%29&envLink=https%3A%2F%2Fgithub.com%2Falibad%2Fsaasassin-studies%231-create-a-firebase-project)

Maze is a trademark of its owner and is named here only to describe what this product
replaces. SaaSassin Studies is an independent, clean-room product and is not affiliated
with or endorsed by Maze.

## What you can do today

| Who | Can now… |
|-----|----------|
| Researcher | Create a workspace, invite teammates by email as admin / researcher / viewer |
| Researcher | Build a study from blocks: welcome & consent, multiple choice (single/multi), opinion scale 1–5 or 1–10, open text, yes/no, website task, closed card sort, **tree test** |
| Researcher | Add skip-to logic and screener answers that end the study; preview the draft with nothing recorded |
| Researcher | Publish (freezes an immutable version), close, reopen; cap participants and set a close date; duplicate |
| Participant | Open `/apps/studies/p/{tenantId}/{studyId}`, consent and answer anonymously, resumable within the tab |
| Participant | **Navigate a text-only tree one task at a time** (open items, back up, "I'd find it here" or skip) |
| Researcher | See results per block, completion funnel, card-sort agreement matrix |
| Researcher | **See tree-test results per task: success rate, directness, median time, outcome split (direct/indirect success/failure, skip), first-click and chosen-destination distributions** |
| Researcher | **See every session with automatic quality flags and their reasons: speeders (study minimum, or a third of the median once 5 sessions completed), straight-lining (same rating on 3+ scales), empty or junk open text** |
| Researcher | **Exclude a session with a required reason, or keep a flagged one; both are audited. Results, block CSVs and the shared link leave excluded sessions out; a toggle shows everything** |
| Researcher | **View a session's raw answers and cite any answer straight into a finding** |
| Researcher | **Record findings with title, summary, severity, status and at least one piece of evidence (a specific session's answer or a whole block); the server resolves every citation and stores an excerpt; evidence from later-excluded sessions is called out** |
| Anyone with the share link | **Read findings with their evidence above the results** (no raw sessions on the link) |
| Researcher | **Import a Maze results CSV: preview exactly how each column maps (block type, timing, metadata, dropped), then import it as a closed, read-only study they can analyse, exclude, cite and export — or duplicate to re-run in Studies** |
| Owner/admin | **Add HTTPS webhook endpoints for `response.completed` and `study.closed` (manual close or cap reached); secret shown once; HMAC-SHA256 signed; enable/disable, send a test, delete; delivery log with honest failures** |
| Any member | View the audit log; download the study responses JSON (now with quality flags, decisions and findings), block CSVs, and the complete workspace export (now with findings, webhooks with secrets redacted, and delivery log) |

Full status, verification and known gaps: [docs/STATUS.md](docs/STATUS.md).

## Self-host

SaaSassin Studies is a Next.js app backed by Firebase Authentication and Cloud Firestore.
You need Node.js 20.9 or newer, [pnpm](https://pnpm.io/installation) and a Firebase project of
your own.

### 1. Create a Firebase project

1. Create a project in the [Firebase console](https://console.firebase.google.com/).
2. **Firestore:** Build → Firestore Database → Create database.
3. **Authentication:** Build → Authentication → Get started. Studies signs people in with
   **Google** and **Email/Password**; step 2 below enables both, or enable them yourself under
   Authentication → Sign-in method.
4. **Web app config:** Project settings → General → Your apps → add a Web app. Copy its config
   into the `NEXT_PUBLIC_FIREBASE_*` variables.
5. **Service account:** Project settings → Service accounts → Generate new private key. From the
   downloaded JSON, set `FIREBASE_PROJECT_ID` (`project_id`), `FIREBASE_CLIENT_EMAIL`
   (`client_email`) and `FIREBASE_PRIVATE_KEY` (`private_key`). Never commit the key, and never
   give these variables a `NEXT_PUBLIC_` prefix.
6. Once you deploy, add your domain under Authentication → Settings → Authorized domains
   (`localhost` is already allowed).

### 2. Deploy the rules, indexes and sign-in providers

```bash
pnpm install
pnpm exec firebase login
pnpm exec firebase deploy --only firestore:rules,firestore:indexes,auth --project <your-project-id>
```

`firestore.rules` denies all browser access to product data: every read and write goes through the server routes, which check the session, tenant membership and role with the Firebase Admin SDK. `auth` enables the Google and email/password providers listed in `firebase.json`.

### 3. Run it

```bash
cp .env.example .env.local   # fill in the values from step 1
pnpm dev
```

Open [http://localhost:3000](http://localhost:3000). This repository serves SaaSassin
Studies at the root of every host: `next.config.ts` sets `STANDALONE_PRODUCT=studies`,
and `src/proxy.ts` maps `/` to the product (`/apps/studies`) and `/sign-in` to its sign-in page.

### 4. Deploy to Vercel

[![Deploy with Vercel](https://vercel.com/button)](https://vercel.com/new/clone?repository-url=https%3A%2F%2Fgithub.com%2Falibad%2Fsaasassin-studies&project-name=saasassin-studies&repository-name=saasassin-studies&env=NEXT_PUBLIC_FIREBASE_API_KEY%2CNEXT_PUBLIC_FIREBASE_AUTH_DOMAIN%2CNEXT_PUBLIC_FIREBASE_PROJECT_ID%2CNEXT_PUBLIC_FIREBASE_APP_ID%2CFIREBASE_PROJECT_ID%2CFIREBASE_CLIENT_EMAIL%2CFIREBASE_PRIVATE_KEY&envDescription=Firebase+web+app+config+and+service-account+credentials+%28see+the+README%29&envLink=https%3A%2F%2Fgithub.com%2Falibad%2Fsaasassin-studies%231-create-a-firebase-project)

The button copies this repository into your GitHub account, creates a Vercel project and asks for
the Firebase variables. To deploy an existing fork instead, import it in Vercel and add the
variables from `.env.example` under Project Settings → Environment Variables. Either way, add the
Vercel domain to your Firebase authorized domains afterwards.

## Tests

```bash
pnpm lint
pnpm typecheck
pnpm test        # unit tests
pnpm build
```

The Firestore suites run against the local Firestore emulator, which needs a Java 21+ JDK first on
your `PATH`:

```bash
export JAVA_HOME=/path/to/jdk-21   # macOS example: /Library/Java/JavaVirtualMachines/temurin-21.jdk/Contents/Home
export PATH="$JAVA_HOME/bin:$PATH"
pnpm test:rules      # Firestore security rules
pnpm test:emulator   # the golden workflow against a real Firestore emulator
```

## Contributing

Development happens in the SaaSassin monorepo,
[github.com/alibad/saasassin](https://github.com/alibad/saasassin), the single source of truth for
all ten SaaSassin products. This repository is generated from it by the monorepo's
`scripts/export-product.mjs`, so accepted changes land in the monorepo first and are then synced
here. Open issues and pull requests there or here.

Report vulnerabilities privately as described in [SECURITY.md](SECURITY.md).

## Talk to us

Moving off Maze, running Studies for your team, or want to help build it?
[Tell us on saasassin.net](https://saasassin.net/targets/maze#interest).

## License

[GNU Affero General Public License v3.0](LICENSE) (AGPL-3.0-only). Use it, self-host it and modify
it; if you run a modified version as a network service, publish your changes under the same
license.
