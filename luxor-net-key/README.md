# Luxor Net Key

**A secret picture is your encryption key.**
Invented by **Christian Rivera (DJ BrightFuture)**. Open source under the MIT License.

A hard password can be a picture you captured that only you know is *the one* — and it's
even stronger when that picture is a **random pencil drawing or piece of art you made**,
because a unique hand-drawn scribble is a high-entropy secret that exists nowhere else in
the world.

---

## How it works (zero-knowledge)

1. A random 256-bit **Data Encryption Key (DEK)** encrypts your data with **AES-256-GCM**.
2. The DEK is stored only as a **wrap**: the DEK encrypted under a key derived from the
   **exact bytes of your secret picture** (**PBKDF2**, 250,000 iterations, SHA-256).
3. To unlock, you provide the same picture → the same key is derived → the DEK unwraps →
   your data decrypts. A wrong picture fails the AES-GCM authentication tag and simply
   won't open.
4. **The picture itself is never stored or transmitted.** Only the wrap
   (`salt + iv + ciphertext`) is. A server — or a thief — who holds the wrap learns nothing
   about your picture and cannot open your data.

This is the same pattern that lets you add a **second** key (another picture, or a
passphrase for recovery) *without ever re-encrypting your data* — you just add another wrap
of the same DEK.

## Install / use

No dependencies. Uses the Web Crypto API — works in browsers, Node 18+, Cloudflare Workers, Deno.

```js
import { lockWithPicture, unlockWithPicture, readFileBytes } from "./luxor-net-key.js";

// In a browser, from <input type="file"> or a camera capture:
const pictureBytes = await readFileBytes(fileInput.files[0]);

const envelope = await lockWithPicture("my secret text", pictureBytes);
// -> store `envelope` anywhere; it is pure ciphertext.

const text = await unlockWithPicture(envelope, samePictureBytes); // throws if the picture is wrong
```

Low-level API is also exported: `pictureCredential`, `wrapSecret`, `unwrapSecret`, `addKey`.

Try it live: open **[`demo.html`](./demo.html)** — it runs entirely in your browser, nothing is uploaded.

## Honest security notes

A security tool that hides its limits is lying. Here are Luxor Net Key's, plainly:

- **It is EXACT-BYTES.** The key is the exact file, not "what the picture looks like."
  Photograph your random drawing **once**; that file is your key. A fresh re-photograph
  produces different bytes and will **not** unlock. This is deliberate and tested — it is
  exactly what makes a stolen re-photograph useless. Keep the one file safe and secret,
  like a physical key.
- **A copy of your picture file = a copy of your key.** Anyone who obtains the file can
  unlock your data, like anyone holding a physical key. Its strength is that it is *secret*
  and *unique*. A public image (your face, a famous photo) is a weak choice; a private,
  one-of-a-kind drawing is a strong one.
- **Entropy comes from the picture.** A real photo or hand drawing carries far more entropy
  than a typed password. A tiny, plain, or widely-shared image is weak.
- **Client-side crypto.** Run it from a page you trust. The whole point is that the key
  material never leaves the device.
- **Watch out for re-encoding on phones.** Some phones and share sheets re-encode an image
  (e.g. HEIC → JPEG) when you *select* or *share* it, which changes its bytes — so the "same
  photo" from your camera roll can fail to unlock. Use a **saved key file you control** and
  select that same file each time. If a picture that should work won't open your data, it was
  probably re-encoded; keep a stable, untouched copy of your key file (back it up somewhere
  safe, like any key).

## Proven

The cryptography ships with a test that verifies: the same picture unlocks; a different
picture is rejected; a 1-byte-different re-scan is rejected (exact-bytes property); the
store holds no picture bytes; a second key can be added without re-encrypting; two random
drawings never collide; and derivation is deterministic. **9/9 pass.**

## License

MIT © 2026 Christian Rivera. Invented by Christian Rivera (DJ BrightFuture),
Luxor Net Technologies.
