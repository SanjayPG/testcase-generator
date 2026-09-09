# Test Cases — User login with email and password

**Story:** A registered user can sign in to the web app using their registered email address and password, so they can access their account dashboard. Authentication is handled via a POST to `/api/auth/login`.

---

**Title:** `[Positive] Login authenticates a registered user and redirects to the dashboard`
**Preconditions:**
- A user is registered with email `registered.user@example.com` and password `Correct-Horse-9!`
- The user is signed out and on the login page

**Steps:**
1. Type `registered.user@example.com` into the email field — the value appears in the field
2. Type `Correct-Horse-9!` into the password field — the characters render masked
3. Observe the "Sign in" button — it is enabled
4. Click "Sign in" — a POST to `/api/auth/login` is sent with the entered email and password
5. Wait for the response — the API returns a success status and an authenticated session is established
6. Observe the browser — the app navigates to `/dashboard` and the account dashboard renders

**Expected Result:** The user is authenticated and the URL is `/dashboard`. Pass condition: the browser is on `/dashboard` with an authenticated session after submitting valid credentials.
**Test Data:**
- email: `registered.user@example.com`
- password: `Correct-Horse-9!`
**Type:** Positive

---

**Title:** `[Positive] Sign in button enables once both email and password are non-empty`
**Preconditions:**
- The user is on the login page
- Both the email and password fields are empty

**Steps:**
1. Observe the "Sign in" button — it is disabled
2. Type `registered.user@example.com` into the email field — the "Sign in" button remains disabled (password still empty)
3. Type `Correct-Horse-9!` into the password field — the "Sign in" button becomes enabled
4. Clear the password field — the "Sign in" button becomes disabled again

**Expected Result:** The button is enabled only while both fields hold non-empty values. Pass condition: button state toggles disabled→enabled→disabled exactly in step with both fields being non-empty.
**Test Data:**
- email: `registered.user@example.com`
- password: `Correct-Horse-9!`
**Type:** Positive

---

**Title:** `[Positive] Login with "Remember me" checked authenticates and redirects to the dashboard`
**Preconditions:**
- A user is registered with email `registered.user@example.com` and password `Correct-Horse-9!`
- The user is signed out and on the login page

**Steps:**
1. Type `registered.user@example.com` into the email field — the value appears in the field
2. Type `Correct-Horse-9!` into the password field — the characters render masked
3. Click the "Remember me" checkbox — the checkbox becomes checked
4. Click "Sign in" — a POST to `/api/auth/login` is sent including the entered credentials
5. Wait for the response — the API returns a success status
6. Observe the browser — the app navigates to `/dashboard`

**Expected Result:** The user is authenticated and lands on `/dashboard` with "Remember me" selected during submission. Pass condition: the browser is on `/dashboard` after submitting valid credentials with "Remember me" checked.
**Test Data:**
- email: `registered.user@example.com`
- password: `Correct-Horse-9!`
- remember_me: `checked`
**Type:** Positive

---

**Title:** `[Negative] Login rejects a wrong password`
**Preconditions:**
- A user is registered with email `registered.user@example.com` and password `Correct-Horse-9!`
- The user is on the login page

**Steps:**
1. Type `registered.user@example.com` into the email field — the value appears in the field
2. Type `Wrong-Password-1!` into the password field — the characters render masked
3. Click "Sign in" — a POST to `/api/auth/login` is sent
4. Wait for the response — the API returns a non-success (authentication failure) status
5. Observe the page — the URL stays on the login page and an authentication error message is shown

**Expected Result:** The user is not authenticated and is not redirected. Pass condition: the browser remains on the login page and no authenticated session is created after submitting a wrong password.
**Test Data:**
- email: `registered.user@example.com`
- password: `Wrong-Password-1!`
**Type:** Negative

---

**Title:** `[Negative] Login rejects an unregistered email`
**Preconditions:**
- No user is registered with email `no.such.user@example.com`
- The user is on the login page

**Steps:**
1. Type `no.such.user@example.com` into the email field — the value appears in the field
2. Type `Correct-Horse-9!` into the password field — the characters render masked
3. Click "Sign in" — a POST to `/api/auth/login` is sent
4. Wait for the response — the API returns a non-success (authentication failure) status
5. Observe the page — the URL stays on the login page and an authentication error message is shown

**Expected Result:** The user is not authenticated and is not redirected. Pass condition: the browser remains on the login page and no authenticated session is created for an unregistered email.
**Test Data:**
- email: `no.such.user@example.com`
- password: `Correct-Horse-9!`
**Type:** Negative

---

**Title:** `[Negative] Sign in button stays disabled when only the email is filled`
**Preconditions:**
- The user is on the login page
- Both fields are empty

**Steps:**
1. Type `registered.user@example.com` into the email field — the value appears in the field
2. Leave the password field empty — the password field shows no value
3. Observe the "Sign in" button — it is disabled
4. Attempt to click "Sign in" — no POST to `/api/auth/login` is sent and the page does not navigate

**Expected Result:** Submission is blocked while the password is empty. Pass condition: the "Sign in" button is disabled and no login request is sent when only the email is provided.
**Test Data:**
- email: `registered.user@example.com`
- password: `(empty)`
**Type:** Negative

---

**Title:** `[Negative] Sign in button stays disabled when only the password is filled`
**Preconditions:**
- The user is on the login page
- Both fields are empty

**Steps:**
1. Leave the email field empty — the email field shows no value
2. Type `Correct-Horse-9!` into the password field — the characters render masked
3. Observe the "Sign in" button — it is disabled
4. Attempt to click "Sign in" — no POST to `/api/auth/login` is sent and the page does not navigate

**Expected Result:** Submission is blocked while the email is empty. Pass condition: the "Sign in" button is disabled and no login request is sent when only the password is provided.
**Test Data:**
- email: `(empty)`
- password: `Correct-Horse-9!`
**Type:** Negative

---

**Title:** `[Edge] Password field keeps input masked while typing and on re-focus`
**Preconditions:**
- The user is on the login page

**Steps:**
1. Type `Correct-Horse-9!` into the password field — each character displays as a mask character, not plain text
2. Click into the email field and type `registered.user@example.com` — the value appears in the email field
3. Click back into the password field — the previously entered password is still present and still rendered masked

**Expected Result:** The password value is never shown in plain text. Pass condition: the password field renders masked at every point during and after entry.
**Test Data:**
- email: `registered.user@example.com`
- password: `Correct-Horse-9!`
**Type:** Edge

---

**Title:** `[Edge] Login submits successfully when the email is entered with surrounding whitespace`
**Preconditions:**
- A user is registered with email `registered.user@example.com` and password `Correct-Horse-9!`
- The user is on the login page

**Steps:**
1. Type `  registered.user@example.com  ` (leading and trailing spaces) into the email field — the value including spaces appears in the field
2. Type `Correct-Horse-9!` into the password field — the characters render masked
3. Observe the "Sign in" button — it is enabled because both fields are non-empty
4. Click "Sign in" — a POST to `/api/auth/login` is sent
5. Wait for the response — the API returns a success status for the registered account
6. Observe the browser — the app navigates to `/dashboard`

**Expected Result:** The whitespace-padded email resolves to the registered account and the user reaches `/dashboard`. Pass condition: the browser is on `/dashboard` after submitting the padded email with the correct password.
**Test Data:**
- email: `"  registered.user@example.com  "`
- password: `Correct-Horse-9!`
**Type:** Edge

---

**Title:** `[Edge] Direct navigation to /dashboard while signed out does not grant access`
**Preconditions:**
- The user has no authenticated session
- The user has not submitted the login form

**Steps:**
1. Navigate the browser directly to `/dashboard` — the request is made without an authenticated session
2. Observe the result — the account dashboard content is not shown and the user is sent to the login page

**Expected Result:** The dashboard is reachable only after authentication. Pass condition: an unauthenticated visit to `/dashboard` does not render the dashboard and routes the user to login.
**Test Data:**
- session: `none`
- target_url: `/dashboard`
**Type:** Edge

---

**Title:** `[Boundary] Sign in button enables with a single-character value in each field`
**Preconditions:**
- The user is on the login page
- Both fields are empty

**Steps:**
1. Type `a` into the email field — one character appears in the field
2. Observe the "Sign in" button — it is still disabled (password empty)
3. Type `b` into the password field — one masked character appears in the field
4. Observe the "Sign in" button — it is enabled

**Expected Result:** The enablement rule treats any non-empty value (down to one character) as satisfying "non-empty". Pass condition: the "Sign in" button is enabled once each field contains at least one character.
**Test Data:**
- email: `a`
- password: `b`
**Type:** Boundary

---

**Title:** `[Performance] POST /api/auth/login responds within threshold under concurrent load`
**Preconditions:**
- A registered account exists with email `registered.user@example.com` and password `Correct-Horse-9!`
- A load tool can issue concurrent requests to `POST /api/auth/login`

**Steps:**
1. Send 100 concurrent `POST /api/auth/login` requests, each with body email `registered.user@example.com` and password `Correct-Horse-9!` — all requests are accepted by the endpoint
2. Record the response status and latency for every request — each request returns a success status
3. Compute the 95th percentile response time across all 100 requests — the value is produced

**Expected Result:** All requests authenticate successfully and stay responsive under load. Pass condition: 100% of the 100 concurrent requests return a success status and the 95th percentile response time is at or below 1000 ms.
**Test Data:**
- email: `registered.user@example.com`
- password: `Correct-Horse-9!`
- concurrent_requests: `100`
- p95_threshold_ms: `1000`
**Type:** Performance
