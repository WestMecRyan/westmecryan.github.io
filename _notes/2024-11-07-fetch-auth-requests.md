---
title: "API Auth Requests"
date: 2024-11-07
categories: [JavaScript, Basics, API]
keyterms: ["authentication", "jwt", "bearer", "token", "btoa", "headers" ]
---

```javascript
// Basic Authentication Example
async function makeBasicAuthRequest(username, password) {
  // Create Base64 encoded string of username:password
  const credentials = btoa(`${username}:${password}`);

  try {
    const response = await fetch('https://api.example.com/data', {
      headers: {
        'Authorization': `Basic ${credentials}`,
        'Content-Type': 'application/json'
      }
    });

    if (!response.ok) {
      throw new Error('Authentication failed');
    }

    return await response.json();
  } catch (error) {
    console.error('Error:', error);
    throw error;
  }
}

// Bearer Token Authentication Example
class APIClient {
  constructor() {
    this.token = null;
  }

  async login(username, password) {
    try {
      const response = await fetch('https://api.example.com/login', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify({ username, password })
      });

      if (!response.ok) {
        throw new Error('Login failed');
      }

      // Store the token received from server
      const data = await response.json();
      this.token = data.token;

      // Optional: Store in localStorage for persistence
      localStorage.setItem('authToken', this.token);
    } catch (error) {
      console.error('Login error:', error);
      throw error;
    }
  }

  async makeAuthenticatedRequest(endpoint, options = {}) {
    if (!this.token) {
      throw new Error('Not authenticated');
    }

    try {
      const response = await fetch(`https://api.example.com${endpoint}`, {
        ...options,
        headers: {
          ...options.headers,
          'Authorization': `Bearer ${this.token}`,
          'Content-Type': 'application/json'
        }
      });

      if (response.status === 401) {
        // Token might be expired
        this.token = null;
        localStorage.removeItem('authToken');
        throw new Error('Token expired');
      }

      return await response.json();
    } catch (error) {
      console.error('Request error:', error);
      throw error;
    }
  }

  // Example usage with token refresh
  async refreshToken() {
    try {
      const response = await fetch('https://api.example.com/refresh', {
        method: 'POST',
        headers: {
          'Authorization': `Bearer ${this.token}`,
          'Content-Type': 'application/json'
        }
      });

      if (!response.ok) {
        throw new Error('Token refresh failed');
      }

      const data = await response.json();
      this.token = data.newToken;
      localStorage.setItem('authToken', this.token);
    } catch (error) {
      console.error('Refresh error:', error);
      throw error;
    }
  }
}

// Usage example
const api = new APIClient();

async function example() {
  try {
    // Login and get token
    await api.login('username', 'password');

    // Make authenticated requests
    const userData = await api.makeAuthenticatedRequest('/user/profile');
    console.log('User data:', userData);

    // Handle token refresh when needed
    try {
      const protectedData = await api.makeAuthenticatedRequest('/protected-endpoint');
    } catch (error) {
      if (error.message === 'Token expired') {
        await api.refreshToken();
        // Retry the request with new token
        const protectedData = await api.makeAuthenticatedRequest('/protected-endpoint');
      }
    }
  } catch (error) {
    console.error('Error in authentication flow:', error);
  }
}

```

Let me break down the key differences between Basic and Bearer authentication:

**Basic Authentication:**
- Simple username/password authentication
- Credentials are Base64 encoded (not encrypted!)
- Sent with every request
- Format: `Authorization: Basic base64(username:password)`
- Less secure because:
  - Credentials sent with every request
  - Base64 encoding can be easily decoded
  - No expiration mechanism
  - Vulnerable to man-in-the-middle attacks if not using HTTPS

**Bearer Authentication:**
- Token-based authentication
- Usually implements JWT (JSON Web Tokens)
- Format: `Authorization: Bearer <token>`
- More secure because:
  - Credentials only sent once during login
  - Tokens can expire
  - Can include user roles and permissions in token
  - Can be invalidated server-side
  - Stateless - server doesn't need to store session info

**Why Bearer is Better for Modern JavaScript Applications:**

1. **Single Page Applications (SPAs)**
   - Can store token in memory or localStorage
   - No need to store sensitive credentials
   - Easy to handle token expiration

2. **API Security**
   - Tokens can be scoped to specific permissions
   - Can implement refresh token flow
   - Better protection against CSRF attacks

3. **Performance**
   - Stateless - reduces server load
   - Can validate tokens without database lookups
   - Easy to scale across multiple servers

4. **JSON Integration**
   - JWTs are JSON-based
   - Easy to decode and use in JavaScript
   - Can include custom claims in token payload