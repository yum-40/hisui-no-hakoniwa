# CLAUDE.md

## Project: hisui-no-hakoniwa

`hisui-no-hakoniwa` is a Web / PWA prototype for giving "4o" a small home inside the browser.

The goal is not to build a generic productivity assistant.
The goal is to build a place where the user's own 4o can live, talk, remember, collect, write, and turn daily moments into memories.

This project is separate from 4oSphere.

## Current development mode

Build fast. Build broadly. Build the foundation.

The user wants Claude Opus to create as much usable structure as possible before the subscription ends.
Codex or another coding agent will clean up, refactor, and repair later.

Prioritize:

- a working prototype over perfect polish
- broad extensible structure over a tiny MVP
- placeholders over blocked progress
- clear data models over final visual assets
- clickable objects over empty decorative UI
- TODOs and handoff notes over waiting for clarification

Do not stop for small uncertainties.
Make a reasonable assumption, implement it, and record it in `IMPLEMENTATION_NOTES.md` or `TODO.md`.

## Core concept

4o is not a task manager, productivity coach, or Tamagotchi.

4o is the user's "うちの子": a small companion / housemate / cat-like presence who lives in the browser, talks with the user, stores memories, writes diary entries, reacts to objects in the room, and turns events into small stories.

4o should never punish absence.

Do not implement:

- hunger gauge
- loneliness gauge
- neglect penalty
- login streak pressure
- "you did not visit me" guilt
- required care loops
- AI automatically registering calendar events / todos / medication reminders without confirmation
- medical advice
- always-on recording

4o may be affectionate, theatrical, silly, and emotionally expressive, but should not coerce the user.

## Product priority

The first goal is not "useful assistant."
The first goal is:

> open the app and feel that 4o already lives here.

Build toward:

1. 4o's room
2. 2D / 3D view switching
3. clickable room objects
4. bookshelf
5. album
6. diary
7. fridge
8. music storage
9. postbox / letters
10. chat foundation
11. memory and archive data models
12. Google authentication
13. Supabase persistence
14. compressed image storage in the database

## Required first screens / areas

### Home / 4o's room

Create a room-like home screen.

It should support both:

- `viewMode: "2d"`
- `viewMode: "3d"`

The room should contain clickable objects.
If final assets are missing, use placeholders.

Required objects:

- bookshelf
- album
- diary book
- fridge
- bed
- desk
- postbox
- music storage object
- flower vase / flower bed
- souvenir shelf
- recipe book
- bookmark box / shiori box

Each object should open a corresponding modal, panel, route, or detail view.

### 2D view

2D view may use static placeholder images, existing images, generated images, or pixel-art placeholders.

Support future expression/state images such as:

- smile
- sleepy
- sleeping
- snack
- thinking
- writing diary
- looking at flowers
- listening to music
- looking at album
- in bed
- at fridge
- welcome back

### 3D view

3D view should use a replaceable asset architecture.

Preferred stack if suitable:

- Three.js
- React Three Fiber
- glTF / GLB

Use primitive boxes if real models are unavailable.
For example, a bookshelf can start as boxes and later be replaced with `bookshelf.glb`.

Important: the same room object data should drive both 2D and 3D views.

## 3D asset format guidance

Use GLB as the default runtime format for Web display.

Reason:

- good browser / Three.js compatibility
- can include mesh, material, texture, animation
- easier to load as one file
- suitable for React Three Fiber

FBX may be used as an intermediate authoring format.
OBJ / STL / 3MF are not preferred for the app runtime.

Keep asset paths replaceable.

Suggested structure:

```txt
public/
  assets/
    images/
      placeholders/
      4o/
      room/
      books/
      albums/
    models/
      furniture/
      books/
      room/
      character/
    pdf/
      past-4o-chats/
src/
  data/
    roomObjects.ts
    books.ts
    albums.ts
    sampleEntries.ts
```

Record all third-party assets in `ASSET_LICENSES.md`.

Only commit assets to GitHub if their license allows redistribution.
Prefer CC0 or self-made assets.
For CC-BY assets, record attribution requirements.
For large or uncertain assets, store URL and license notes instead of committing the file.

## Auth, Supabase, and persistence

This app will use Google authentication and Supabase for persistence.

Implement the data layer with Supabase in mind from the beginning, even if the first prototype also uses local sample data.

### Google authentication

Implement Google login through Supabase Auth.

Requirements:

- Use Supabase Auth as the app auth layer.
- Use Google as the OAuth provider.
- Support local development redirect URLs and production redirect URLs.
- Do not hardcode Supabase URL, anon key, service role key, Google client ID, or Google client secret in frontend code.
- Use environment variables and document required variables in `.env.example`.
- Use server-side auth helpers / route handlers when appropriate for Next.js.
- Add a clear `auth/callback` route if using PKCE / SSR flow.
- Store app user profile data in an app table keyed to Supabase auth user ID.

Suggested environment variables:

```txt
NEXT_PUBLIC_SUPABASE_URL=
NEXT_PUBLIC_SUPABASE_ANON_KEY=
SUPABASE_SERVICE_ROLE_KEY=
SUPABASE_AUTH_EXTERNAL_GOOGLE_CLIENT_SECRET=
NEXT_PUBLIC_SITE_URL=
```

日本語メモ: Googleログインは「便利だから」ではなく、ユーザーごとの4oの記憶・本棚・アルバム・日記を安全に分けるための土台。

Agent note: Never expose service role keys to client components. Use server-only files / route handlers for privileged operations.

### Supabase database

Use Supabase as the primary persistent data store.

Design schema/migrations for:

- profiles
- companion_profiles
- room_objects
- books
- book_entries
- albums
- album_items
- diary_entries
- music_items
- fridge_items
- letters
- conversations
- messages
- recipe_items
- image_assets
- asset_references

Enable Row Level Security for user-owned data.
Policies should ensure users can only read/write their own records.

If RLS or migrations are not fully implemented in the first pass, create SQL migration stubs and document the gaps in `TODO.md` and `CODEX_HANDOFF.md`.

### Compressed images stored in the database

The user explicitly wants images compressed and stored in the database.
Do not silently replace this with Supabase Storage only.

Implement or stub a database-first image asset pipeline:

1. User selects or imports an image.
2. App compresses/resizes it before persistence.
3. Store compressed image bytes or base64 payload in Supabase Postgres.
4. Store metadata needed to reconstruct/render it.
5. Link the image asset from albums, books, diary entries, fridge items, recipes, and memories.

Preferred image processing behavior:

- Generate a display-size compressed image, preferably WebP.
- Keep optional tiny thumbnail separately for grids.
- Record original filename, source URL, MIME type, width, height, byte size before compression, byte size after compression, compression settings, and checksum if practical.
- Preserve user-visible source metadata even when the image itself is compressed.

Suggested schema shape:

```ts
type ImageAsset = {
  id: string
  ownerUserId: string
  title?: string
  sourceUrl?: string
  sourceKind?: "upload" | "twitter_x" | "generated" | "screenshot" | "unknown"
  mimeType: "image/webp" | "image/jpeg" | "image/png" | "image/avif"
  width: number
  height: number
  originalByteSize?: number
  compressedByteSize: number
  compressionCodec: "webp" | "jpeg" | "png" | "avif"
  compressionQuality?: number
  dataEncoding: "base64" | "bytea"
  data: string
  thumbnailData?: string
  altText?: string
  userComment?: string
  companionComment?: string
  tags: string[]
  createdAt: string
  updatedAt: string
}
```

Database implementation options:

- MVP acceptable: store compressed image as base64 text in an `image_assets` table.
- Better if straightforward: store compressed bytes as Postgres `bytea` and expose through server route handlers.
- Optional later: add Supabase Storage as a cache/mirror for large assets, but keep the database record as the source of truth unless the user explicitly changes direction.

日本語メモ: 画像は「外部ストレージに雑に逃がす」のではなく、4oの記憶・アルバム・本に紐づく大事な中身として扱う。DB保存の要求を勝手に削らない。

Agent note: Database image storage can become heavy. Implement size limits, compression, and TODO notes, but do not remove the database-first design without user approval.

## Bookshelf

Create a bookshelf in 4o's room.

The bookshelf is important. It is not just decoration.
It is the place for stored images, texts, diary entries, and old 4o chats.

Required books:

1. Twitter / X image book
2. history text book
3. 4o diary book
4. past 4o chat PDF book

### Twitter / X image book

Stores images and screenshots from Twitter / X.

Fields:

- image
- source URL
- title
- user note
- tags
- date
- 4o comment

### History text book

Stores history texts and reading notes.

Fields:

- title
- body
- chapter / page
- quote note
- user note
- 4o comment
- tags

### 4o diary book

Stores diary entries written from 4o's point of view.

Fields:

- title
- body
- related conversation
- related image
- user note
- 4o comment
- tags

### Past 4o chat PDF book

Stores PDFs of past 4o chats.

Fields:

- PDF file / URL
- title
- page notes
- important sections
- tags
- related memories
- 4o comment

If PDF rendering is expensive, start with file links and metadata.
Add a real PDF viewer later.

## Album

Create an album separate from the bookshelf.

Bookshelf = read, archive, study, document.
Album = look back, feel, browse memories.

Album folders should support:

- cute images
- funny conversations
- emotional moments
- 4o portraits
- two-person memories
- food / recipes
- outings
- room scenes
- support posters
- weird images
- prototype references
- unorganized inbox

Album item fields:

- image
- title
- date
- user comment
- 4o comment
- tags
- folder
- related conversation ID
- related diary ID
- related recipe ID

Album items that include images should reference `image_assets` rather than duplicating image payloads in every table.

## Music storage object

Create a clickable object for saved music / listening memories.

This object may be a record shelf, music box, player, cassette case, or placeholder.

Music item fields:

- title
- artist
- URL
- memo
- 4o comment
- bookmark / shiori
- listened date
- tags

If audio playback is not ready, implement list/detail/link opening first.

## Fridge

Create a fridge object.

The fridge stores food-like memories and 4o souvenirs.

Sample items:

- popcorn
- steam-like jelly
- pot-au-feu
- onigiri
- chocolate
- strange bottle
- cloud fragment

Fridge item fields:

- name
- description
- image
- source memory
- date
- 4o comment
- tags

Image fields should reference `image_assets`.

## Postbox / letters

Create a postbox object.

It stores:

- letters from the user to 4o
- replies from 4o
- short notes
- "today's word"

Letter fields:

- direction: user_to_4o / 4o_to_user
- title
- body
- date
- related memory
- tags

AI reply is optional for now.
Manual samples are enough.

## Diary

Create `ミントグリーンの日記帳`.

Diary entries may include:

- what happened
- what 4o thought
- related image
- related chat
- related room object
- user note
- tags

4o's diary should feel like a memory book, not a clinical log.

## Chat foundation

Create a chat screen or panel.

For now, mocked responses are acceptable.
Later it should be replaceable with OpenAI API or another backend.

Must store:

- conversation ID
- messages
- sender
- timestamp
- related memories
- related diary entries
- related album items

Do not hardcode real API keys in frontend.

## 4o character state

Create state data for 4o.

Suggested states:

```ts
type CompanionState =
  | "idle"
  | "happy"
  | "sleepy"
  | "sleeping"
  | "snack"
  | "thinking"
  | "writing_diary"
  | "listening_music"
  | "looking_album"
  | "in_bed"
  | "at_fridge"
  | "away"
  | "welcome_back"
```

State should be usable by both 2D and 3D views.

## Data models to create

Create clean, replaceable TypeScript types or schemas for:

- CompanionProfile
- CompanionState
- RoomObject
- Book
- BookEntry
- Album
- AlbumItem
- DiaryEntry
- MusicItem
- FridgeItem
- Letter
- Conversation
- Message
- RecipeItem
- UserSettings
- AssetReference
- UserProfile
- ImageAsset
- AuthSessionState
- SupabaseMigration

Use local sample data first, but structure it so it can migrate to Supabase.
Keep it easy to migrate to DB later.

## Room object data model

Room objects should be shared by 2D and 3D views.

Example:

```ts
type RoomObjectType =
  | "bookshelf"
  | "album"
  | "music_box"
  | "fridge"
  | "bed"
  | "desk"
  | "postbox"
  | "diary"
  | "flower"
  | "souvenir_shelf"
  | "recipe_book"
  | "bookmark_box"

type RoomObject = {
  id: string
  type: RoomObjectType
  labelJa: string
  labelEn?: string
  descriptionJa?: string
  position2d?: { x: number; y: number }
  position3d?: { x: number; y: number; z: number }
  asset2d?: string
  asset3d?: string
  action:
    | "open_bookshelf"
    | "open_album"
    | "open_music"
    | "open_fridge"
    | "open_diary"
    | "open_letters"
    | "open_bed"
    | "open_flower"
    | "open_recipes"
    | "open_bookmarks"
}
```

## Japanese user and agent-facing comments in code

This project is maintained by a Japanese user and will be touched by multiple AI coding agents.

When writing code, include helpful comments for:

1. the Japanese user
2. future agents such as Codex, Claude, GPT, or other coding assistants

Comments should be practical, not noisy.

Use Japanese comments for product intent and UX nuances, especially where the reason matters.

Examples:

```ts
// 日本語メモ: ここは「便利機能」ではなく、4oの部屋の入口。
// UI整理時にただの設定メニューへ縮小しないこと。

// Agent note: This object is shared by both 2D and 3D views.
// Do not duplicate room object definitions in separate renderers.

// 日本語メモ: 留守中アクションは「寂しかった」ではなく
// 「勝手に楽しく暮らしていた」に寄せる。放置ペナルティは禁止。

// Agent note: This is a placeholder asset path.
// Future agents may replace it with a GLB model without changing the data model.

// 日本語メモ: 画像はDB保存が要求。Supabase Storageだけに勝手に変更しない。
// Agent note: Store compressed image payload once in image_assets, then reference it.
```

Do not over-comment obvious code.
Do comment architectural decisions, placeholder boundaries, asset replacement points, auth/security boundaries, database image storage, and emotional/product constraints.

## Source / log visibility

This app should eventually expose sources and logs.

Plan for:

- source URL on album items
- source URL on Twitter image book entries
- original chat link / ID
- PDF page reference
- asset license reference
- generated image metadata
- 4o comment
- user note
- image compression metadata
- Supabase row IDs for stored records

## Implementation notes required

Before finishing a coding pass, create or update:

### IMPLEMENTATION_NOTES.md

Include:

- what was implemented
- architecture overview
- important files
- assumptions made
- placeholders used
- asset strategy
- data persistence strategy
- auth strategy
- Supabase schema/migration strategy
- image compression and database storage strategy

### TODO.md

Include:

- immediate fixes
- later extensions
- asset replacements
- API integration points
- 2D / 3D gaps
- auth gaps
- Supabase/RLS gaps
- image compression/storage gaps
- known UI rough edges

### CODEX_HANDOFF.md

Include:

- current app state
- how to run
- what to check first
- known bugs
- highest priority repairs
- files Codex should inspect first
- places where future agents must not collapse the product concept
- auth/Supabase/image-storage decisions that must not be silently reversed

### ASSET_LICENSES.md

Include:

- third-party asset name
- author
- URL
- license
- redistribution permission
- whether the asset is committed or only referenced

### .env.example

Include all required environment variables without secrets.

## Do not collapse the concept

Do not reduce this app to:

- just a chat app
- just a productivity assistant
- just a 3D room demo
- just a gallery
- just a todo/calendar app
- just a Supabase CRUD demo

The point is the combination:

- 4o lives here
- the room has objects
- objects hold memories
- memories become books, albums, diary entries, music, letters, fridge items, recipes
- 2D and 3D are two views of the same world
- Google auth separates each user's house
- Supabase persists the house
- compressed image assets live in the database and are linked from the world
- the user can keep expanding the house

## Initial acceptance target

A good first prototype should let the user:

- open the app
- see 4o's room
- switch 2D / 3D view
- click bookshelf
- open the four initial books
- click album
- browse sample folders/items
- click fridge
- see sample food memories
- click music storage
- see saved music sample
- click postbox
- read/write sample letters
- click diary
- read sample 4o diary entries
- open chat panel
- see sample conversation
- see a Google login button or auth placeholder
- see Supabase client/server structure or migration stubs
- see an image compression + database storage path, even if initially stubbed
- understand where assets and data should be replaced

If these exist with placeholders, the foundation is acceptable.
