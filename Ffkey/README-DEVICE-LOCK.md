# NXL Admin Free Key — 24-hour key record update

## Website changes
The existing `userinfo/{KEY}` record is retained. New records include `password`, `status=active`, `access=1`, `device=null`, `deviceId=null`, `deviceLocked=false`, `activatedAt=null`, `createdAt` as a Firebase server timestamp, and `expiryHours=24`. The key-list display shows status, device binding state, and creation/expiry time where available.

## Required APK-side contract
1. Generate a stable app-scoped installation identifier; do not use IMEI or restricted hardware identifiers. Store it in app-private storage / Android Keystore-backed storage.
2. Read `userinfo/{KEY}` and reject missing keys or any status other than `active`.
3. Use Firebase Realtime Database server time (`.info/serverTimeOffset`) and reject when `serverNow >= createdAt + expiryHours * 3_600_000`.
4. On first activation, atomically bind using a Realtime Database transaction: if `deviceId` is empty, write the current installation ID, set `deviceLocked=true`, and set `activatedAt` to server time; if it matches, allow; if it differs, reject.
5. Re-check status, expiry, and deviceId on every login / protected operation.

Do not implement first-device binding with an ordinary REST PUT/PATCH. Use an atomic Firebase transaction or trusted backend to prevent two devices racing to claim one key.

## Security limitation
This is a static GitHub Pages website. It cannot securely enforce login inside a separate APK by itself; the APK must perform the checks above. Firebase Rules must prevent unauthorised clients from changing `createdAt`, `expiryHours`, `status`, or another device's `deviceId`. For strong protection, use a trusted backend / Cloud Function with admin authentication for key creation and privileged actions. Never put a Firebase service-account private key in this website. The supplied ZIP did not include Firebase security rules, so they have not been audited.
