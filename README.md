# GoalCanvas AI

Turn today's goals into a unique, cinematic motivational wallpaper — every time you tap Generate.

## What's in this project

A complete Android Studio project (Kotlin + Jetpack Compose, Material 3, Hilt DI, Room):

- **Goal input** — add/edit/remove daily goals in a clean Compose list.
- **AI Prompt Builder** (`ai/PromptBuilder.kt`) — never sends your raw goal text to any external
  service. Instead it composes a full creative brief: background style, art style, color
  palette, typography, layout, and lighting/effects, all randomized per generation, plus a
  rich image-generation prompt with mood/color psychology baked in.
- **Two-layer rendering pipeline**:
  - `ai/LocalFallbackGenerator.kt` — a fully offline procedural background (angled gradients,
    glow blobs, grain, vignette) so the app **always** produces a finished, premium-looking
    wallpaper even with zero network access.
  - `ai/RemoteImageGenerationService.kt` — an optional, provider-agnostic HTTP client you can
    point at any text-to-image API (Stability AI, your own proxy, etc.). Wire in your endpoint
    + API key in `ApiConfig`; on any failure it silently falls back to the offline renderer.
  - `wallpaper/WallpaperCompositor.kt` — draws the goal checklist + a never-repeating
    motivational quote (`ai/QuoteGenerator.kt`) over the background with randomized fonts,
    sizes, rotation, glow, and shadow.
- **WallpaperManager integration** (`wallpaper/WallpaperManagerHelper.kt`) — one-tap Set Home /
  Set Lock / Set Both, no manual steps.
- **History + Favorites** — Room database, capped at the 30 most recent non-favorited
  wallpapers; favorites are exempt from pruning.
- **Only the permissions it needs**: `SET_WALLPAPER`, `INTERNET`, `ACCESS_NETWORK_STATE`
  (the last two only matter if you configure a remote AI provider).

## Important — read before you build

This project was generated in a sandbox that has **no Android SDK, no Gradle distribution
access, and no code signing available**, so I could not compile it into an actual `.apk` here.
What you have is the complete, real source (no TODO placeholders in the app logic) ready to
open and build yourself:

1. Install [Android Studio](https://developer.android.com/studio) (Koala or newer).
2. `File → Open` and select the unzipped `GoalCanvasAI/` folder.
3. Let Gradle sync (Android Studio will fetch the wrapper jar and dependencies automatically —
   the `gradle-wrapper.properties` here points at Gradle 8.7).
4. Optional: configure remote AI image generation in
   `app/src/main/java/com/goalcanvas/ai/ai/RemoteImageGenerationService.kt` →
   `ApiConfig.baseUrl` / `ApiConfig.apiKey`. Leave blank to run fully offline — the app still
   works great, since the local procedural renderer is the primary path this build is tuned for.
5. `Run ▶` on a device/emulator, or `Build → Generate Signed Bundle / APK` for a signed release
   build once you've set up your own signing key.

## Architecture at a glance

```
ui/            Compose screens (Home, History/Favorites), theme, ViewModel
ai/            DesignSpec, PromptBuilder, palettes, quote generator, image sourcing
wallpaper/     Canvas compositor, WallpaperManager helper, file storage
data/          Room entity/DAO/database, repository (single source of truth)
di/            Hilt modules
```

Every "Generate"/"Regenerate" tap calls `PromptBuilder.build(goals)`, which draws a fresh
random seed and picks new background style, art style, palette, typography, layout, and
effects from curated pools — so results feel designed rather than randomly noisy, while no
two wallpapers are ever identical.

## Extending it

- Swap in a real text-to-image model by adapting `buildRequestBody` / `parseResponse` in
  `RemoteImageGenerationService.kt` to your provider's actual API contract.
- Add more `BackgroundStyle` / `ArtStyle` entries or `Palettes` in the `ai/` package — the
  randomizer and prompt builder pick these up automatically.
- Add sharing: wire an `Intent.ACTION_SEND` using `WallpaperFileStore.shareUri()` from the
  Share button in `HomeScreen.kt` (stubbed with a comment where it belongs).
