# Challenge I - Walkthrough

## Step 1 - Reviewing the Code

The first challenge involves inspecting the `.js` file of the page, where we notice suspicious strings. By quickly checking their structure and seeing they end in `=`, we recognize that they are Base64-encoded strings.

For fun, let's decode the first one:

```bash
echo "SGEgaGEhIEJ1ZW4gaW50ZW50byBwZXJvIG5vIGVzIGFzw60uIDxicj48YnI+RXN0ZSByZXRvIGVzIGEgcHJ1ZWJhIGRlIGhhY2tlcnMuIDxicj4g8J+SqiBObyB0ZSByaW5kYXMsIHNpZ3VlIGludGVudMOhbmRvbG8uIPCfkqo=" | base64 -d

Ha ha! Nice try, but that's not how it works. <br><br>This challenge is hacker-proof. <br> 💪 Don't give up, keep trying. 💪
```

We also find the following variables in the JavaScript code:

```javascript
x = "aGFja2F0b24";
L = "QXpjbDl6ZE";
_ = "RSeVgxOD0";
v = "WDE5emRY";
```

Along with a key line of code:

```javascript
x = btoa(v + L + _);
```

## Step 2 - Decoding the Strings

We proceed to decode the concatenated values:

```bash
echo "WDE5emRYQXpjbDl6ZERSeVgxOD0" | base64 -d

X19zdXAzcl9zdDRyX18=base64: invalid input
```

We extract the key string: `X19zdXAzcl9zdDRyX18=` (it was padded with `=` to complete the block).

Now, we execute the **Request** as indicated in the JavaScript code:

```javascript
// Extracted from the original script
b = e => {
    r.innerHTML = "",
    p.innerHTML = "",
    s.classList.remove("hidden");
    let o = e ? `option=${e}` : null;
    return fetch("/__challenge__01__/api/", {
        method: "POST",
        headers: {
            "Content-Type": "application/x-www-form-urlencoded",
            "X-CSRFToken": I.value
        },
        body: o
    }).then(a => a.json()).finally(() => {
        s.classList.add("hidden")
    })
}
```

## Step 3 - Executing the Request

We send the request with the extracted value:

```bash
curl -X POST "https://hackaton.cyberh2o.es/__challenge__01__/api/" \
     -H "Content-Type: application/x-www-form-urlencoded" \
     -H "X-CSRFToken: YOUR_CSRF_TOKEN" \
     -H "Cookie: csrftoken=YOUR_CSRF_COOKIE; sessionid=YOUR_SESSION_ID" \
     --data "option=X19zdXAzcl9zdDRyX18="
```

Response:

```json
{"text": "V29XLCBoYXMgc3VwZXJhZG8gZWwgZGVzYWbDrW8uIFRlIHZveSBhIHByZW1pYXIgY29uIHVuYSBiYW5kZXJhLiDwn5qpIDxicj48YnI+IEhBQ0t7NCRzdXBlciRzdGFyISF9", "options": []}
```

## Step 4 - Decoding the Response

We decode the returned `text`:

```bash
echo "V29XLCBoYXMgc3VwZXJhZG8gZWwgZGVzYWbDrW8uIFRlIHZveSBhIHByZW1pYXIgY29uIHVuYSBiYW5kZXJhLiDwn5qpIDxicj48YnI+IEhBQ0t7NCRzdXBlciRzdGFyISF9" | base64 -d

WoW, you have completed the challenge. I will reward you with a flag. 🚩 <br><br> HACK{4$super$star!!}
```

## Final Result

`HACK{4$super$star!!}`


