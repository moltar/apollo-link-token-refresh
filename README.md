# Token Refresh Link

## Purpose
An Apollo Link that performs renew expired JWT (access tokens)

## Installation

`npm install apollo-link-token-refresh --save`

## Usage
Token Refresh Link is `non-terminating` link, which means that this link shouldn't be the last link in the composed chain.

```js
import { TokenRefreshLink } from "apollo-link-token-refresh";

const link = new TokenRefreshLink({
  accessTokenField: 'accessToken',
  isTokenValidOrUndefined: () => boolean,
  fetchAccessToken: () => Promise<Response>,
  handleFetch: (accessToken: string) => void
  handleError: (err: Error) => void
});
```

## Options
Token Refresh Link takes an object with four options on it to customize the behavior of the link

|name|value|default|required|explanation|
|---|---|---|:---:|---|
|accessTokenField|string|'access_token'||This is a name of access token field in response
|isTokenValidOrUndefined|(...args: any[]) => boolean||✓|Indicates the current state of access token expiration. If token not yet expired or user doesn't have a token (guest) `true` should be returned|
|fetchAccessToken|(...args: any[]) => Promise<Response>||✓|Function covers fetch call with request fresh access token|
|handleFetch|(accessToken: string) => void||✓|Callback which receives a fresh token from Response. From here we can save token to the storage|
|handleError|(err: Error) => void|||Token fetch error callback. Allows to run additional actions like logout. Don't forget to handle Error if you are using this option|

## Example
```js
import { TokenRefreshLink } from 'apollo-link-token-refresh';

link: ApolloLink.from([
  new TokenRefreshLink({
    isTokenValidOrUndefined: () => !isTokenExpired() || typeof getAccessToken() !== 'string'
    fetchAccessToken: () => {
      return fetch(getEndpoint('getAccessTokenPath'), {
        method: 'GET',
        headers: {
          Authorization: `Bearer ${getAccessToken()}`,
          'refresh-token': getRefreshToken()
        }
      });
    }
    handleFetch: accessToken => {
      const accessTokenDecrypted = jwtDecode(accessToken);
      setAccessToken(accessToken);
      setExpiresIn(parseExp(accessTokenDecrypted.exp).toString());
    },
    handleError: err => {
    	// full control over handling token fetch Error
    	console.warn('Your refresh token is invalid. Try to relogin');
    	console.error(err);

    	// your custom action here
    	user.logout();
    }
  }),
  errorLink,
  requestLink,
  ...
])
```

## Context
The Token Refresh Link does not use the context for anything
