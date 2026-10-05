# Field Tracker Mobile

The employee app for Field Tracker, a small field-workforce tool. A field employee signs in, sees the tasks assigned to them for a given day on a map, opens each task's location in Google Maps, updates task status, and sends their current GPS position so an admin can see where they are.

Related: [field-tracker-web](https://github.com/Vedant-29/field-tracker-web) (the admin console). Both apps share one Supabase project.

Built with Expo SDK 51, expo-router, React Native 0.74 and Supabase.

## Features

- Email sign up and sign in through Supabase Auth. Sign up also creates the employee's `employee_users` row.
- Home tab with a map and a bottom sheet listing tasks, filtered by date and by status (To complete, In Progress, Completed).
- Tap a task to move the map to it, open its map link, or change its status.
- "Send Location" writes the phone's current coordinates to the employee's row, which the admin console shows on its map.
- Profile tab showing the employee's name, email, phone and bio.

## Requirements

- Node 18+
- A Supabase project with the tables below and RLS enabled. Set it up from the [field-tracker-web Services section](https://github.com/Vedant-29/field-tracker-web#services) so both apps use the same project.
- A Google Maps API key for Android development and release builds (not needed in Expo Go)
- Xcode for iOS builds, Android Studio for Android builds, or Expo Go on a phone

## Setup

```sh
git clone https://github.com/Vedant-29/field-tracker-mobile.git
cd field-tracker-mobile
npm install
cp .env.example .env
npx expo start
```

Scan the QR code with Expo Go, or press `i` or `a` to open the iOS simulator or Android emulator. The app throws on start if the two Supabase variables are missing.

The app asks for foreground location permission at runtime. `app.json` also declares `ACCESS_BACKGROUND_LOCATION` on Android. Remove it if you do not need background location.

## Services

You need your own accounts and keys. Nothing is shipped with the repo.

| Service | Used for | Required | Config |
|---|---|---|---|
| Supabase | Auth, employee profile, tasks, last location | Yes | `EXPO_PUBLIC_SUPABASE_URL`, `EXPO_PUBLIC_SUPABASE_ANON_KEY` |
| Google Maps SDK for Android | Map on Android builds (`react-native-maps`) | Android builds only | `android.config.googleMapsApiKey` in `app.json` |

### Supabase

1. Use the same project as the web console. Copy the project URL and anon key from Project Settings > API into `.env`.
2. Sign up inserts the `employee_users` row right after `auth.signUp`. If Confirm email is on, there is no session yet, so either allow that insert in RLS or turn off Confirm email for testing.
3. Write RLS so an employee can only read and update their own `employee_users` row and the `employee_tasks` rows where `assigned_to_id` is their id.

### Google Maps

1. In Google Cloud console, enable Maps SDK for Android and create an API key under APIs & Services > Credentials.
2. Paste the key into `android.config.googleMapsApiKey` in `app.json`. It is empty in the repo and is not read from `.env`, so the `GOOGLE_MAPS_ANDROID_API_KEY` line in `.env.example` currently does nothing (or move the config to `app.config.js` and read it there).
3. Restrict the key by package name and SHA-1 (and iOS bundle id) before release.

iOS uses Apple Maps, because the map sets no provider, so it needs no key.

## Environment variables

| Variable | Required | What it is for | Where to get it |
|---|---|---|---|
| `EXPO_PUBLIC_SUPABASE_URL` | Yes | Supabase project URL | Supabase dashboard > Project Settings > API |
| `EXPO_PUBLIC_SUPABASE_ANON_KEY` | Yes | Supabase anon key | Same page |
| `GOOGLE_MAPS_ANDROID_API_KEY` | No | Listed in `.env.example` but not read by the app yet. See Google Maps above. | Google Cloud console > APIs & Services > Credentials |

`EXPO_PUBLIC_*` values are bundled into the app, so anyone with the app can read them. Rely on RLS so each employee can only read and write their own rows.

## Database

| Table | Used for |
|---|---|
| `employee_users` | One row per employee, keyed by `employee_id` (the auth user id). Holds `name`, `email`, `phoneNo`, `bio`, `latitude`, `longitude`. Sign up also writes a `password` column (see Notes). |
| `employee_tasks` | Tasks assigned to an employee (`assigned_to_id`), with `status`, `completion_date`, `created_at`, `location_name`, `location_poc_name`, `location_poc_email`, `location_poc_phoneNo`, `location_map_link`, `latitude`, `longitude` and `description`. |

`status` must be exactly `To complete`, `In Progress` or `Completed`. The home tab shows tasks whose `completion_date` falls on the selected day. Tasks are created by hand in Supabase, since neither app has a create form yet.

## Scripts

| Command | What it does |
|---|---|
| `npm start` | Start the Expo dev server |
| `npm run ios` / `npm run android` / `npm run web` | Start and open on a platform |
| `npm test` | Run Jest in watch mode |
| `npm run lint` | Run `expo lint` |
| `npm run reset-project` | Expo template script that moves `app/` to `app-example/` and creates a blank `app/`. Do not run it unless you mean to. |

## Notes

- Sign up stores the password in plain text in `employee_users.password`. Drop that field from the insert in `components/Auth.tsx` and the column before real use.
- The Sign Out button on the Profile tab has no handler yet. `components/Accounts.tsx` (an edit-profile form with a working sign out) is imported but not rendered.
- Phone numbers are shown with a hardcoded `+91` prefix.
- `app-example/` holds the original Expo template screens. The app does not use them.

## License

No license file. All rights reserved.
