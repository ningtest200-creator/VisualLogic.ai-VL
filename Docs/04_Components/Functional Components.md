# Functional Components
Functional components provide system-level functionality and utilities. They have **no UI** and must be declared in the **# Frontend Tree** before use (except system method classes).

---

## Frontend Functional Components

### 1. Trigger - Frontend Timer Component

**Purpose**: Execute code at regular time intervals or specific timing events.

**Use Cases**:
- Periodic data refresh or polling
- Animations and transitions
- Countdown timers
- Real-time data updates
- Auto-save functionality
- Game loops and frame-based updates

**Key Properties**:
| Property | Type | Description |
|----------|------|-------------|
| `repeatTimes` | INT | Maximum times to fire; -1 = infinite. Default: -1 |
| `interval` | FLOAT | Time interval between fires in seconds; "" = every frame |
| `autoPlay` | BOOL | Auto-start playback; default: false |
| `isAnimate` | BOOL | Use requestAnimationFrame for animations; default: true |

**Key Methods**:
- `play()` - Start playback from beginning (or resume from pause)
- `pause()` - Pause current playback; can resume later
- `stop()` - Stop and reset to initial state

**Key Events**:
- `@tick(counts, interval, duration)` - Triggered on each interval fire

**VL Example**:
```vl
# Frontend Global Vars
$elapsedTime(FLOAT) = 0
$animationValue(FLOAT) = 0

# Frontend Tree
<Trigger-AnimationTrigger "animTrigger"> interval:"0.016" repeatTimes:(-1) autoPlay:true isAnimate:true

# Frontend Event Handlers

<Trigger-AnimationTrigger "animTrigger">.@tick(counts, interval, duration)
-$elapsedTime = duration
-$animationValue = (duration * 100) / 10
-IF duration > 10
--<Trigger-AnimationTrigger "animTrigger">.stop()

# Usage: Auto-refresh data every 30 seconds
<Trigger-RefreshTrigger "refreshTrigger"> interval:"30" repeatTimes:(-1)

<Trigger-RefreshTrigger "refreshTrigger">.@tick()
-loadLatestData() -> _result
-IF _result.success
--$data = _result.data
```

---

### 2. WindowEventListener - Browser Window Event Listener

**Purpose**: Listen to browser window events and respond to them.

**Use Cases**:
- Handle window resize events
- Detect hash/route changes
- Monitor page visibility
- Keyboard shortcuts
- Scroll detection
- Window focus/blur detection

**Key Events** (React-style camelCase):
- `@keyDown(event)` - Keyboard key pressed
- `@keyUp(event)` - Keyboard key released
- `@resize(event)` - Window resized
- `@scroll(event)` - Window scrolled
- `@hashChange(event)` - URL hash changed
- `@beforeUnload(event)` - Before page unload
- `@focus(event)` - Window gains focus
- `@blur(event)` - Window loses focus
- `@online(event)` - Connection restored
- `@offline(event)` - Connection lost

**VL Example**:
```vl
# Frontend Global Vars
$windowWidth(INT) = 1920
$windowHeight(INT) = 1080
$currentRoute(STRING) = "home"

# Frontend Tree
<WindowEventListener "windowListener">

# Frontend Event Handlers

<WindowEventListener "windowListener">.@resize()
-$windowWidth = WINDOW.innerWidth
-$windowHeight = WINDOW.innerHeight
-IF $windowWidth < 768
--$layoutMode = "mobile"
-ELSE IF $windowWidth < 1024
--$layoutMode = "tablet"
-ELSE
--$layoutMode = "desktop"

<WindowEventListener "windowListener">.@keyDown(event)
-IF event.ctrlKey && event.key == "s"
--event.preventDefault()
--saveDocument() -> _result

<WindowEventListener "windowListener">.@hashChange()
-_newHash(STRING) = WINDOW.location.hash.substring(1)
-$currentRoute = _newHash
-loadRouteData(_newHash) -> _result
```

---

### 3. FrontendApi - Frontend API Request Client (Section/Component Only)

**Purpose**: Make HTTP API requests from frontend to backend services.

**Use Cases**:
- Fetch data from external APIs
- Submit forms to backend
- Upload files
- Real-time data synchronization
- Integration with third-party services

**Key Properties**:
| Property | Type | Description |
|----------|------|-------------|
| `url` | STRING | Required. API endpoint URL; supports path params like `/users/{id}` |
| `method` | STRING | HTTP method: GET, POST, PUT, PATCH, DELETE. Default: GET |
| `headers` | {} | Optional. Default request headers |
| `timeout` | INT | Optional. Request timeout in seconds |
| `params` | (...) | Parameter type declarations, e.g., `params:(page(INT),size(INT))` |
| `returns` | (...) | Return type declarations, e.g., `returns:(success(BOOL),data([{}]))` |

**Key Methods**:
- `send(param1, param2, ...)` - Smart parameter allocation based on HTTP method
- `customSend(headers, body, params, url, timeout, method)` - Full customization

**Return Value**:
- `{success:BOOL, message:STRING, data:JSON}`

**VL Example**:
```vl
# Frontend Global Vars
$users([{}]) = []
$isLoading(BOOL) = false

# Frontend Tree
<FrontendApi-GetUsers "getUsersApi"> url:"/api/users" method:"GET" params:(page(INT),size(INT)) returns:(success(BOOL),users([{}]))
<FrontendApi-CreateUser "createUserApi"> url:"/api/users" method:"POST" params:(name(STRING),email(STRING)) returns:(success(BOOL),userId(INT))
<FrontendApi-UpdateUser "updateUserApi"> url:"/api/users/{id}" method:"PUT" params:(id(INT),name(STRING),email(STRING)) returns:(success(BOOL),message:STRING)

# Frontend Event Handlers

<Button-LoadUsers>.@click()
-$isLoading = true
-<FrontendApi-GetUsers "getUsersApi">.send(1, 20) -> _result
-$isLoading = false
-IF _result.success
--$users = _result.users
-ELSE
--<SysUI>.showToast(_result.message, "error")

<Button-CreateUser>.@click()
-<FrontendApi-CreateUser "createUserApi">.send($newUserName, $newUserEmail) -> _result
-IF _result.success
--<SysUI>.showToast("User created successfully", "success")
--$newUserName = ""
--$newUserEmail = ""
-ELSE
--<SysUI>.showToast(_result.message, "error")

<Button-UpdateUser>.@click()
-<FrontendApi-UpdateUser "updateUserApi">.send($editingUser.id, $editingUser.name, $editingUser.email) -> _result
-IF _result.success
--<SysUI>.showToast("User updated successfully", "success")
```

---

### 4. ClientUserCenter - User Center Client

**Purpose**: Interact with system user authentication and authorization center.

**Use Cases**:
- User login/logout
- User registration
- User profile management
- Permission checking
- Navigate to user center

**Key Methods**:
| Method | Parameters | Return |
|--------|-----------|--------|
| `userLoginPassword` | userName, password | isSuccess, failReason |
| `userLogout` | - | isSuccess, failReason |
| `jumpToUserCenter` | - | - |
| `sendRegistrationSMS` | phoneNumber | isSuccess, failReason |
| `registerUser` | userName, password, verificationCode, phoneNumber | isSuccess, failReason |

**Key Events**:
- `@loginDone()` - Triggered when login completes

**VL Example**:
```vl
# Frontend Global Vars
$isLoggedIn(BOOL) = false
$currentUser({userId:STRING,userName:STRING}) = {userId:"",userName:""}

# Frontend Tree
<ClientUserCenter-UserAuth "userCenter">

# Frontend Event Handlers

<Button-Login>.@click()
-<ClientUserCenter-UserAuth "userCenter">.userLoginPassword($email, $password) -> _loginResult
-IF _loginResult.isSuccess
--$isLoggedIn = true
--$currentUser.userId = _loginResult.userId
--$currentUser.userName = _loginResult.userName
--<SysUI>.showToast("Login successful", "success")
--<ClientUtils>.switchRoute("dashboard")
-ELSE
--<SysUI>.showToast(_loginResult.failReason, "error")

<Button-Logout>.@click()
-<ClientUserCenter-UserAuth "userCenter">.userLogout() -> _logoutResult
-IF _logoutResult.isSuccess
--$isLoggedIn = false
--$currentUser = {userId:"",userName:""}
--<SysUI>.showToast("Logged out successfully", "success")
--<ClientUtils>.switchRoute("home")

<ClientUserCenter-UserAuth "userCenter">.@loginDone()
-$isLoggedIn = true
```

---

## Frontend System Method Classes (No Declaration Needed)

### 1. ClientUtils - Frontend Utilities

**Purpose**: General-purpose frontend utility methods.

**Key Methods**:
| Method | Parameters | Description |
|--------|-----------|-------------|
| `delay` | milliSecond(INT) | Delay execution for specified milliseconds |
| `consoleLog` | title(STRING), message(STRING) | Log message to browser console for debugging |
| `switchRoute` | path(STRING) | Navigate to different route, e.g., "dashboard", "user/profile" |

**VL Example**:
```vl
<Button-Navigate>.@click()
-<ClientUtils>.consoleLog("Navigation", "User navigating to dashboard")
-<ClientUtils>.delay(500)
-<ClientUtils>.switchRoute("dashboard")
```

---

### 2. SysUI - System UI Methods

**Purpose**: Display system UI elements like modals, toasts, and loading indicators.

**Key Methods**:
| Method | Parameters | Description |
|--------|-----------|-------------|
| `showModal` | title(STRING), content(STRING) | Show confirm/cancel modal; returns {confirm:BOOL, cancel:BOOL} |
| `showLoading` | title(STRING) | Show loading spinner overlay |
| `hideLoading` | - | Hide loading spinner |
| `showToast` | message(STRING), iconType(STRING), customizeIcon(STRING), duration(INT) | Show timed notification (duration in seconds) |
| `hideToast` | - | Immediately hide toast notification |

**VL Example**:
```vl
<Button-Delete>.@click()
-<SysUI>.showModal("Confirm Delete", "Are you sure?") -> _result
-IF _result.confirm
--<SysUI>.showLoading("Deleting...")
--deleteItem($itemId) -> _deleteResult
--<SysUI>.hideLoading()
--IF _deleteResult.success
---<SysUI>.showToast("Item deleted successfully", "success")

<Button-Save>.@click()
-<SysUI>.showToast("Data saved!", "success", "", 3)
```

---

### 3. SysFile - File Operations

**Purpose**: Handle file uploads, downloads, and local file reading.

**Key Methods for Upload**:
| Method | Parameters | Description |
|--------|-----------|-------------|
| `uploadImage` | url(STRING) | Upload single image; returns file info |
| `uploadImages` | quantityLimit(INT), urls([STRING]) | Batch upload images (max 20) |
| `uploadFile` | url(STRING), accept(STRING) | Upload single file |
| `uploadFiles` | quantityLimit(INT), urls([STRING]) | Batch upload files (max 20) |

**Key Methods for Reading Local Files**:
| Method | Parameters | Description |
|--------|-----------|-------------|
| `browseImage` | outputBase64(BOOL), compressedWidth(INT), compressedHeight(INT), type(STRING), encoderOptions(INT) | Read local image; returns base64, name, size, width, height, temporary path, file object |
| `browseImages` | maxNumberPic(INT), outputBase64(BOOL), type(STRING), encoderOptions(INT) | Read multiple local images |
| `browseFile` | outputBase64(BOOL), accept(STRING) | Read local file |
| `browseFiles` | accept(STRING), maxNumberFile(INT) | Read multiple local files |

**Key Methods for Download**:
| Method | Parameters | Description |
|--------|-----------|-------------|
| `downloadFile` | fileName(STRING), url(STRING) | Download file from URL to local |

**VL Example**:
```vl
# Upload Image
<Button-UploadProfilePic>.@click()
-<SysFile>.browseImage(true, 200, 200, "jpg", 0.8) -> _imageResult
-IF _imageResult.name != ""
--<SysUI>.showLoading("Uploading...")
--<SysFile>.uploadImage("/api/upload/profile") -> _uploadResult
--<SysUI>.hideLoading()
--IF _uploadResult.success
---$profileImage = _uploadResult.url
---<SysUI>.showToast("Profile picture updated", "success")

# Download File
<Button-DownloadReport>.@click()
-<SysFile>.downloadFile("report.pdf", "https://api.example.com/reports/report.pdf") -> _result
-IF _result.success
--<SysUI>.showToast("Download started", "success")
```

---

### 4. SysLocalStorage - Local Storage Management

**Purpose**: Store and retrieve data in browser storage (Cookie, SessionStorage, LocalStorage).

**Key Methods**:
| Method | Parameters | Description |
|--------|-----------|-------------|
| `setCookie` | path(STRING), domain(STRING), maxAge(INT), key(STRING), value(STRING) | Set single cookie |
| `setCookies` | path(STRING), domain(STRING), maxAge(INT), keyValue({}) | Set multiple cookies |
| `getCookie` | key(STRING) | Get cookie value by key |
| `removeCookie` | key(STRING) | Remove cookie by key |

**VL Example**:
```vl
# Save user preference
<Button-SavePreference>.@click()
-<SysLocalStorage>.setCookie("/", "example.com", 86400, "theme", "dark")
-<SysLocalStorage>.setCookie("/", "example.com", 86400, "language", "en")
-<SysUI>.showToast("Preferences saved", "success")

# Load user preference
<Page-Home>.@init()
-<SysLocalStorage>.getCookie("theme") -> _theme
-IF _theme != ""
--$userTheme = _theme
```

---

### 5. SysDevice - Device Hardware Access

**Purpose**: Access device hardware features like camera and sensors.

**Key Methods**:
| Method | Parameters | Description |
|--------|-----------|-------------|
| `scanCode` | - | Open camera for QR/barcode scanning; returns scan result |

**VL Example**:
```vl
<Button-ScanQRCode>.@click()
-<SysUI>.showLoading("Opening camera...")
-<SysDevice>.scanCode() -> _scanResult
-<SysUI>.hideLoading()
-IF _scanResult.data != ""
--$scannedCode = _scanResult.data
--processScannedCode(_scanResult.data) -> _result
-ELSE
--<SysUI>.showToast("Scan failed: " + _scanResult.errMsg, "error")
```

---

## Backend Functional Components

### 1. ServerApi - Backend API Request Client

**Purpose**: Make API requests from backend services.

**Properties & Methods**: Same as FrontendApi

**Usage**: Identical to FrontendApi, used in backend Service definitions

---

### 2. MQ - Message Queue

**Purpose**: Asynchronous message processing for decoupling operations.

**Use Cases**:
- Async task processing
- Event-driven architecture
- Request peak shaving

- Decoupled service communication

**Key Properties**:


| Property | Type | Description                                       |
| -------- | ---- | ------------------------------------------------- |
| `struct` | {}   | Message structure definition, single-layer object |

**Key Methods**:

- `send(field1, field2, ...)` - Send message to queue; structure must match struct definition

**Key Events**:

- `@message(field1, field2, ...)` - Triggered when message received; process message here

**VL Example**:

```vl
# Backend Tree
<MQ-OrderQueue "orderQueue"> struct:{orderId:INT,userId:INT,amount:FLOAT,status:STRING}

# Services
SERVICE PlaceOrder(orderId(INT), userId(INT), amount(FLOAT)); RETURN {success:BOOL, message:STRING}
-<MQ-OrderQueue "orderQueue">.send(orderId, userId, amount, "pending")
-RETURN {success:true, message:"Order placed successfully"}

# Backend Event Handlers
<MQ-OrderQueue "orderQueue">.@message(orderId, userId, amount, status)
-_order({id:INT,userId:INT,amount:FLOAT,status:STRING}) = {id:orderId,userId:userId,amount:amount,status:status}
-<VirtualTable-Orders "orderTable">.insert(_order) -> _insertResult
-IF _insertResult.success
--sendOrderConfirmationEmail(userId, orderId) -> _emailResult
--log(("Order " + (orderId | STRING) + " processed successfully"))
-ELSE
--log(("Failed to process order: " + _insertResult.message))
```

---

## System Variables

### SYSENV - System Environment

**Available Properties**:


| Property             | Scope              | Type                                                        | Description                        |
| -------------------- | ------------------ | ----------------------------------------------------------- | ---------------------------------- |
| `SYSENV.currentUser` | Frontend & Backend | {isLogin:BOOL, userId:STRING, userInfo:{...}, tenantId:INT} | Current logged-in user information |
| `SYSENV.currentTime` | Frontend & Backend | TIMESTAMP                                                   | Server's current time              |
| `SYSENV.requestInfo` | Backend Only       | {ip:STRING, userAgent:STRING, headerInfo:{}, cookie:{}}     | Current request info               |

**VL Example**:

```vl
# Frontend
-IF SYSENV.currentUser.isLogin
--$currentUserId = SYSENV.currentUser.userId
--$currentUserInfo = SYSENV.currentUser.userInfo
--<ClientUtils>.switchRoute("dashboard")
-ELSE
--<ClientUtils>.switchRoute("login")

# Record creation timestamp
-_createdAt(TIMESTAMP) = SYSENV.currentTime
-<VirtualTable-Documents "docTable">.insert({title:$title, createdAt:_createdAt}) -> _result

# Backend - Log request IP
-_clientIp(STRING) = SYSENV.requestInfo.ip
-log(("Request from IP: " + _clientIp))
```

### WINDOW - Browser Window Object

**Available Properties** (Frontend Only):

- `WINDOW.innerWidth`, `WINDOW.innerHeight` - Viewport dimensions
- `WINDOW.outerWidth`, `WINDOW.outerHeight` - Full window dimensions
- `WINDOW.devicePixelRatio` - Device pixel ratio
- `WINDOW.scrollX`, `WINDOW.scrollY` - Scroll position
- `WINDOW.location` - URL and navigation info

**VL Example**:

```vl
# Responsive layout
-IF WINDOW.innerWidth < 768
--$layoutMode = "mobile"
-ELSE IF WINDOW.innerWidth < 1024
--$layoutMode = "tablet"
-ELSE
--$layoutMode = "desktop"

# Get current URL path
-_currentPath(STRING) = WINDOW.location.pathname

# Scroll detection
<WindowEventListener "scrollListener">

<WindowEventListener "scrollListener">.@scroll()
-IF WINDOW.scrollY > 100
--$showScrollToTop = true
-ELSE
--$showScrollToTop = false
```

---

## Theme Tokens

**Access Format**: `--tokenName` in CSS properties

**Common Token Categories**:

- **Colors**: `--colorBrandPrimary`, `--colorTextDefault`, `--colorBgBody`, `--colorSuccess`, `--colorError`, `--colorWarning`
- **Typography**: `--fontSizeBase`, `--fontSizeLarge`, `--fontWeightMedium`, `--fontWeightBold`, `--fontFamilyBase`
- **Spacing**: `--spacing1`, `--spacing2`, `--spacing4`, `--spacing6`, `--spacing8`
- **Sizing**: `--heightBase`, `--heightLarge`, `--radiusSm`, `--radiusMd`, `--radiusLg`
- **Shadows**: `--shadowBase`, `--shadowMd`, `--shadowLg`, `--shadowCard`
- **Z-Index**: `--zIndexDropdown`, `--zIndexModal`, `--zIndexToast`

**VL Example**:

```vl
<Block-Card> padding:"--spacing4" background-color:"--colorBgBody" border-radius:"--radiusMd" box-shadow:"--shadowCard"
-<Text-Title> value:"Title" StyleClass:HeadingMedium color:"--colorTextDefault"
-<Button-Action> value:"Click" StyleClass:ButtonPrimary

<Input-Field> padding:"--spacing2 --spacing3" border-radius:"--radiusSm" font-size:"--fontSizeBase"
```

---

## Best Practices for Functional Components

### 1. **Proper Trigger Usage**

```vl
# ✅ CORRECT: Use Trigger for repeated actions
<Trigger-DataRefresh "refreshTrigger"> interval:"30" repeatTimes:(-1) autoPlay:true

<Trigger-DataRefresh "refreshTrigger">.@tick()
-loadLatestData() -> _result

# ❌ WRONG: Using repetitive manual loops
-FOR (_i(INT) = 0; _i < 100; _i++)
--loadData()  # This blocks; use Trigger instead
```

### 2. **Window Events for Responsive Design**

```vl
# ✅ CORRECT: Business logic-based conditions
<If-HasData> conditions:($data.length > 0)
--<For-DataList> sourceArray:$data

# ❌ WRONG: Device type detection in If condition
<If-MobileLayout> conditions:(WINDOW.innerWidth < 768)
--# This violates architecture; use separate apps for different devices
```

### 3. **API Error Handling**

```vl
# ✅ CORRECT: Use system default error handling
<Button-Load>.@click()
-<FrontendApi-GetData "dataApi">.send(1, 20) -> _result
-IF _result.success
--$data = _result.data
-ELSE
--<SysUI>.showToast(_result.message, "error")

# With timeout handling
<Button-LoadWithTimeout>.@click()
-<FrontendApi-TimeoutApi "api"> timeout:5
--send() -> _result
```

### 4. **File Upload with Progress**

```vl
# ✅ CORRECT: Show progress feedback
<SysFile>.browseFile(false, ".pdf") -> _fileResult
-IF _fileResult.name != ""
--<SysUI>.showLoading("Uploading " + _fileResult.name + "...")
--<SysFile>.uploadFile("/api/upload") -> _uploadResult
--<SysUI>.hideLoading()
--IF _uploadResult.success
---<SysUI>.showToast("Upload complete", "success")
```

### 5. **User Authentication Flow**

```vl
# ✅ CORRECT: Check auth state on page init
<Page-Dashboard>.@init()
-IF !SYSENV.currentUser.isLogin
--<ClientUtils>.switchRoute("login")
--RETURN
-loadDashboardData() -> _result

# Store user in global
<ClientUserCenter-Auth "authCenter">.@loginDone()
-$currentUser.id = SYSENV.currentUser.userId
-$currentUser.name = SYSENV.currentUser.userInfo.fullName
-$isLoggedIn = true
```

---

## Summary Table: Functional Components


| Component               | Type     | Purpose           | Main Feature                    |
| ----------------------- | -------- | ----------------- | ------------------------------- |
| **Trigger**             | Frontend | Timed execution   | Periodic firing, animation      |
| **WindowEventListener** | Frontend | Browser events    | Window resize, keyboard, scroll |
| **FrontendApi**         | Frontend | HTTP requests     | Smart parameter allocation      |
| **ClientUserCenter**    | Frontend | Authentication    | Login/logout, permissions       |
| **ClientUtils**         | System   | General utilities | Navigation, logging, delay      |
| **SysUI**               | System   | UI display        | Modal, toast, loading           |
| **SysFile**             | System   | File operations   | Upload, download, read          |
| **SysLocalStorage**     | System   | Data persistence  | Cookie management               |
| **SysDevice**           | System   | Hardware access   | QR scanning                     |
| **ServerApi**           | Backend  | HTTP requests     | Backend API calls               |
| **MQ**                  | Backend  | Async processing  | Message queue, event handling   |

---

## Key Rules Summary

### Declaration Requirements

- ✅ Trigger, WindowEventListener, FrontendApi, ClientUserCenter: **must declare in # Frontend Tree**
- ✅ ServerApi, MQ: **must declare in # Backend Tree**
- ✅ ClientUtils, SysUI, SysFile, SysLocalStorage, SysDevice: **no declaration needed** (system-provided)

### System Variables

- ✅ SYSENV: available in both frontend and backend
- ✅ WINDOW: frontend only
- ✅ Theme tokens: use `--tokenName` format in CSS properties

### Best Practices

- Use Trigger for animations instead of manual loops
- Use WindowEventListener for responsive behavior, not If conditions
- Always handle API errors with @onChange events
- Check SYSENV.currentUser.isLogin before protected pages
- Use SysUI for all user feedback (toasts, modals, loading)
