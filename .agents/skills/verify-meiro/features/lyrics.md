# Lyrics

The Now Playing panel has a Lyrics tab, with the lyrics of the song playing. A
song with timed lyrics lights the line being sung and keeps it in the middle;
songs whose source times each word light each word as it is sung. The mini
player and the macOS notch show the same lyrics in their own layouts.

## Sub-features

- `lyrics-order`: Better Lyrics is asked first, then YouTube Music, then LRCLIB, and the first source with timed lyrics wins. A source that fails or has nothing is skipped quietly.
- `lyrics-credits`: the lookup names the track's artist, album and length, which Better Lyrics keys its cache on. An artist a list left out is filled in from what another list named, and a lookup made before the track's length was known is made again once it is.
- `lyrics-word-sync`: Better Lyrics times each word, or syllable for a song in Japanese or Korean, so a line fills in as the singer reaches each word. Words still to come are dim.
- `lyrics-line-sync`: YouTube Music and LRCLIB time whole lines, which stay lit for as long as they are sung and fade with their distance from it.
- `lyrics-source`: the Lyrics tab names where the lyrics came from: Better Lyrics, YouTube Music, or LRCLIB.
- `lyrics-modes`: Synced and Static choose between the timed lines and the plain text, remembered in Settings. Static shows the words of the timed lines joined, when the source has no plain text.
- `lyrics-seek`: clicking a line seeks to it.
- `lyrics-fallback`: no lyrics and a failed lookup are different states; a failure is tried again the next time the tab opens.
- `lyrics-cache`: lyrics are kept per track while it plays; changing songs supersedes the lookup.

## How to get to it (user POV)

Play a song. Open the mini player from the tray (Click the music note in the
menu bar on macOS, or the tray icon on Linux and Windows), or open Now Playing
in the main window on a wide screen, and choose Lyrics. With word-synced
lyrics, the line being sung fills in word by word. The name at the right of the
Synced and Static buttons is where the lyrics came from.

## Driving it with the headless tester

Set `a.betterLyrics` to return `[]lyrics.Line` with `Words` for word-synced
lyrics, or `a.lyricsSource` for LRCLIB lines. Both are stubbed to nothing in
`newTestApp(t)`, so no test reaches the network. Set `a.total` together with
`a.resumeID` and `a.scrub` (in seconds) to place playback at a moment: without
a known duration the player's scrubber resets `a.scrub` to 0 every frame.

The mini player shows the lyrics tab at 344 × 640. The Now Playing panel only
shows its side panel on a wide window: build `ui.NewTester(a.view, 1400, 800)`,
call `a.openPlayer(tabLyrics, false)`, draw a frame, then advance the clock so
the player finishes rising.

## Gotchas

- `m3.Chip` labels repeat ("Synced", "Static") on both the panel and the mini
  player; a tester can only click what its own window shows.
- A line with no words still renders through `m3.EmphasizedText`; only lines
  with `Words` use `ui.RichText`, so a lyric-only line keeps its old look.
- The footer text drops its `Source: ` prefix before showing, so assert on
  `Better Lyrics`, not `Source: Better Lyrics`.
- Better Lyrics serves cached songs without a key and wants one for the rest,
  so a live run can show YouTube Music or LRCLIB lyrics for a song it has not
  cached. That is the fallback working, not a failure.
- Better Lyrics finds a cached song by artist and length, so `a.total` matters
  in tests too: a source stub can assert the duration it was asked for.
- Album rows from YouTube Music carry no artist, so album playback asks Better
  Lyrics with none and falls through unless another list named the artist. A
  playlist row carries both, and is the way to see Better Lyrics live.
