---
layout: post
title: Tracking UTM Parameters on your Web App
---

This post is for those who want to track their user's UTM parameters on their web app and save them in local storage for later use.

#### Prerequisites
- Understand what [UTM parameters](https://www.autopilothq.com/blog/what-are-utm-parameters/) are.
- Knowledge of local storage (and why we will use it instead of session storage). Recommended article [here](https://medium.com/better-programming/how-to-use-local-storage-with-javascript-9598834c8b72).

## Why do I need to do this?
Recently, I came across the need to do more than just track UTM params to google analytics. I wanted to save the UTM params to a specific user and when they completed a purchase, I would save these UTM params to the user's order as well as to their profiles across the board. The catch is that a user can always add items to their cart, leave the site, and then come back another day to complete their purchase. In this case, I wanted to be sure that their original UTM params would still be associated with that order.
Another edge case I wanted to catch was if a user came to the site via multiple different UTM params parameters, I wanted to save each one in the order they visited to capture all routes taken.

## The Solution
```javascript
const LOCAL_STORAGE = window.localStorage;
const DEFAULT_COOKIE_TTL = 10368000000;   // 120 days in ms
// The UTM parameters you want to track
const UTM_CAMPAIGN = 'utm_campaign';
const UTM_MEDIUM = 'utm_medium';
const UTM_SOURCE = 'utm_source';
export const UTM_PARAMS = [
  UTM_CAMPAIGN,
  UTM_MEDIUM,
  UTM_SOURCE
];

export class UtmTrackingService {
  /**
   * Private helper function to handle
   * setting the new UTM parameter's value
   * @param {String} utmParam
   * @param {String} utmValue
   */
  private setNewUTMCookie(utmParam, utmValue) {
    if (utmParam && utmValue) {
      const now = new Date();
      const utmCookie = {
        value: [utmValue],
        expiry: now.getTime() + DEFAULT_COOKIE_TTL
      };
      LOCAL_STORAGE.setItem(
        utmParam,
        JSON.stringify(utmCookie)
      );
    }
    return true;
  }

  /**
   * Private helper function to handle appending
   * a UTM value to an existing UTM parameter's cookie
   * @param {String} utmParam
   * @param {String} utmValue
   * @param {Object} utmCookie
   */
  private setExistingUTMCookie(utmParam, utmValue, utmCookie) {
    if (utmParam && utmValue && utmCookie) {
      if (!utmCookie.value.includes(utmValue)) {
        const now = new Date();
        utmCookie.value.push(utmValue);
        utmCookie.expiry = now.getTime() + DEFAULT_COOKIE_TTL;
        LOCAL_STORAGE.setItem(
          utmParam,
          JSON.stringify(utmCookie)
        );
      }
    }
    return true;
  }

  /**
   * Entry function to handle how
   * to store the UTM parameter found in the URL
   * @param utmParam
   * @param utmValue
   */
  private storeUTMParam(utmParam, utmValue) {
    if (utmParam && utmValue) {
      const rawUTMCookie = LOCAL_STORAGE.getItem(utmParam);
      // If no cookie exists, create a new one with expiration for 120 days
      if (!rawUTMCookie) {
        this.setNewUTMCookie(utmParam, utmValue);
      } else {
        const utmCookie = JSON.parse(rawUTMCookie);
        const now = new Date();
        // If cookie has expired, remove it and set a new one with default expiration
        if (now.getTime() > utmCookie.expiry) {
          LOCAL_STORAGE.removeItem(utmParam);
          this.setNewUTMCookie(utmParam, utmValue);
        } else {
          // Cookie has not expire and exists, so append the new utm value
          this.setExistingUTMCookie(utmParam, utmValue, utmCookie);
        }
      }
    }
  }

  /**
   * Search the array of parameters for the specified UTM parameter
   * @param {String} utmParam
   * @param {Array} urlParams
   */
  private getUTMParamFromURL(utmParam, urlParams = []) {
    if (utmParam) {
      for (let i = 0; i < urlParams.length; i++) {
        const param = urlParams[i];
        const keyValue = param.split('=');
        if (keyValue[0] === utmParam && keyValue[1]) {
          return decodeURIComponent(keyValue[1]);
        }
      }
    }
  }

  /**
   * Trigger to check the current URL for the UTM params
   */
  public checkForUTMParams() {
    const URLParams = window.location.search.substr(1).split('&');
    UTM_PARAMS.forEach((utmParam) => {
      const utmValue = this.getUTMParamFromURL(utmParam, URLParams);
      if (utmValue) {
        this.storeUTMParam(utmParam, utmValue);
      }
    });
  }

  /**
   * If the given parameter is valid, attempt to
   * retrieve the UTM value from local storage
   * (If the cookie has expired, delete it and return null)
   * @param {String} param
   */
  public getUTMParamFromCookie(param) {
    if (param && UTM_PARAMS.includes(param)) {
      const rawUTMCookie = LOCAL_STORAGE.getItem(param);
      if (!rawUTMCookie) {
        return null;
      } else {
        const now = new Date();
        const utmCookie = JSON.parse(rawUTMCookie);
        // If the cookie is expired, delete it and return null
        if (now.getTime() > utmCookie.expiry) {
          LOCAL_STORAGE.removeItem(param);
          return null;
        } else {
          return utmCookie.value;
        }
      }
    } else {
      return null;
    }
  }
}
```

## Breakdown
Let's step through how each function above connects to each other.

### checkForUTMParams

```javascript
/**
 * Trigger to check the current URL for the UTM params
 */
public checkForUTMParams() {
  const URLParams = window.location.search.substr(1).split('&');
  UTM_PARAMS.forEach((utmParam) => {
    const utmValue = this.getUTMParamFromURL(utmParam, URLParams);
    if (utmValue) {
      this.storeUTMParam(utmParam, utmValue);
    }
  });
}
```

This is our entry point that kicks off everything else. This is the function you want to put
 in the root of your application so it's triggered when the app is initialized.
 Here we need to grab the current URL before the user can navigate to another page
 and our search parameters are lost. Then for each of our constant UTM parameters
 we defined at the top of this file, we want to check for in the existing URL.
 This is done via the `getUTMParamFromURL` function inside our loop.

### getUTMParamFromURL

```javascript
/**
 * Search the array of parameters for the specified UTM parameter
 * @param {String} utmParam
 * @param {Array} urlParams
 */
private getUTMParamFromURL(utmParam, urlParams = []) {
  if (utmParam) {
    for (let i = 0; i < urlParams.length; i++) {
      const param = urlParams[i];
      const keyValue = param.split('=');
      if (keyValue[0] === utmParam && keyValue[1]) {
        return decodeURIComponent(keyValue[1]);
      }
    }
  }
}
```
For each given UTM parameter and the array of URL parameters, we split up the
query parameter and try to match on our UTM. If we find a match, we decode
and return that value to our entry function above. If the `utmValue` is a truthy
value, we will then store this value so use later!

### storeUTMParam
```javascript
/**
 * Entry function to handle how
 * to store the UTM parameter found in the URL
 * @param utmParam
 * @param utmValue
 */
private storeUTMParam(utmParam, utmValue) {
  if (utmParam && utmValue) {
    const rawUTMCookie = LOCAL_STORAGE.getItem(utmParam);
    // If no cookie exists, create a new one with expiration for 120 days
    if (!rawUTMCookie) {
      this.setNewUTMCookie(utmParam, utmValue);
    } else {
      const utmCookie = JSON.parse(rawUTMCookie);
      const now = new Date();
      // If cookie has expired, remove it and set a new one with default expiration
      if (now.getTime() > utmCookie.expiry) {
        LOCAL_STORAGE.removeItem(utmParam);
        this.setNewUTMCookie(utmParam, utmValue);
      } else {
        // Cookie has not expire and exists, so append the new utm value
        this.setExistingUTMCookie(utmParam, utmValue, utmCookie);
      }
    }
  }
}
```
This is where things get a bit tricky. Before we store our UTM value, we
need to first check to see if a value is already stored in an existing cookie.
To do so, we check our local storage for our UTM parameter key (i.e. `utm_source`).
If the cookie does not exist, we hand off the UTM value to be added as
a new cookie. Otherwise, we need to check the cookie's expiration date
(which I set to be 120 days from today's date. You can set this to whatever is appropriate
to your application). If the cookie expired, we should delete the existing value
and then add our UTM as a new cookie again. The last case is where our cookie has not
expired and we handoff our existing UTM cookie with our new parameters to handle merging
the two together.

### setNewUTMCookie
```javascript
/**
 * Private helper function to handle
 * setting the new UTM parameter's value
 * @param {String} utmParam
 * @param {String} utmValue
 */
private setNewUTMCookie(utmParam, utmValue) {
  if (utmParam && utmValue) {
    const now = new Date();
    const utmCookie = {
      value: [utmValue],
      expiry: now.getTime() + DEFAULT_COOKIE_TTL
    };
    LOCAL_STORAGE.setItem(
      utmParam,
      JSON.stringify(utmCookie)
    );
  }
  return true;
}
```
This function is where the magic happens and where we finally save our UTM value
in our cookie. Our value is an array of strings because in the future, we will
want to append new values under the existing cookie if a user comes back to visit from another source
(assuming the cookie has not expired yet). Which leads to the second key in the object. Our expiration
is set for today's date plus 120 days. We then properly format our cookie object and
store it in local storage under our UTM parameter value.

### setExistingUTMCookie
```javascript
/**
 * Private helper function to handle appending
 * a UTM value to an existing UTM parameter's cookie
 * @param {String} utmParam
 * @param {String} utmValue
 * @param {Object} utmCookie
 */
private setExistingUTMCookie(utmParam, utmValue, utmCookie) {
  if (utmParam && utmValue && utmCookie) {
    if (!utmCookie.value.includes(utmValue)) {
      const now = new Date();
      utmCookie.value.push(utmValue);
      utmCookie.expiry = now.getTime() + DEFAULT_COOKIE_TTL;
      LOCAL_STORAGE.setItem(
        utmParam,
        JSON.stringify(utmCookie)
      );
    }
  }
  return true;
}
```
Given that a UTM parameter is already in local storage, we first check to see
if the given string value exists in our value array (since we only want to
append new sources). If it doesn't, we simply push the string onto our array
and then update our expiration time.


## Congratulations! 🎉
You can always test and see if your implementation is working as expected by opening the browser dev console and typing `window.localStorage` and you should see your UTM parameters successfully stored inside the object. Otherwise, you can use the `getUTMParamFromCookie` at the bottom of the solution above to retrieve a valid UTM parameter from local storage and use in other parts of your application.
