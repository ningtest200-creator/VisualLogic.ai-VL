# Structural Container Components

Structural container components are used to manage page structure and routing. They can be used in **App** files.

---

## Page - Page Container (App Only)

### Introduction

- **Purpose**: Application page container responsible for route mapping and page lifecycle management
- **Use Cases**:
  - Creating different views/screens in an application
  - Route-based page navigation
  - Page-level initialization and data loading
  - Managing page state and lifecycle
- **Characteristics**:
  - Must be placed directly under the root App component
  - Cannot be nested inside other containers
  - Automatically manages routing and page transitions

### Key Properties


| Property | Type   | Description                                                                                           |
| -------- | ------ | ----------------------------------------------------------------------------------------------------- |
| `path`   | STRING | Route path identifier (logical path without leading slash), e.g., "home", "dashboard", "user/profile" |

### Common Events


| Event     | Parameters | Description                                                                            |
| --------- | ---------- | -------------------------------------------------------------------------------------- |
| `@init()` | None       | Triggered when page initialization completes; common entry point for loading page data |

### VL Syntax Examples

**Example 1: Basic App with Multiple Pages**

```vl
// VL_VERSION:2.2
<App-MainApp "root">

# SysConfig
DEVICE_TARGET:"PC"
SCREEN_RESOLUTION:"1920x1080"

# Frontend Global Vars
$currentUser({id:INT,name:STRING,email:STRING,role:STRING}) = {id:0,name:"",email:"",role:""}
$isLoggedIn(BOOL) = false
$notifications(INT) = 0

# Frontend Tree
<Page-Home "homePage"> path:"home"
-<Section-Homepage "homepage">

<Page-Dashboard "dashboardPage"> path:"dashboard"
-<Section-Dashboard "dashboard">

<Page-Profile "profilePage"> path:"profile"
-<Section-UserProfile "userProfile">

<Page-Settings "settingsPage"> path:"settings"
-<Section-Settings "settings">

# Frontend Event Handlers

<Page-Home "homePage">.@init()
-IF !$isLoggedIn
--<ClientUtils>.switchRoute("login")
--RETURN
-<SysUI>.showLoading("Loading home page...")
-loadHomeData() -> _result
-<SysUI>.hideLoading()
-IF !_result.success
--<SysUI>.showToast("Failed to load home page", "error")

<Page-Dashboard "dashboardPage">.@init()
-IF !$isLoggedIn
--<ClientUtils>.switchRoute("login")
--RETURN
-<SysUI>.showLoading("Loading dashboard...")
-loadDashboardData() -> _result
-<SysUI>.hideLoading()
-IF _result.success
--$notifications = _result.notificationCount
-ELSE
--<SysUI>.showToast(_result.message, "error")

<Page-Profile "profilePage">.@init()
-loadUserProfile($currentUser.id) -> _result
-IF _result.success
--$currentUser = _result.userData
-ELSE
--<SysUI>.showToast("Failed to load profile", "error")

<Page-Settings "settingsPage">.@init()
-loadUserSettings($currentUser.id) -> _result
-IF !_result.success
--<SysUI>.showToast("Failed to load settings", "error")

# Frontend Internal Methods

METHOD loadHomeData(); RETURN {success:BOOL,data:{}}
-loadFeaturedProducts() -> _products
-loadBanners() -> _banners
-RETURN {success:true,data:{products:_products,banners:_banners}}

METHOD loadDashboardData(); RETURN {success:BOOL,notificationCount:INT}
-<ServiceDomain-Dashboard "dashboardService">.GetDashboardStats() -> _stats
-<ServiceDomain-Notification "notifService">.GetNotifications() -> _notifications
-IF !_stats.success || !_notifications.success
--RETURN {success:false,notificationCount:0}
-$notifications = _notifications.data.length
-RETURN {success:true,notificationCount:_notifications.data.length}

METHOD loadUserProfile(userId(INT)); RETURN {success:BOOL,userData:{}}
-<ServiceDomain-User "userService">.GetUserProfile(userId) -> _profile
-IF !_profile.success
--RETURN {success:false,userData:{}}
-RETURN {success:true,userData:_profile.data}

METHOD loadUserSettings(userId(INT)); RETURN {success:BOOL,settings:{}}
-<ServiceDomain-Settings "settingsService">.GetUserSettings(userId) -> _settings
-IF !_settings.success
--RETURN {success:false,settings:{}}
-RETURN {success:true,settings:_settings.data}

</App-MainApp>
```

**Example 2: E-commerce App**

```vl
// VL_VERSION:2.2
<App-EcommerceApp "root">

# SysConfig
DEVICE_TARGET:"PC"
SCREEN_RESOLUTION:"1920x1080"

# Frontend Global Vars
$currentUser({id:INT,name:STRING,email:STRING}) = {id:0,name:"",email:""}
$isLoggedIn(BOOL) = false
$cartItems(INT) = 0
$wishlistItems(INT) = 0

# Frontend Tree
<Page-Home "homePage"> path:"home"
-<Section-Homepage "homepage"> cartCount:$cartItems

<Page-ProductList "productsPage"> path:"products"
-<Section-ProductList "productList"> onProductSelect:$onProductSelected

<Page-ProductDetail "productDetailPage"> path:"product/:id"
-<Section-ProductDetail "productDetail">

<Page-Cart "cartPage"> path:"cart"
-<Section-ShoppingCart "cart"> itemCount:$cartItems

<Page-Checkout "checkoutPage"> path:"checkout"
-<Section-Checkout "checkout">

<Page-OrderConfirmation "confirmationPage"> path:"order/confirmation"
-<Section-OrderConfirmation "confirmation">

<Page-UserAccount "accountPage"> path:"account"
-<Section-UserAccount "account">

# Frontend Event Handlers

<Page-Home "homePage">.@init()
-<SysUI>.showLoading("Loading...")
-loadHomepageData() -> _result
-<SysUI>.hideLoading()
-IF !_result.success
--<SysUI>.showToast(_result.message, "error")

<Page-ProductList "productsPage">.@init()
-<SysUI>.showLoading("Loading products...")
-loadProducts() -> _result
-<SysUI>.hideLoading()
-IF !_result.success
--<SysUI>.showToast("Failed to load products", "error")

<Page-ProductDetail "productDetailPage">.@init()
-_productId(STRING) = WINDOW.location.pathname.split("/").pop()
-<SysUI>.showLoading("Loading product...")
-loadProductDetail(_productId) -> _result
-<SysUI>.hideLoading()
-IF !_result.success
--<SysUI>.showToast("Product not found", "error")
--<ClientUtils>.switchRoute("products")

<Page-Cart "cartPage">.@init()
-loadCartItems() -> _result
-IF _result.success
--$cartItems = _result.itemCount
-ELSE
--<SysUI>.showToast("Failed to load cart", "error")

<Page-Checkout "checkoutPage">.@init()
-IF !$isLoggedIn
--<SysUI>.showToast("Please login first", "warning")
--<ClientUtils>.switchRoute("home")
--RETURN
-loadCheckoutData() -> _result
-IF !_result.success
--<SysUI>.showToast("Failed to load checkout data", "error")

<Page-OrderConfirmation "confirmationPage">.@init()
-_orderId(STRING) = WINDOW.location.search.split("=")[1]
-loadOrderDetails(_orderId) -> _result
-IF !_result.success
--<SysUI>.showToast("Order not found", "error")
--<ClientUtils>.switchRoute("home")

<Page-UserAccount "accountPage">.@init()
-IF !$isLoggedIn
--<ClientUtils>.switchRoute("home")
--RETURN
-loadUserAccountData($currentUser.id) -> _result
-IF !_result.success
--<SysUI>.showToast("Failed to load account data", "error")

# Frontend Internal Methods

METHOD loadHomepageData(); RETURN {success:BOOL,message:STRING}
-<ServiceDomain-Product "productService">.GetFeaturedProducts() -> _featured
-<ServiceDomain-Promotion "promoService">.GetActiveBanners() -> _banners
-IF !_featured.success || !_banners.success
--RETURN {success:false,message:"Failed to load homepage"}
-RETURN {success:true,message:""}

METHOD loadProducts(); RETURN {success:BOOL,message:STRING}
-<ServiceDomain-Product "productService">.GetAllProducts() -> _result
-IF !_result.success
--RETURN {success:false,message:_result.message}
-RETURN {success:true,message:""}

METHOD loadProductDetail(productId(STRING)); RETURN {success:BOOL,message:STRING}
-_id(INT) = Number(productId)
-<ServiceDomain-Product "productService">.GetProductDetail(_id) -> _result
-IF !_result.success
--RETURN {success:false,message:"Product not found"}
-RETURN {success:true,message:""}

METHOD loadCartItems(); RETURN {success:BOOL,itemCount:INT}
-<ServiceDomain-Cart "cartService">.GetCartItems($currentUser.id) -> _result
-IF !_result.success
--RETURN {success:false,itemCount:0}
-RETURN {success:true,itemCount:_result.items.length}

METHOD loadCheckoutData(); RETURN {success:BOOL,message:STRING}
-<ServiceDomain-Cart "cartService">.GetCartItems($currentUser.id) -> _cartResult
-<ServiceDomain-User "userService">.GetShippingAddresses($currentUser.id) -> _addressResult
-IF !_cartResult.success || !_addressResult.success
--RETURN {success:false,message:"Failed to load checkout data"}
-RETURN {success:true,message:""}

METHOD loadOrderDetails(orderId(STRING)); RETURN {success:BOOL,message:STRING}
-<ServiceDomain-Order "orderService">.GetOrderDetails(orderId) -> _result
-IF !_result.success
--RETURN {success:false,message:"Order not found"}
-RETURN {success:true,message:""}

METHOD loadUserAccountData(userId(INT)); RETURN {success:BOOL,message:STRING}
-<ServiceDomain-User "userService">.GetUserAccount(userId) -> _account
-<ServiceDomain-Order "orderService">.GetUserOrders(userId) -> _orders
-IF !_account.success || !_orders.success
--RETURN {success:false,message:"Failed to load account data"}
-RETURN {success:true,message:""}

</App-EcommerceApp>
```
