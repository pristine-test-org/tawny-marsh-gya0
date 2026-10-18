# Tawny Marsh: Monorepo · Vercel

A monorepo with three apps on **Vercel**, with preview links **public**. Static Vite +
TypeScript, no framework. The page reads a live list from the shared Supabase test project.
Row in the support matrix: impeccable-reviews#302.

## Deployment links

| App | Folder | Production | Preview gate |
|---|---|---|---|
| violet-beg | `apps/violet-beg` | https://tawny-marsh-gya0-violet-beg.vercel.app | no |
| willow-et3 | `apps/willow-et3` | https://tawny-marsh-gya0-willow-et3.vercel.app | no |
| flax-20b | `apps/flax-20b` | https://tawny-marsh-gya0-flax-20b.vercel.app | no |

## Names

Every prepared repository gets a random name of its own, never reused, for example
`amber-falcon-7k2q`. In a monorepo, each of the three apps also gets a random name (folder, package
name and page title). App names must not repeat across repositories: when `todo-monorepo-2` reused
`todo-monorepo`'s app names, the dashboard opened the old repository's apps.

## Prepare it for an eval

1. Copy (don't move) this folder to `tmp/eval-repos/<name>/`, `<name>` being the new random name.
2. Rename the repository folder, the root `package.json` name, and each app: its folder under `apps/`, its `package.json` name, and its title in `index.html`. Replace `app-one`, `app-two`, `app-three` and `App One`, `App Two`, `App Three` everywhere. Run `npm install` so `package-lock.json` has the new names.
3. All three apps are renamed and `npm run build` passes for each, then `git init -b main`, commit, and push to a new repository
   `pristine-test-org/<name>` (`gh repo create`; push with the token on the URL). Then add a plain `origin`,
   `git remote add origin https://github.com/pristine-test-org/<name>.git`, with no token in it: the
   command quits without one.
4. For each app, create a Git-connected Vercel project named `<name>-<app>` in the `rg-test` team:
   `vercel api "/v11/projects?teamId=team_iuofJm6Q925kcDS0oDMTk9pb" --method POST --input body.json --scope rg-test`,
   body: `name`, `"framework": "vite"`, `"rootDirectory": "apps/<app>"`, `"gitRepository": {"type": "github", "repo": "pristine-test-org/<name>"}`.
5. Right after, PATCH `/v9/projects/<id>?teamId=team_iuofJm6Q925kcDS0oDMTk9pb` with
   `gitProviderOptions={"createDeployments":"enabled"}`. Projects made through the API start with it
   disabled, and the command finds previews on GitHub's deployments.
6. Turn Vercel Authentication **off** (new projects start with Standard Protection): PATCH `/v9/projects/<id>?teamId=team_iuofJm6Q925kcDS0oDMTk9pb` with `"ssoProtection": null`. Check a preview URL opens in a private window without a Vercel login.
7. Push a commit that changes a file in every app's folder (an empty commit builds nothing: the
   host skips projects a commit doesn't touch). That starts
   the first production builds; creating a project deploys nothing.
8. Write the production URLs into **Deployment links** above, delete the template
   notice at the top, and push that to `main`.

## Supabase

Every page reads the latest five rows of the `todos` table in the shared test project
(`https://hszqtfynyogshhltamep.supabase.co`), the same one the todo monorepos use. The URL and publishable key are
defaults in `src/supabase.ts`, so a host needs no environment variables.

**When a new deployment or preview URL comes up**, check Supabase lets it in:

- **Data (CORS):** nothing to add. Supabase's Data API answers any origin; on 2026-10-19 a
  request from a made-up preview origin came back allowed. Open the deployment and check the
  "Live from the shared list" strip shows rows, not "Couldn't reach the list".
- **Auth URLs:** the only per-URL setting Supabase has is Auth → URL Configuration (Site URL and
  Redirect URLs). It matters only for an app that signs people in through Supabase Auth with a
  redirect. These pages don't, so skip it. If an app ever does, add its preview URL pattern to
  Redirect URLs with the Supabase CLI (`supabase link --project-ref hszqtfynyogshhltamep`, then
  `supabase config push`). Careful: `config push` writes the shared project's whole auth config.

## Run the eval

A repo eval, not `npm run eval`: the steps are in `REPO-EVAL.md` beside the templates. In short:
open the repo's app on impeccable.reviews, wait for the scan (Rescan if apps are missing), click
Setup, run the command it gives in `tmp/eval-repos/<name>/`, and wait for Impeccable's review on
the test PR.
