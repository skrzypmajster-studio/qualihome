# webpage-template

Template for static site applications built on Next.js stack

## Tech Stack

| Area            | Technology                                                                                                    |
| --------------- | ------------------------------------------------------------------------------------------------------------- |
| Framework       | [Next.js](https://nextjs.org/docs) 16 (App Router) with static export (`output: 'export'`)                    |
| UI library      | [React](https://react.dev) 19                                                                                 |
| Language        | [TypeScript](https://www.typescriptlang.org) 5.9 in strict mode                                               |
| Styling         | [Tailwind CSS](https://tailwindcss.com) 4 (via `@tailwindcss/turbopack`), global styles in `app/globals.css`  |
| Fonts           | [Geist](https://vercel.com/font) loaded with `next/font/google`                                               |
| Linting         | [ESLint](https://eslint.org) 9 with `eslint-config-next` (Core Web Vitals and TypeScript rules)               |
| Formatting      | [Prettier](https://prettier.io) 3 with `eslint-config-prettier` (see `.prettierrc.json`)                      |
| Git hooks       | [Husky](https://typicode.github.io/husky/) 9 and [lint-staged](https://github.com/lint-staged/lint-staged) 16 |
| Package manager | npm (`package-lock.json`)                                                                                     |
| Node.js         | 24 for local development (see [.nvmrc](.nvmrc)); CI workflows currently use Node.js 22                        |
| CI/CD           | GitHub Actions                                                                                                |
| Hosting         | Netlify (staging) and OVHcloud over SFTP (production)                                                         |

### Project structure

```text
app/                    App Router: root layout, home page, global CSS, favicon
public/                 Static assets served as-is
.github/workflows/      Staging and production deployment pipelines
next.config.ts          Next.js config (static export into dist/)
```

## Getting Started

### Prerequisites

- Node.js 24 (run `nvm use` to pick up the version from `.nvmrc`)
- npm

Install dependencies:

```bash
npm ci
```

First, run the development server:

```bash
npm run dev
# or
yarn dev
# or
pnpm dev
# or
bun dev
```

Open [http://localhost:3000](http://localhost:3000) with your browser to see the result.

You can start editing the page by modifying `app/page.tsx`. The page auto-updates as you edit the file.

This project uses [`next/font`](https://nextjs.org/docs/app/building-your-application/optimizing/fonts) to automatically optimize and load [Geist](https://vercel.com/font), a new font family for Vercel.

## Available Scripts

Scripts are defined in [package.json](package.json). Run them with `npm run <script>`.

| Script         | Runs                 | Description                                                                                                                                                             |
| -------------- | -------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `dev`          | `next dev`           | Starts the development server with hot reload at [http://localhost:3000](http://localhost:3000)                                                                         |
| `build`        | `next build`         | Creates a production build. The static site is exported to `dist/`                                                                                                      |
| `start`        | `next start`         | Not usable with this project. Next.js does not support `next start` with `output: 'export'`. To preview the build, serve the `dist/` folder with any static file server |
| `lint`         | `eslint`             | Runs ESLint on the project                                                                                                                                              |
| `format`       | `prettier --write .` | Formats all files with Prettier                                                                                                                                         |
| `format:check` | `prettier --check .` | Checks formatting without changing files. Fails if any file needs formatting                                                                                            |
| `prepare`      | `husky`              | Runs automatically after `npm install` or `npm ci`. Installs the Git hooks from `.husky/`                                                                               |     |

There is no test script yet.

### Build output

`next.config.ts` sets `output: 'export'`, `distDir: 'dist'`, and `trailingSlash: true`. Each `npm run build` writes a fully static site to `dist/`, starting with `dist/index.html`. The staging and production workflows check for this file before deploying.

## Code Quality

Linting and formatting are enforced locally before each commit.

- **ESLint** (`eslint.config.mjs`) checks the code with the Next.js, Core Web Vitals, and TypeScript rule sets. `eslint-config-prettier` turns off rules that conflict with Prettier. The `dist/` build output is ignored.
- **Prettier** (`.prettierrc.json`, `.prettierignore`) handles formatting. Run `npm run format` to format the project, or `npm run format:check` to verify it without changing files.
- **Pre-commit hook** (`.husky/pre-commit`) runs `npx lint-staged`, which processes only the staged files:
  - JavaScript and TypeScript files (`*.{js,mjs,cjs,jsx,ts,tsx}`): `eslint --fix`, then `prettier --write`.
  - JSON, CSS, Markdown, and YAML files (`*.{json,css,md,yml,yaml}`): `prettier --write`.

The commit is blocked if ESLint reports an error. Warnings do not block it. Auto-fixed changes are re-staged automatically.

The hook is installed by the `prepare` script during `npm install`. If the hook does not run after cloning, run `npm run prepare`.

## Learn More

To learn more about Next.js, take a look at the following resources:

- [Next.js Documentation](https://nextjs.org/docs) - learn about Next.js features and API.
- [Learn Next.js](https://nextjs.org/learn) - an interactive Next.js tutorial.

You can check out [the Next.js GitHub repository](https://github.com/vercel/next.js) - your feedback and contributions are welcome!

## Deployment

Deployments are run manually from GitHub Actions. Neither workflow runs on push. To deploy, open the **Actions** tab, select the workflow, and click **Run workflow**. Both workflows build the site with `npm ci` and `npm run build`, check that `dist/index.html` exists, and then deploy the `dist/` folder.

| Workflow              | File                                                                     | GitHub environment | Source branch                             | Target                          |
| --------------------- | ------------------------------------------------------------------------ | ------------------ | ----------------------------------------- | ------------------------------- |
| Staging deployment    | [staging-deployment.yml](.github/workflows/staging-deployment.yml)       | `staging`          | Branch selected when running the workflow | Netlify site `<repo-name>-test` |
| Production deployment | [production-deployment.yml](.github/workflows/production-deployment.yml) | `production`       | `main` (fixed)                            | OVHcloud over SFTP              |

### Staging (Netlify)

1. The workflow looks up a Netlify site named `<repo-name>-test` (lowercase). It reuses the site if exactly one matches, creates it if none exists, and fails if more than one matches.
2. It deploys `dist/` with `netlify-cli` using `--prod`.
3. The site is available at `https://<repo-name>-test.netlify.app`.

### Production (OVHcloud over SFTP)

1. The workflow checks out `main`, builds the site, and uploads the contents of `dist/` to `SFTP_SERVER_DIR` on the server.
2. Uploads use port 22 and `delete: false`, so files already on the server are not removed.

### Required environment configuration

Create the `staging` and `production` environments in **Settings → Environments** and add the values below to each one. The workflows read them through the `secrets` and `vars` contexts.

| Environment  | Name                 | Type     | Required | Description                                                                                                                          |
| ------------ | -------------------- | -------- | -------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| `staging`    | `NETLIFY_AUTH_TOKEN` | Secret   | Yes      | Netlify auth token used to look up, create, and deploy the staging site. The job fails if it is missing.                             |
| `staging`    | `NETLIFY_TEAM_SLUG`  | Variable | No       | Slug of the Netlify team that owns a newly created staging site. If it is not set, the site is created in the token owner's account. |
| `production` | `SFTP_SERVER`        | Secret   | Yes      | Hostname of the OVHcloud SFTP server.                                                                                                |
| `production` | `SFTP_USERNAME`      | Secret   | Yes      | SFTP username.                                                                                                                       |
| `production` | `SFTP_PASSWORD`      | Secret   | Yes      | SFTP password.                                                                                                                       |
| `production` | `SFTP_SERVER_DIR`    | Secret   | Yes      | Remote directory that receives the contents of `dist/`, typically the web root.                                                      |

Do not commit these values to the repository.
