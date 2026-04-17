# API Refactor Explanation: From fetch to Axios

## 1. What Was Changed

a. **Switched from fetch to axios**
- **Before:** The class used the native fetch API to send HTTP requests to the n8n backend.
- **Now:** It uses an axios instance, which is a popular HTTP client library for Node.js and browsers.

b. **Axios Instance Initialization**
- On class construction, an axiosInstance is created and configured with:
  - `baseURL`: The n8n API base URL from config.
  - `headers`: Includes X-N8N-API-KEY and Accept: application/json.
  - `timeout`: Set from config or defaults to 10 seconds.

c. **Debug Logging (Optional)**
- If the config has `debug: true`, Axios request and response interceptors are added.
- These interceptors log every outgoing request and incoming response to the console for debugging.

d. **Request Logic Refactor**
- The `makeRequest` method now uses `this.axiosInstance.request()` instead of fetch.
- It maps the previous options (like method, body, headers) to Axios’s request config.
- Handles errors using Axios’s error structure, providing clear error messages if the API call fails.

## 2. Change in API Structure

a. **Before (with fetch)**
- Each request was made by calling fetch(url, options), manually setting headers and timeout.
- The API key and endpoint were hardcoded in the request.
- Response handling was manual, including JSON parsing and error checking.

b. **Now (with axios)**
- All requests go through a pre-configured Axios instance, so:
  - The base URL and API key are set once at initialization.
  - Headers like Accept and X-N8N-API-KEY are always included automatically.
  - Timeout is managed by Axios.
- The `makeRequest` method now:
  - Accepts the endpoint and options, mapping them to Axios’s config (`url`, `method`, `data`, `params`, `headers`).
  - Parses the response and returns `response.data` directly.
