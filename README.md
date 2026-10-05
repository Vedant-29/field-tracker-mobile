# Field Tracker Mobile

The employee app for Field Tracker, a small field-workforce tool. A field employee signs in, sees the tasks assigned to them for a given day on a map, opens each task's location in Google Maps, marks tasks done, and sends their current GPS position so an admin can see where they are.

The admin console is [field-tracker-web](https://github.com/Vedant-29/field-tracker-web). Both apps share the same Supabase project.

Built with Expo SDK 51, expo-router, React Native 0.74 and Supabase.

## Features

- Email sign up and sign in through Supabase Auth. Sign up also creates the employee's `employee_users` row.
- Home tab with a map and a bottom sheet listing tasks, filtered by date and by status (to complete or completed).
- Tap a task to move the map to it, open its map link, or change its status.
- "Send Location" writes the phone's current coordinates to the employee's row, which the admin console shows on its map.
- Profile tab showing the employee's name, email, phone and bio.

## Requirements

- Node 18+
- A Supabase project with the `employee_users` and `employee_tasks` tables (see below) and RLS enabled
- A Google Maps API key for Android builds (used by `react-native-maps`)
- Xcode for iOS builds, Android Studio for Android builds, or Expo Go on a phone

## Setup

```sh
git clone https://github.com/Vedant-29/field-tracker-mobile.git
cd field-tracker-mobile
npm install
cp .env.example .env
npx expo start
```

Scan the QR code with Expo Go, or press `i` or `a` to open the iOS simulator or Android emulator.

The app asks for foreground location permission at runtime. `app.json` also declares `ACCESS_BACKGROUND_LOCATION` on Android. Remove it if you do not need background location.

## Environment variables

| Variable | Required | What it is for | Where to get it |
|---|---|---|---|
| `EXPO_PUBLIC_SUPABASE_URL` | Yes | Supabase project URL | Supabase dashboard > Project Settings > API |
| `EXPO_PUBLIC_SUPABASE_ANON_KEY` | Yes | Supabase anon key | Same page |
| `GOOGLE_MAPS_ANDROID_API_KEY` | For Android builds | Google Maps SDK key | Google Cloud console > APIs & Services > Credentials |

`EXPO_PUBLIC_*` values are bundled into the app, so anyone with the app can read them. Rely on RLS so each employee can only read and write their own rows.

## Database

| Table | Used for |
|---|---|
| `employee_users` | One row per employee, keyed by `employee_id` (the auth user id). Holds `name`, `email`, `phoneNo`, `bio`, `latitude`, `longitude`. |
| `employee_tasks` | Tasks assigned to an employee (`assigned_to_id`), with `status`, `completion_date`, location fields, contact details, `location_map_link` and `description`. |

## Scripts

| Command | What it does |
|---|---|
| `npm start` | Start the Expo dev server |
| `npm run ios` / `npm run android` / `npm run web` | Start and open on a platform |
| `npm test` | Run Jest in watch mode |
| `npm run lint` | Run `expo lint` |

## Notes

- `app.json` has an empty `googleMapsApiKey`. It is not filled from `.env` automatically, so paste your key there (or move the config to `app.config.js`) before an Android build.
- Restrict the Google Maps key by package name and SHA-1 (and iOS bundle id) before release.
- `app-example/` is the original Expo template, kept for `npm run reset-project`.

## License

No license file. All rights reserved.
