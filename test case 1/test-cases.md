**Title:** `[Positive] Sign in button enables only when both email and password are non-empty`
**Preconditions:**
- The login page is loaded
- The email field and password field are both empty
- The "Sign in" button is disabled
**Steps:**
1. Type `registered.user@example.com` into the email field
2. Observe the "Sign in" button state
3. Type `Correct!Pass123` into the password field
4. Observe the "Sign in" button state
**Expected Result:**
- After step 2, with only the email field filled, the "Sign in" button is still disabled
- After step 4, with both fields non-empty, the "Sign in" button is enabled
- Pass condition: the "Sign in" button is enabled if and only if both email and password are non-empty
**Test Data:**
- email: registered.user@example.com
- password: Correct!Pass123
**Type:** Positive

---

**Title:** `[Positive] Login authenticates a registered user and redirects to /dashboard`
**Preconditions:**
- A registered user exists with email `registered.user@example.com` and password `Correct!Pass123`
- The login page is loaded
**Steps:**
1. Type `registered.user@example.com` into the email field
2. Type `Correct!Pass123` into the password field
3. Leave the "Remember me" checkbox unchecked
4. Click the "Sign in" button
5. Observe the network request `POST /api/auth/login`
6. Observe the browser URL after the response
**Expected Result:**
- Step 4 sends `POST /api/auth/login` carrying the entered email and password
- The request returns a success response and the user session is authenticated
- The browser navigates to `/dashboard`
- Pass condition: the browser URL is `/dashboard` and the account dashboard is shown
**Test Data:**
- email: registered.user@example.com
- password: Correct!Pass123
- Remember me: unchecked
**Type:** Positive

---

**Title:** `[Negative] Login rejects a registered email with a wrong password`
**Preconditions:**
- A registered user exists with email `registered.user@example.com` and password `Correct!Pass123`
- The login page is loaded
**Steps:**
1. Type `registered.user@example.com` into the email field
2. Type `WrongPass999` into the password field
3. Click the "Sign in" button
4. Observe the `POST /api/auth/login` response
5. Observe the browser URL
**Expected Result:**
- `POST /api/auth/login` returns a non-success authentication-failure response
- The user session is not authenticated
- The browser stays on the login page and does not navigate to `/dashboard`
- Pass condition: no redirect to `/dashboard` occurs and the user remains unauthenticated
**Test Data:**
- email: registered.user@example.com
- password: WrongPass999
**Type:** Negative

---

**Title:** `[Negative] Login rejects an unregistered email address`
**Preconditions:**
- No registered user exists with email `no.such.user@example.com`
- The login page is loaded
**Steps:**
1. Type `no.such.user@example.com` into the email field
2. Type `Correct!Pass123` into the password field
3. Click the "Sign in" button
4. Observe the `POST /api/auth/login` response
5. Observe the browser URL
**Expected Result:**
- `POST /api/auth/login` returns a non-success authentication-failure response
- The user session is not authenticated
- The browser does not navigate to `/dashboard`
- Pass condition: login is denied and the browser URL is not `/dashboard`
**Test Data:**
- email: no.such.user@example.com
- password: Correct!Pass123
**Type:** Negative

---

**Title:** `[Negative] Sign in button stays disabled and no request is sent when the password field is empty`
**Preconditions:**
- The login page is loaded
- Both fields are empty and the "Sign in" button is disabled
**Steps:**
1. Type `registered.user@example.com` into the email field
2. Leave the password field empty
3. Attempt to click the "Sign in" button
**Expected Result:**
- After step 1, the "Sign in" button remains disabled
- The click in step 3 has no effect and no `POST /api/auth/login` request is sent
- Pass condition: no login request is sent while the password field is empty
**Test Data:**
- email: registered.user@example.com
- password: (empty)
**Type:** Negative

---

**Title:** `[Edge] Login succeeds and redirects to /dashboard with "Remember me" checked`
**Preconditions:**
- A registered user exists with email `registered.user@example.com` and password `Correct!Pass123`
- The login page is loaded
**Steps:**
1. Type `registered.user@example.com` into the email field
2. Type `Correct!Pass123` into the password field
3. Check the "Remember me" checkbox
4. Click the "Sign in" button
5. Observe the `POST /api/auth/login` response and the browser URL
**Expected Result:**
- The "Remember me" checkbox reads as checked before submission
- `POST /api/auth/login` returns a success response and the user session is authenticated
- The browser navigates to `/dashboard`
- Pass condition: the browser URL is `/dashboard` with "Remember me" checked at submission
**Test Data:**
- email: registered.user@example.com
- password: Correct!Pass123
- Remember me: checked
**Type:** Edge

---

**Title:** `[Edge] Sign in button re-disables when a previously filled field is cleared`
**Preconditions:**
- The login page is loaded
- The email field contains `registered.user@example.com` and the password field contains `Correct!Pass123`
- The "Sign in" button is enabled
**Steps:**
1. Select all text in the email field and delete it, leaving the field empty
2. Observe the "Sign in" button state
3. Type `registered.user@example.com` back into the email field
4. Observe the "Sign in" button state
**Expected Result:**
- After step 1, the email field is empty and the "Sign in" button becomes disabled
- After step 3, both fields are non-empty again and the "Sign in" button becomes enabled
- Pass condition: the button's enabled state follows the "both fields non-empty" rule as the fields change
**Test Data:**
- email: registered.user@example.com
- password: Correct!Pass123
**Type:** Edge

---

**Title:** `[Boundary] Sign in button toggles at the one-character minimum for each field`
**Preconditions:**
- The login page is loaded
- Both fields are empty and the "Sign in" button is disabled
**Steps:**
1. Type the single character `a` into the email field
2. Type the single character `x` into the password field
3. Observe the "Sign in" button state
4. Delete the single character from the password field, leaving it empty
5. Observe the "Sign in" button state
**Expected Result:**
- After step 2, with exactly one character in each field, the "Sign in" button is enabled
- After step 4, with zero characters in the password field, the "Sign in" button is disabled
- Pass condition: one character satisfies "non-empty" and zero characters does not
**Test Data:**
- email: a
- password: x
**Type:** Boundary

---

**Title:** `[Performance] POST /api/auth/login responds within 2 seconds under concurrent load`
**Preconditions:**
- A registered user exists with email `registered.user@example.com` and password `Correct!Pass123`
- The authentication service is running and reachable
**Steps:**
1. Configure a load test to send `POST /api/auth/login` with body `{ "email": "registered.user@example.com", "password": "Correct!Pass123" }`
2. Run 100 requests per second for 60 seconds
3. Record the response status code and response time for every request
4. Compute the 95th-percentile response time and the error rate over the run
**Expected Result:**
- Every request returns a success authentication response
- The 95th-percentile response time is at or below 2000 ms
- The error rate is 0%
- Pass condition: p95 latency ≤ 2000 ms with a 0% error rate across the 60-second run
**Test Data:**
- endpoint: POST /api/auth/login
- email: registered.user@example.com
- password: Correct!Pass123
- load: 100 requests/second for 60 seconds
**Type:** Performance
