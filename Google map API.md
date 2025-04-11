If you want to **query restaurants (or any places) using the Google Maps API even when they are closed**, you can definitely do that — the **Places API** allows you to retrieve place details regardless of whether the business is currently open or not.

Here’s how to do it:

---

### ✅ Option 1: Use **Nearby Search** (Preferred for proximity search)

#### Endpoint:
```
https://maps.googleapis.com/maps/api/place/nearbysearch/json
```

#### Example Request:
```http
https://maps.googleapis.com/maps/api/place/nearbysearch/json
?location=40.748817,-73.985428
&radius=1500
&type=restaurant
&key=YOUR_API_KEY
```

> This returns a list of nearby restaurants — regardless of whether they are open or closed.

#### Optional: Filter for closed/open now
- If you want **only open** places, add `opennow=true`
- But if you want **all** (including closed), just **don’t include `opennow`** ✅

---

### ✅ Option 2: Use **Text Search**

#### Endpoint:
```
https://maps.googleapis.com/maps/api/place/textsearch/json
```

#### Example:
```http
https://maps.googleapis.com/maps/api/place/textsearch/json
?query=restaurants+in+Times+Square
&key=YOUR_API_KEY
```

Again, this returns restaurants regardless of open/closed status — unless you filter with `opennow=true`.

---

### ✅ Option 3: Get Place Details (to check hours)

Once you have a place's `place_id` (from the above search), use:

#### Endpoint:
```
https://maps.googleapis.com/maps/api/place/details/json
```

#### Example:
```http
https://maps.googleapis.com/maps/api/place/details/json
?place_id=ChIJN1t_tDeuEmsRUsoyG83frY4
&fields=name,opening_hours
&key=YOUR_API_KEY
```

This returns the opening hours and tells you if it’s open now.

---

### TL;DR
| Goal | How to |
|------|--------|
| Get restaurants, open or closed | Use Nearby/Text Search without `opennow=true` |
| Only get open restaurants | Add `opennow=true` to query |
| Know when a place is open | Use Place Details API and check `opening_hours` |
