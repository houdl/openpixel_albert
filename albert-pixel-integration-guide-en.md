# Albert Demo - FeedMob Pixel Event Flow


Fake click

```
https://tracking.feedmob.com/api/v1/vendor/click?FEEDMOB_CLICK_ID=22794&FEEDMOB_PUBLISHER_ID=feed45ab56d2-b6d5-434e-b9d3-eaa34b47e8e8&APP_ID=65-45-TEST-APP_ID&CONVERSION_ID=402b99c6-8111-43c5-b229-1fbf575dbab8-TEST&APP_NAME=Tin-d8-TEST-APP_NAME&CREATIVE_ID=&CAMPAIGN_ID=1077&CONVER_TIME=&FEEDMOB_DEVICE_LANG={FEEDMOB_DEVICE_LANG}&FEEDMOB_DEVICE_MODEL={FEEDMOB_DEVICE_MODEL}&FEEDMOB_DEVICE_OS={test}&DEVICE_PLATFORM_ID=4D172826-7265-BC12-56B8-36AC5A2F79AF
```

## What Happens on Page Load

### 1. Snippet Execution (immediately on page load)

`index.html` in `<head>`:

```javascript
fmpix("init", "d7ab0ac71e9245f48c8d92d59140bfb6");  // Set client ID (test client)
fmpix("event", "pageload");                            // Trigger page load event
```

Since `fmpixel.js` hasn't finished loading yet, these two calls are **queued**.

### 2. Core Script Loaded

Once `fmpixel.js` is asynchronously loaded from CDN:

1. Processes queued `init` → initializes client UUID, generates user UID, stores in Cookie
2. Processes queued `pageload` → fires a pixel request to the tracking server

pageload tracking request:
```
https://pixel-api.feedmob.biz/tracker?id=d7ab0ac71e9245f48c8d92d59140bfb6&uid=1-en2n51az-mm330fbv&ev=pageload&ed=&v=1&dl=https%3A%2F%2Fawg440wogoko8cw4c0osssww.coolify-dev-pa.tonob.net%2F%3Fad%3Drf_test_add%26utm_source%3DFeedMob_TestyMcTest%26fm_publisher_id%3Dfeed45ab56d2-b6d5-434e-b9d3-eaa34b47e8e8_feed-click-id22794%26fm_click_id%3D22794%26fm_conversion_id%3D402b99c6-8111-43c5-b229-1fbf575dbab8-TEST%26utm_medium%3DTesty_McTest%26creative%3D%26utm_custom2%3D%26FEEDMOB_VENDOR_NAME%3DTesty_McTest%26FEEDMOBVENDORNAME%3DTestyMcTest%26FEEDMOB_BILLING_METHOD%3DCPI%26feedmob_token%3D60c124b6-3ccb-41b3-b886-f400f8fcce80%26FEEDMOB__TOKEN%3D60c124b6-3ccb-41b3-b886-f400f8fcce80&rl=&ts=1774682182162&de=UTF-8&sr=1470x956&vp=1470x195&cd=30&dt=Albert%20%7C%20Your%20personal%20financial%20assistant&bn=Chrome%20146&md=false&ua=Mozilla%2F5.0%20(Macintosh%3B%20Intel%20Mac%20OS%20X%2010_15_7)%20AppleWebKit%2F537.36%20(KHTML%2C%20like%20Gecko)%20Chrome%2F146.0.0.0%20Safari%2F537.36&tz=-480&utm_source=FeedMob_TestyMcTest&utm_medium=Testy_McTest&utm_term=&utm_content=&utm_campaign=&utm_partner=&fm_click_id=22794&fm_publisher_id=feed45ab56d2-b6d5-434e-b9d3-eaa34b47e8e8_feed-click-id22794&fm_conversion_id=402b99c6-8111-43c5-b229-1fbf575dbab8-TEST
```


Debug panel output:
```
QUEUED: init d7ab0ac71e9245f48c8d92d59140bfb6
QUEUED: event pageload
pageload Page loaded automatically
INFO Direct pixel integration (no GTM)
```

---

## What Happens on Sign Up


### Fill in the form and click Sign up → triggers REG event

`index.html`:

```javascript
document.getElementById('signup-form').addEventListener('submit', function(e) {
  e.preventDefault();
  var userName = document.getElementById('signup-name').value;
  var userEmail = document.getElementById('signup-email').value;

  // Call fmpix directly, no GTM needed
  fmpix('event', 'REG');
});
```

The `addEventListener` does two things:
1. Calls `fmpix('event', 'REG')` → fires a pixel request to the tracking server
2. Writes to the Debug panel for display (for debugging purposes only, can be ignored)

REG tracking request:
```
https://pixel-api.feedmob.biz/tracker?id=d7ab0ac71e9245f48c8d92d59140bfb6&uid=1-en2n51az-mm330fbv&ev=REG&ed=%7B%22name%22%3A%221%22%2C%22email%22%3A%22houdl730%40gmail.com%22%7D&v=1&dl=https%3A%2F%2Fawg440wogoko8cw4c0osssww.coolify-dev-pa.tonob.net%2F%3Fad%3Drf_test_add%26utm_source%3DFeedMob_TestyMcTest%26fm_publisher_id%3Dfeed45ab56d2-b6d5-434e-b9d3-eaa34b47e8e8_feed-click-id22794%26fm_click_id%3D22794%26fm_conversion_id%3D402b99c6-8111-43c5-b229-1fbf575dbab8-TEST%26utm_medium%3DTesty_McTest%26creative%3D%26utm_custom2%3D%26FEEDMOB_VENDOR_NAME%3DTesty_McTest%26FEEDMOBVENDORNAME%3DTestyMcTest%26FEEDMOB_BILLING_METHOD%3DCPI%26feedmob_token%3D60c124b6-3ccb-41b3-b886-f400f8fcce80%26FEEDMOB__TOKEN%3D60c124b6-3ccb-41b3-b886-f400f8fcce80&rl=&ts=1774681754061&de=UTF-8&sr=1470x956&vp=1470x478&cd=30&dt=Albert%20%7C%20Your%20personal%20financial%20assistant&bn=Chrome%20146&md=false&ua=Mozilla%2F5.0%20(Macintosh%3B%20Intel%20Mac%20OS%20X%2010_15_7)%20AppleWebKit%2F537.36%20(KHTML%2C%20like%20Gecko)%20Chrome%2F146.0.0.0%20Safari%2F537.36&tz=-480&utm_source=FeedMob_TestyMcTest&utm_medium=Testy_McTest&utm_term=&utm_content=&utm_campaign=&utm_partner=&fm_click_id=22794&fm_publisher_id=feed45ab56d2-b6d5-434e-b9d3-eaa34b47e8e8_feed-click-id22794&fm_conversion_id=402b99c6-8111-43c5-b229-1fbf575dbab8-TEST
```

Debug panel output:
```
REG {"name":"test","email":"test@feedmob.com"}
```

---

## Complete Event Timeline

```
Page Load
  |
  +-- [Auto] QUEUED: init + QUEUED: pageload    <-- Snippet execution
  +-- [Auto] pageload                            <-- Processed after fmpixel.js loads
  |
  |   ...user browses the page...
  |
  +-- [User] Submit form -> REG                  <-- Registration event with username and email
  |
  +-- [Auto] pageclose                           <-- Triggered automatically when user leaves
```

Actual Debug panel output (reverse chronological order):
```
[14:21:39] REG {"name":"test","email":"test@feedmob.com"}
[14:21:18] INFO Direct pixel integration (no GTM)
[14:21:18] pageload Page loaded automatically
[14:21:18] QUEUED: event pageload
[14:21:18] QUEUED: init d7ab0ac71e9245f48c8d92d59140bfb6
```

---

## How to Verify

Open browser DevTools (`F12`) → Network tab → filter by `tracker`. You'll see each event sending a request to `pixel-api.feedmob.biz/tracker`.

To test the full tracking pipeline, visit with MMP parameters:
```
https://pixel-api.feedmob.biz/tracker?utm_source=FeedMob_Test&fm_conversion_id=CONV123&fm_publisher_id=PUB456&fm_click_id=CLICK789
```
