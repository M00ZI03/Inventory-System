# Converting from a cloud setup (Vercel + Supabase) to a local system

## Why I moved it

The first version ran in the cloud: the frontend was deployed on Vercel and the data lived in Supabase. That worked, but it had drawbacks for this project:

- **Ongoing dependence on third-party services** and their free-tier limits
- **Data stored outside the organization's control**, with no simple way to back it up locally
- **Internet required** every time the app was used
- **More moving parts** than a small team needed for one inventory list

A single local machine could do the whole job, so I moved everything onto it.

## What changed

| | Before | After |
|---|---|---|
| Frontend hosting | Vercel (build and CDN) | Vite build served by Nginx on the local PC |
| Backend and database | Supabase (hosted Postgres, auth, API) | PocketBase (one binary, SQLite) on the same PC |
| Access | Public URL | Local network address, plus NetBird for remote access |
| Config | `vercel.json`, hosted environment variables | Local `.env` and an Nginx config |
| Backups | Provider dashboard | Copy the `pb_data` folder |

## Code changes

1. **Swap the client library.** Remove `@supabase/supabase-js` and add the PocketBase SDK:

```bash
   npm uninstall @supabase/supabase-js
   npm install pocketbase
```

2. **Create one shared client:**

```js
   import PocketBase from 'pocketbase';
   export const pb = new PocketBase(import.meta.env.VITE_POCKETBASE_URL || '/');
```

3. **Map each query.** Typical conversions:

   | Supabase | PocketBase |
   |---|---|
   | `supabase.from('items').select('*')` | `pb.collection('items').getFullList()` |
   | `.insert({...})` | `pb.collection('items').create({...})` |
   | `.update({...}).eq('id', id)` | `pb.collection('items').update(id, {...})` |
   | `.delete().eq('id', id)` | `pb.collection('items').delete(id)` |
   | `.eq('category', x)` | `getFullList({ filter: 'category = "x"' })` |

4. **Auth.** Replace Supabase sign-in with `pb.collection('users').authWithPassword(email, password)`.

5. **Remove Vercel files** such as `vercel.json` and any Vercel-specific build settings.

## Moving the data

1. Export each table from Supabase as CSV.
2. Create matching collections in the PocketBase admin dashboard (same field names and types).
3. Import the CSV files using the dashboard's import option.
4. Clean as you go. In my case I removed stale duplicate rows and gave variants that shared a product code their own unique codes before importing.

## Keeping the backend private

PocketBase is bound to `127.0.0.1`, so it cannot be reached from other devices directly. Nginx serves the app and forwards `/api/` to PocketBase on the same machine. For use away from home or the office, a NetBird tunnel lets approved devices reach the Nginx address securely, without opening any ports on the router.

## Things to watch on low-memory hardware

- **Build the frontend on another computer** and copy the `dist/` folder to the server. Running `npm run build` on a 3 GB machine can be slow or run out of memory.
- PocketBase itself is light and runs comfortably alongside Nginx.
