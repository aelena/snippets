This creates a playlist in Spotify from a CSV file of very simple structure

Artist - Album Name - Release date (YYYYMMDD)

to make sure the playlist is created in chronological order. This is just something I wanted, not essential or needed by the API. The script uses spotipy. It preserves the CSV order and can either add the first track from each album or the full album (in album track order).

`pip install spotipy pandas python-dotenv`

Both ChatGPT and Gemini on Google Colab helped me with this. 

## Set up Auth

Set environment variables before running (or use a .env file in the same folder):
    
```
    SPOTIPY_CLIENT_ID=...
    SPOTIPY_CLIENT_SECRET=...
    SPOTIPY_REDIRECT_URI=http://localhost:8888/callback
```
The script requests scopes: playlist-modify-private, playlist-modify-public

## Usage

```python
python make_spotify_playlist_from_csv.py --csv path/to/metal_albums_sorted_full.csv \
    --playlist-name "Chrono Metal (Albums)" \
    --public True \
    --mode album_tracks \
    --market ES
```

- album_tracks       -> add every track of each album, preserving album track order
- first_track        -> add the first (opening) track from each album only

## Notes

The playlist will follow the CSV order (already chronological) and keep album-internal order.
Spotify "search" isn't perfect. We filter by artist, album name, and year when possible. The script prints any albums it couldn't confidently match.
Adds tracks in batches of up to 100 items per Spotify API.

## Dependencies

```python
!pip -q install spotipy pandas python-dotenv

import io, os, time
from typing import Optional, List, Tuple

import pandas as pd
from spotipy import Spotify
from spotipy.oauth2 import SpotifyOAuth
```

If you are running this from Google Colab, which I find convenient, add this too

```python
from google.colab import userdata
from google.colab import files
```

You will need to set your secrets in Colab itself. And read them with something like this

```python
os.environ["SPOTIPY_CLIENT_ID"] = userdata.get("SPOTIPY_CLIENT_ID")
os.environ["SPOTIPY_CLIENT_SECRET"] = userdata.get("SPOTIPY_CLIENT_SECRET")
os.environ["SPOTIPY_REDIRECT_URI"] = userdata.get("SPOTIPY_REDIRECT_URI")
```

# The Script

```python

import os
import time
import argparse
from typing import Optional, List, Tuple
import pandas as pd
from spotipy import Spotify
from spotipy.oauth2 import SpotifyOAuth

SCOPE = "playlist-modify-private playlist-modify-public"

def parse_args():
    print("Entering parse_args()")
    ap = argparse.ArgumentParser()
    ap.add_argument("--csv", required=True, help="Input CSV with columns: artist,album,yyyymmdd")
    ap.add_argument("--playlist-name", required=True, help="Name of the playlist to create")
    ap.add_argument("--public", type=lambda x: x.lower() in {"1","true","yes","y"}, default=False, help="Create as public playlist (default False)")
    ap.add_argument("--mode", choices=["album_tracks","first_track"], default="album_tracks")
    ap.add_argument("--market", default="ES", help="Spotify market for track availability, e.g. ES, US")
    # For running in Colab, we'll parse known args and ignore the rest
    args, unknown = ap.parse_known_args()
    print(f"Exiting parse_args() with args: {args}")
    return args

def auth_spotify() -> Spotify:
    print("Entering auth_spotify()")
    # Use an authentication flow suitable for Colab
    # This will print a URL and ask the user to paste a code back
    redirect_uri = os.environ["SPOTIPY_REDIRECT_URI"]
    auth_manager = SpotifyOAuth(
        scope=SCOPE,
        show_dialog=True,  # Important for Colab
        open_browser=False, # Prevent opening a browser tab
        redirect_uri=redirect_uri # Use the redirect URI from env vars
    )
    # This will open a browser tab if open_browser is True (which it is by default).
    # In Colab, we want to get the URL and have the user manually open it.
    auth_url = auth_manager.get_authorize_url()
    print(f"Please visit this URL to authorize this application: {auth_url}")
    response = input("Paste the redirect URL here (or just the code from it): ") # Updated prompt
    auth_manager.get_access_token(response)
    print("Spotify authentication successful.")
    return Spotify(auth_manager=auth_manager)


def get_current_user_id(sp: Spotify) -> str:
    print("Entering get_current_user_id()")
    me = sp.me()
    print(f"Authenticated as: {me.get('display_name', 'Unknown')}")
    print(f"Exiting get_current_user_id() with user_id: {me['id']}")
    return me["id"]

def yyyymmdd_to_year(yyyymmdd: str) -> Optional[int]:
    print(f"Entering yyyymmdd_to_year() with input: {yyyymmdd}")
    s = str(yyyymmdd).strip()
    if len(s) >= 4 and s[:4].isdigit():
        year = int(s[:4])
        print(f"Exiting yyyymmdd_to_year() with year: {year}")
        return year
    print("Exiting yyyymmdd_to_year() with None")
    return None

def search_album(sp: Spotify, artist: str, album: str, year: Optional[int], market: str) -> Optional[dict]:
    print(f"Entering search_album() for artist: {artist}, album: {album}, year: {year}, market: {market}")
    query_parts = [f'album:"{album}"', f'artist:"{artist}"']
    if year:
        query_parts.append(f"year:{year}")
    query = " ".join(query_parts)
    print(f"Searching Spotify with query: {query}")
    results = sp.search(q=query, type="album", limit=10, market=market)
    items = results.get("albums", {}).get("items", [])
    if not items and year:
        print("No items found with year, retrying without year.")
        # retry without year
        query = f'album:"{album}" artist:"{artist}"'
        results = sp.search(q=query, type="album", limit=10, market=market)
        items = results.get("albums", {}).get("items", [])
    if not items:
        print(f"Exiting search_album() - Album not found: {artist} - {album}")
        return None

    # Rank candidates: exact name (case-insensitive), then by release_date proximity to CSV year
    def score(item):
        name_match = int(item["name"].lower() == album.lower())
        rdate = item.get("release_date", "")[:4]
        ydiff = abs(int(rdate) - year) if (year and rdate.isdigit()) else 99
        return (name_match, - (10 - min(10, ydiff)))  # prefer name match, then smaller ydiff

    best = sorted(items, key=score, reverse=True)[0]
    print(f"Exiting search_album() - Found best match: {best['name']} by {best['artists'][0]['name']}")
    return best

def get_album_tracks(sp: Spotify, album_id: str, market: str) -> List[str]:
    print(f"Entering get_album_tracks() for album_id: {album_id}")
    tracks = []
    offset = 0
    while True:
        res = sp.album_tracks(album_id, limit=50, offset=offset, market=market)
        items = res.get("items", [])
        if not items:
            break
        tracks.extend([t["uri"] for t in items])
        offset += len(items)
        if len(items) < 50:
            break
    print(f"Exiting get_album_tracks() - Found {len(tracks)} tracks.")
    return tracks

def get_first_track(sp: Spotify, album_id: str, market: str) -> Optional[str]:
    print(f"Entering get_first_track() for album_id: {album_id}")
    res = sp.album_tracks(album_id, limit=1, offset=0, market=market)
    items = res.get("items", [])
    track_uri = items[0]["uri"] if items else None
    print(f"Exiting get_first_track() with track_uri: {track_uri}")
    return track_uri

def add_tracks_batched(sp: Spotify, playlist_id: str, uris: List[str]):
    print(f"Entering add_tracks_batched() with {len(uris)} tracks for playlist_id: {playlist_id}")
    BATCH = 100
    for i in range(0, len(uris), BATCH):
        print(f"Adding batch of {min(BATCH, len(uris) - i)} tracks starting from index {i}")
        sp.playlist_add_items(playlist_id, uris[i:i+BATCH])
        time.sleep(0.1)  # tiny pause to be polite
    print("Exiting add_tracks_batched()")

def main(csv_path: str, playlist_name: str, public: bool = False, mode: str = "album_tracks", market: str = "ES"):
    print(f"Entering main() with csv_path: {csv_path}, playlist_name: {playlist_name}, public: {public}, mode: {mode}, market: {market}")
    # Simulate args for Colab
    class Args:
        def __init__(self, csv, playlist_name, public, mode, market):
            self.csv = csv
            self.playlist_name = playlist_name
            self.public = public
            self.mode = mode
            self.market = market
    args = Args(csv_path, playlist_name, public, mode, market)

    print(f"Reading CSV from: {args.csv}")
    # Read and keep CSV order
    df = pd.read_csv(args.csv, dtype=str)
    print(f"CSV read successfully. Shape: {df.shape}")
    for col in ["artist","album","yyyymmdd"]:
        if col not in df.columns:
            raise SystemExit(f"CSV must contain columns: artist, album, yyyymmdd (missing {col})")
    print("CSV columns validated.")

    sp = auth_spotify()
    user_id = get_current_user_id(sp)

    # Create playlist
    print(f"Creating playlist: {args.playlist_name}")
    playlist = sp.user_playlist_create(user_id, args.playlist_name, public=args.public, description="The albums I've listened the most in my life")
    playlist_id = playlist["id"]
    print(f"Created playlist: {playlist['name']} ({playlist_id})")

    missing: List[Tuple[str,str]] = []
    track_uris: List[str] = []

    print("Processing albums from CSV...")
    for index, row in df.iterrows():
        artist = str(row["artist"]).strip()
        album = str(row["album"]).strip()
        ymd = str(row["yyyymmdd"]).strip()
        year = yyyymmdd_to_year(ymd)
        print(f"\nProcessing row {index}: Artist: {artist}, Album: {album}, YYYYMMDD: {ymd}, Year: {year}")


        alb = search_album(sp, artist, album, year, market=args.market)
        if not alb:
            print(f"[WARN] Not found: {artist} - {album}")
            missing.append((artist, album))
            continue

        if args.mode == "album_tracks":
            uris = get_album_tracks(sp, alb["id"], args.market)
        else:
            uris = get_first_track(sp, alb["id"], args.market)
            uris = [uris] if uris else [] # Ensure uris is a list

        if not uris:
            print(f"[WARN] No tracks found for album: {artist} - {album}")
            missing.append((artist, album))
            continue

        print(f"Added {'all tracks' if args.mode=='album_tracks' else 'first track'} from: {artist} - {alb['name']} ({alb.get('release_date','')})")
        track_uris.extend(uris)

    # Add to playlist in CSV order
    if track_uris:
        add_tracks_batched(sp, playlist_id, track_uris)
        print(f"Done. Added {len(track_uris)} tracks in CSV order to playlist '{args.playlist_name}'.")
    else:
        print("No tracks found to add to the playlist.")


    if missing:
        print("\nAlbums not matched (review manually):")
        for a, b in missing:
            print(f"  - {a} — {b}")
    print("Exiting main()")

if __name__ == "__main__":
    # Example usage in Colab: Replace with your CSV path and desired playlist name
    main(csv_path="albums.csv", playlist_name="My Chronological Playlist")

```

I liberally added `print` statements, so you will hopefully see output like this

```
Processing row 0: Artist: Judas Priest, Album: Screaming for Vengeance, YYYYMMDD: 19820701, Year: 1982
Entering search_album() for artist: Judas Priest, album: Screaming for Vengeance, year: 1982, market: ES
Searching Spotify with query: album:"Screaming for Vengeance" artist:"Judas Priest" year:1982
Exiting search_album() - Found best match: Screaming For Vengeance by Judas Priest
Entering get_album_tracks() for album_id: 0V7mTTzioHiYIjfM8ATZBI
Exiting get_album_tracks() - Found 12 tracks.
Added all tracks from: Judas Priest - Screaming For Vengeance (1982)
Entering yyyymmdd_to_year() with input: 19831115
Exiting yyyymmdd_to_year() with year: 1983
```

## Possible improvements

- filter for a specific release version, if you want to avoid multiple remasters and prefer the original version
- filter out all those silly live tracks, demos, live demos and shit they put in re-releases. Even filtering title tracks that contain the words demo, live could be pretty effective


