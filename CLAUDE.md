# Meet Site

## Team emails
Team members' emails follow the pattern `firstname@britishprogress.org` (e.g. `martha@britishprogress.org`). Derive from first name — don't ask the user.

Exception: Matthew Grant uses `matthew.grant@britishprogress.org` (since `matthew@` is already taken by Matthew Stubbs).

## Adding a team member

1. **Get their details**: first name, last name, role, email, and photo URL from `britishprogress.org/about#team`. The photo filename is the `url` param in the `_next/image` URL (e.g. `m%20(1).webp`).

2. **`index.html` — team array**: Add an entry at the end of the `team` array (new index = previous length):
   ```js
   { first: "Name", last: "Surname", role: "Their Role", img: "photo.webp", email: "name@britishprogress.org" }
   ```

3. **`index.html` — suggestions**: Add an entry for the new index in the `suggestions` object with likely collaborators (by index):
   ```js
   12: [0, 1, 7],  // Name → David, Julia, Tom
   ```

4. **`booking-urls.json`**: Run this Python snippet to generate all combinations (238 entries for a 13-person team), then rebuild the file in canonical order:
   ```python
   # Add new slug to the slugs list, then run:
   slugs = ['david','julia','maxx','pedro','ed','alys','kane','tom','ezra','matthew-s','esther','matthew-g','martha']  # update as needed
   # See the script used when adding Martha for the full logic
   ```
   The script adds solo (4 suffixes), pairs (3 suffixes each), and trios (3 suffixes each), all with empty string values.

## Removing a team member

1. **`index.html` — team array**: Delete their line from the `team` array.

2. **`index.html` — suggestions**: Delete their entry from `suggestions`, remove their index from all other entries' arrays, then **renumber** all indices that were higher than theirs (each decrements by 1). Update the comments too.

3. **`booking-urls.json`**: Run Python to remove all keys containing their first name slug:
   ```python
   cleaned = {k: v for k, v in data.items() if 'firstname' not in k}
   ```

## Slug rules

- Each person's URL slug is their lowercase first name (e.g. `david`, `martha`).
- If two people share a first name, the slug is `firstname-lastinitial` (e.g. `matthew-s`, `matthew-g`).
- The `getSlug()` function in `index.html` handles this automatically.
