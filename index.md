The provided HTML and JavaScript code implements a client-side OTP (One-Time Password) service. It's designed as a single-page application that dynamically shows and hides elements based on the OTP process flow (request, verify, resend, cancel).

Here's a breakdown of the code, segmented into logical parts for explanation in a Markdown document.

-----

# OTP Authorization Service Documentation

This document explains the client-side implementation of an OTP (One-Time Password) authorization service. The provided code combines the functionalities for requesting, verifying, resending, and invalidating OTPs within a single HTML page using JavaScript.

## Service Flow

The OTP service follows a simple flow:

1.  **Request OTP**: The user enters their phone number and requests an OTP.
2.  **Verify OTP**: If the OTP request is successful, a form appears for the user to enter the received OTP.
3.  **Resend OTP**: If the user doesn't receive the OTP or it expires, they can request a resend.
4.  **Cancel OTP**: The user can cancel the current OTP process.
5.  **Redirection**:
      * On successful OTP verification, the user is redirected to a success page.
      * On failure (e.g., incorrect OTP, API error), an error message is displayed, and the user can attempt to re-verify or resend.

## Core Components

The service is encapsulated within an `OtpClient` IIFE (Immediately Invoked Function Expression) in JavaScript, which manages the UI and API interactions.

### 1\. HTML Structure (Base Page)

This is the foundational HTML structure for the OTP service. It includes the necessary CSS for styling and a placeholder `div` where the OTP functionalities will be dynamically rendered.

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>OTP Authorization</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            margin: 0;
            background-color: #f0f2f5;
        }

        #otp-container {
            background: white;
            padding: 20px;
            border-radius: 8px;
            box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
            text-align: center;
            max-width: 400px;
            width: 100%;
        }

        .otp-button {
            background-color: #1a73e8;
            color: white;
            padding: 10px 20px;
            border: none;
            border-radius: 4px;
            cursor: pointer;
            font-size: 16px;
            margin: 5px;
        }

        .otp-button:hover {
            background-color: #1557b0;
        }

        .otp-form {
            margin-top: 10px;
            display: none; /* Hidden by default */
        }

        .otp-input {
            padding: 8px;
            font-size: 16px;
            margin: 5px;
            width: 150px;
            border: 1px solid #ccc;
            border-radius: 4px;
        }

        .otp-error,
        .otp-success,
        .otp-loading {
            margin-top: 10px;
            font-size: 14px;
        }

        .otp-error {
            color: red;
        }

        .otp-success {
            color: green;
        }

        .otp-loading {
            color: gray;
        }

        .phone-input {
            padding: 8px;
            font-size: 16px;
            margin: 5px;
            width: 200px;
            border: 1px solid #ccc;
            border-radius: 4px;
        }
    </style>
</head>

<body>
    <div id="otp-container">
        <input type="text" class="phone-input" id="phone-input" placeholder="Enter phone number">
    </div>
    </body>

</html>
```

### 2\. OTP Client JavaScript Module (`OtpClient`)

This module contains all the logic for interacting with the OTP API and managing the UI elements.

#### Configuration

The `OtpClient` starts with a default configuration that can be overridden during initialization.

```javascript
const OtpClient = (function () {
    // Default configuration
    const defaultConfig = {
        apiBaseUrl: 'https://heading-to-paris-op.briq.tz', // Default API base URL
        appKey: 'briq_tq8oyuxed2b7p95y', // Default application key
        senderId: 'BRIQ', // Default sender ID for SMS
        otpLength: 6, // Default OTP length
        minutesToExpire: 10, // Default OTP expiry time
        deliveryMethod: 'sms', // Default delivery method
        environment: 'TEST' // Default environment
    };

    // Merges user-provided configuration with default settings
    function initialize(config) {
        return { ...defaultConfig, ...config };
    }

    // ... rest of the OtpClient code
})();
```

#### API Call Wrapper

A utility function `apiCall` handles all interactions with the backend API, including error handling.

```javascript
// Inside OtpClient...
async function apiCall(endpoint, payload) {
    try {
        const response = await fetch(`${apiBaseUrl}${endpoint}`, {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify(payload)
        });
        const data = await response.json();
        // Check for HTTP errors or API-specific success flags
        if (!response.ok || !data.success) throw new Error(data.message || 'API request failed');
        return data;
    } catch (error) {
        throw new Error(error.message || 'Network error');
    }
}
```

#### Message Display Utility

A helper function `showMessage` updates the UI with status messages (loading, success, error).

```javascript
// Inside OtpClient...
function showMessage(text, type) {
    const messageDiv = document.querySelector('.otp-message'); // Assumes messageDiv is defined within createOtpButton scope
    if (messageDiv) {
        messageDiv.textContent = text;
        messageDiv.className = `otp-message otp-${type}`;
    }
}
```

### 3\. Snippets for OTP Functionalities

The `createOtpButton` function dynamically creates the necessary UI elements (request button, OTP input form) and attaches event listeners for all OTP operations.

#### A. Send OTP (Request) Functionality

This snippet handles the initial request for an OTP. It's triggered when the "Request OTP" button is clicked.

**Code Snippet (`requestOtp` function and associated event listener):**

```javascript
// Inside OtpClient.createOtpButton function...

// Request OTP API call
async function requestOtp(phoneNumber) {
    if (!phoneNumber) {
        showMessage('Phone number is required', 'error');
        return;
    }
    showMessage('Requesting OTP...', 'loading');
    try {
        const data = await apiCall('/otp/developer-app/request', {
            phone_number: phoneNumber,
            app_key: appKey,
            sender_id: senderId,
            otp_length: otpLength,
            minutes_to_expire: minutesToExpire,
            delivery_method: deliveryMethod,
            message_template: messageTemplate
        });
        showMessage('OTP sent successfully', 'success');
        form.style.display = 'block'; // Show the OTP verification form
        button.style.display = 'none'; // Hide the request OTP button
        phoneInput.style.display = 'none'; // Hide the phone input
        otpInput.dataset.phoneNumber = phoneNumber; // Store phone number for subsequent operations
        if (onRequestSuccess) onRequestSuccess(data); // Callback on success
    } catch (error) {
        showMessage(error.message || 'Failed to request OTP', 'error');
        if (onError) onError(error); // Callback on error
    }
}

// Event Listener for Request OTP button
button.addEventListener('click', () => {
    const phoneNumber = phoneInput.value.trim();
    if (phoneNumber) {
        requestOtp(phoneNumber);
    }
});
```

**Placement on Page:** This snippet should be part of the main JavaScript block that initializes the `OtpClient`. The `phone-input` element and the dynamically created `button` (for requesting OTP) would be present on the initial "send OTP" page.

-----

#### B. Verify OTP Functionality

This snippet handles the verification of the entered OTP against the backend. It is displayed after a successful OTP request.

**Code Snippet (`verifyOtp` function and associated event listener):**

```javascript
// Inside OtpClient.createOtpButton function...

// Verify OTP API call
async function verifyOtp(phoneNumber, code) {
    if (!code) {
        showMessage('OTP code is required', 'error');
        return;
    }
    showMessage('Verifying OTP...', 'loading');
    try {
        const data = await apiCall('/otp/developer-app/verify', {
            phone_number: phoneNumber,
            app_key: appKey,
            code // The OTP code to verify
        });
        showMessage('OTP verified successfully', 'success');
        form.style.display = 'none'; // Hide the OTP form
        button.style.display = 'block'; // Show the request OTP button again
        phoneInput.style.display = 'block'; // Show the phone input again
        phoneInput.value = ''; // Clear phone input
        otpInput.value = ''; // Clear OTP input
        if (onVerifySuccess) onVerifySuccess(data); // Callback on success
    } catch (error) {
        showMessage(error.message || 'OTP verification failed', 'error');
        if (onError) onError(error); // Callback on error
    }
}

// Event Listener for Verify OTP button
verifyButton.addEventListener('click', () => {
    const phoneNumber = otpInput.dataset.phoneNumber; // Retrieve stored phone number
    const code = otpInput.value.trim();
    verifyOtp(phoneNumber, code);
});
```

**Placement on Page:** This snippet is part of the JavaScript that gets executed when the `OtpClient` is initialized. The `otp-input` and `verify-button` are dynamically created and become visible on the "verify OTP" page, which is essentially the same page but with different UI elements shown. The `onVerifySuccess` callback is crucial here for redirection.

-----

#### C. Resend OTP Functionality

This snippet allows the user to request a new OTP if the previous one was not received or expired.

**Code Snippet (`resendOtp` function and associated event listener):**

```javascript
// Inside OtpClient.createOtpButton function...

// Resend OTP API call
async function resendOtp(phoneNumber) {
    if (!phoneNumber) {
        showMessage('Phone number is required', 'error');
        return;
    }
    showMessage('Resending OTP...', 'loading');
    try {
        const data = await apiCall('/otp/developer-app/resend', {
            phone_number: phoneNumber,
            app_key: appKey,
            sender_id: senderId,
            otp_length: otpLength,
            minutes_to_expire: minutesToExpire,
            delivery_method: deliveryMethod,
            message_template: messageTemplate
        });
        showMessage('OTP resent successfully', 'success');
        if (onRequestSuccess) onRequestSuccess(data); // Callback on success (same as request)
    } catch (error) {
        showMessage(error.message || 'Failed to resend OTP', 'error');
        if (onError) onError(error); // Callback on error
    }
}

// Event Listener for Resend OTP button
resendButton.addEventListener('click', () => {
    const phoneNumber = otpInput.dataset.phoneNumber; // Retrieve stored phone number
    resendOtp(phoneNumber);
});
```

**Placement on Page:** This snippet is part of the JavaScript that is active when the OTP verification form is displayed. The `resend-button` is part of the dynamically created form on the "verify OTP" page. On failure of verification, the user can click this to receive a new OTP without leaving the page.

-----

#### D. Invalidate (Cancel) OTP Functionality

This snippet allows the user to cancel an active OTP session.

**Code Snippet (`invalidateOtp` function and associated event listener):**

```javascript
// Inside OtpClient.createOtpButton function...

// Invalidate OTP API call
async function invalidateOtp(phoneNumber) {
    if (!phoneNumber) {
        showMessage('Phone number is required', 'error');
        return;
    }
    showMessage('Canceling OTP...', 'loading');
    try {
        const data = await apiCall('/otp/developer-app/invalidate', {
            phone_number: phoneNumber,
            app_key: appKey
        });
        showMessage('OTP canceled', 'success');
        form.style.display = 'none'; // Hide the OTP form
        button.style.display = 'block'; // Show the request OTP button again
        phoneInput.style.display = 'block'; // Show the phone input again
        phoneInput.value = ''; // Clear phone input
        otpInput.value = ''; // Clear OTP input
        if (onRequestSuccess) onRequestSuccess(data); // Callback on success (can be customized)
    } catch (error) {
        showMessage(error.message || 'Failed to cancel OTP', 'error');
        if (onError) onError(error); // Callback on error
    }
}

// Event Listener for Cancel button
cancelButton.addEventListener('click', () => {
    const phoneNumber = otpInput.dataset.phoneNumber; // Retrieve stored phone number
    invalidateOtp(phoneNumber);
});
```

**Placement on Page:** This snippet is also part of the JavaScript active when the OTP verification form is displayed. The `cancel-button` is part of the dynamically created form on the "verify OTP" page.

-----

### 4\. Initialization and Callbacks

This is the final part of the script, where the `OtpClient` is initialized with specific configurations and callback functions.

```javascript
// After the OtpClient IIFE definition...

// Initialize OTP client with custom configuration and callbacks
const otpClient = OtpClient.createOtpButton({
    apiBaseUrl: 'http://143.198.159.135:8000', // Override API base URL
    appKey: 'briq_tq8oyuxed2b7p95y', // Your specific app key
    senderId: 'BRIQ',
    otpLength: 6,
    minutesToExpire: 10,
    deliveryMethod: 'sms',
    messageTemplate: 'Your OTP code is {code}. It will expire in {minutes} minutes.'
}, {
    onRequestSuccess: (data) => console.log('OTP Request Success:', data),
    onVerifySuccess: (data) => {
        console.log('OTP Verify Success:', data);
        window.location.href = '/success'; // Redirect to a success page
    },
    onError: (error) => console.error('OTP Error:', error)
});
```

**Placement on Page:** This snippet should be placed at the very end of the `<body>` tag, within a `<script>` block, after the `OtpClient` definition. This ensures that the HTML elements are available in the DOM before the script tries to access them.

## Page Redirection Logic

As per the requirements:

  * **On success it is redirected to the verify, on failure it is redirected back to resend.**

    In this consolidated code, the "redirection" to "verify" is handled by dynamically showing the OTP input form and hiding the phone number input and request button. This happens within the `requestOtp` function on success.

    For "failure it is redirected back to resend," the current implementation displays an error message. The user can then manually click the "Resend OTP" button that is already visible on the "verify" form. If a more explicit "redirection" back to resend (e.g., hiding the verify input and showing only the resend button) is desired on verification failure, you would need to adjust the `verifyOtp`'s `catch` block to modify the UI state accordingly. However, the current setup where the resend button is always available when the verification form is shown provides a direct path for the user.

  * **On successful OTP verification:**
    The `onVerifySuccess` callback is specifically designed for this. In the provided code, it redirects the user to `/success`:

    ```javascript
    onVerifySuccess: (data) => {
        console.log('OTP Verify Success:', data);
        window.location.href = '/success'; // This is the redirection on success
    }
    ```

## How to Use These Snippets

1.  **Base HTML:** Use the full HTML structure (including CSS) as the foundation for your OTP authorization page.
2.  **JavaScript (`OtpClient` Module):** Place the entire `OtpClient` IIFE within a `<script>` tag, preferably at the end of your `<body>` to ensure all HTML elements are loaded.
3.  **Initialization:** The final `const otpClient = OtpClient.createOtpButton(...)` block is crucial for activating the service. Customize the `apiBaseUrl`, `appKey`, and callback functions as needed for your specific application.

This setup provides a complete client-side OTP authorization experience within a single page, simplifying deployment.
