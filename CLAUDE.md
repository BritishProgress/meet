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

4. **`index.html` — directBookingOverrides**: Only if this person needs a direct-load booking link (skips method selection). This map is keyed by **team index** — add an entry for the new index. If they don't need one, leave it alone.

5. **`booking-urls.json`**: Regenerate the full file (see [Regenerating booking-urls.json](#regenerating-booking-urlsjson)) with the updated slug list.

6. **Validate** (see [Validating after any roster change](#validating-after-any-roster-change)).

## Removing a team member

1. **`index.html` — team array**: Delete their line from the `team` array.

2. **`index.html` — suggestions**: Delete their entry from `suggestions`, remove their index from all other entries' arrays, then **renumber** all indices that were higher than theirs (each decrements by 1). Update the comments too.

3. **`index.html` — directBookingOverrides**: This map is keyed by **team index**, so a removal silently reattaches an override to whoever shifts into that slot (this is how Maxx's Calendly link ended up on Pedro's page). Delete the departing person's entry, then **renumber** all keys higher than theirs (each decrements by 1). Update the comments too.

4. **`booking-urls.json`**: Regenerate the full file (see [Regenerating booking-urls.json](#regenerating-booking-urlsjson)) with the departing person's slug removed. Do **not** filter by substring (`'tom' in k`) — that also matches `tom-blake`/`tom-barrowcliff`; regenerate from the slug list instead.

5. **Validate** (see [Validating after any roster change](#validating-after-any-roster-change)).

## Slug rules

- Each person's URL slug is their lowercase first name (e.g. `david`, `martha`).
- If two people share a first name, the slug is `firstname-lastinitial` (e.g. `matthew-s`, `matthew-g`).
- If the last initial *also* collides (e.g. Tom **B**lake vs Tom **B**arrowcliff), the slug falls back to the full surname (`tom-blake`, `tom-barrowcliff`).
- The `getSlug()` function in `index.html` handles all of this automatically — always derive slugs by running it, never hand-write them.

## Regenerating booking-urls.json

The file maps every 1-, 2-, and 3-person combination to a booking URL (mostly empty strings; a few are real Google Calendar / Calendly links). It is fully generated — regenerate it whenever the roster or a slug changes, **preserving existing non-empty values**:

```python
import json, itertools, collections
old = json.load(open('booking-urls.json'))
# Slugs in TEAM-INDEX order (must match index.html's team array order, using getSlug's output):
slugs = ['david','julia','pedro','alys','kane','tom-blake','ezra','matthew-s',
         'esther','matthew-g','martha','shaamini','bel','tom-barrowcliff','adash']
SOLO  = ['80-strand','in-person','video-call','phone-call']  # solo: 4 suffixes
MULTI = ['80-strand','in-person','video-call']               # pairs/trios: 3 suffixes (no phone-call)
def base(c): return c[0] if len(c)==1 else '-'.join(c[:-1]) + '-and-' + c[-1]
entries = []
for r in (1,2,3):
    for c in itertools.combinations(slugs, r):   # combinations preserve index order
        entries.append((base(c), SOLO if r==1 else MULTI))
entries.sort(key=lambda e: e[0])                 # canonical order: bases sorted as strings
result = collections.OrderedDict((f'{b}-{s}', '') for b, sufs in entries for s in sufs)
for k, v in old.items():                         # carry over real booking links
    if v and k in result: result[k] = v
json.dump(result, open('booking-urls.json','w'), indent=2, ensure_ascii=False)
open('booking-urls.json','a').write('\n')
```

Sizes: an n-person team yields n·4 + C(n,2)·3 + C(n,3)·3 keys (e.g. 15 people → 1740).

## Validating after any roster change

`getSlug()` and `suggestions` can silently drift out of sync with the `team` array (an earlier removal left a stale entry and misaligned indices). After any add/remove/rename, run this check and confirm it passes:

```bash
node -e "
const html=require('fs').readFileSync('index.html','utf8');
const team=eval(html.match(/const team = (\[[\s\S]*?\]);/)[1]);
const suggestions=eval('('+html.match(/const suggestions = (\{[\s\S]*?\});/)[1]+')');
const getSlug=i=>{const m=team[i],f=m.first.toLowerCase();if(!team.some((x,j)=>j!==i&&x.first===m.first))return f;const li=m.last[0].toLowerCase();const col=team.some((x,j)=>j!==i&&x.first===m.first&&x.last[0].toLowerCase()===li);return f+'-'+(col?m.last.toLowerCase():li);};
const slugs=team.map((_,i)=>getSlug(i));
console.assert(new Set(slugs).size===slugs.length,'DUPLICATE SLUGS');
const keys=Object.keys(suggestions).map(Number);
console.assert(keys.length===team.length&&keys.every(k=>k>=0&&k<team.length),'SUGGESTIONS KEYS != team indices');
for(const [k,arr] of Object.entries(suggestions))for(const v of arr)console.assert(v>=0&&v<team.length&&String(v)!==k,'BAD SUGGESTION REF '+k+'->'+v);
console.log('team',team.length,'slugs OK, suggestions OK');
"
```

Also confirm `booking-urls.json` key count matches the formula above and contains no keys for a removed slug.
