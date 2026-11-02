Here is a deep summary of the first two parts of the [Inside look at modern web browser](https://developer.chrome.com/blog) series, structured as detailed study notes:

---
<img width="1024" height="1024" alt="chrome logo with it's process" src="https://github.com/user-attachments/assets/6384e613-5448-4b43-b544-e8e5451cafb2" />



# Part 1: Multi-Process Architecture

### 1. Why a Multi-Process Architecture?

* **Early Browsers:** Historically ran as a single process. If one tab or plugin (like Flash) crashed, the entire browser crashed.
* **Modern Browsers:** Shifted to a **multi-process architecture** to isolate tasks. If a single tab misbehaves or crashes, the browser can show a "Aw, Snap!" error page on that tab alone while the rest of the browser and your other tabs keep working normally.

### 2. Core Browser Processes & Their Roles

* **Browser Process:** The "brain" that controls the core application. Manages the address bar, bookmarks, back/forward buttons, and coordinates all networking and security requests.
* **Renderer Process:** Handles everything that displays *inside* a tab (HTML, CSS, JavaScript). Each tab typically gets its own renderer process (though site isolation can separate cross-site iframes into independent processes).
* **Plugin Process:** Manages browser extensions and plugins (like Flash or PDF viewers) to isolate them from crashing the core system.
* **GPU Process:** Handles GPU tasks separately in an isolated process to deal with hardware-accelerated drawing and compositing smoothly.

### 3. Safety and Sandbox Security

* Because renderer processes handle untrusted web content from the internet, the operating system places them inside a **sandbox**.
* The sandbox restricts the renderer's ability to read or write files directly to your hard drive, preventing malicious code from executing **arbitrary file access** or taking over your machine.

---

# Part 2: Navigation Flow (What Happens When You Type a URL)

### Step 1: Handling Input

* When you type into the address bar, the UI thread in the **Browser Process** evaluates your input:
* **Search Query:** Sends it to your default search engine via the network.
* **URL:** Directly initiates a network request to load that specific website address.



### Step 2: Start Navigation

* When you press Enter, the UI thread triggers the **Network Thread** to handle internet communication (performing DNS lookups, establishing TCP/TLS connections, and sending HTTP GET requests).

### Step 3: Read Response & Security Checks

* As the server responds with data, the network thread reads the initial bytes (MIME type sniffing) to determine what kind of file it is (`text/html`, zip, image, etc.).
* **Security Interceptors:**
* **SafeBrowsing:** Checks if the URL matches known malicious or phishing sites.
* **CORB (Cross-Origin Read Blocking):** Ensures sensitive cross-site data isn't improperly leaked to the renderer.



### Step 4: Find a Renderer Process

* Once security checks pass and data arrives, the network thread signals the UI thread that the site is ready.
* The browser concurrently spins up a **Renderer Process** in the background while data is downloading to save precious milliseconds.

### Step 5: Commit Navigation

* An **IPC (Inter-Process Communication)** message is sent from the browser process to the renderer process to officially hand over the data stream.
* The renderer processes the incoming HTML/assets, the address bar updates its URL, and the session history updates.

---

### Key Architectural Concepts to Remember

* **IPC (Inter-Process Communication):** Because processes are isolated in their own memory spaces, they rely on IPC messages to coordinate actions (e.g., Browser telling the Renderer to render a page, or the Renderer sending a compositor frame back).
* **Service Workers:** If a site registers a service worker, it can intercept network requests, bypass the live web by loading cached assets locally, and speed up navigation.
