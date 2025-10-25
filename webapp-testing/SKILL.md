---
name: webapp-testing
description: "Professional web application testing and automation using Playwright with support for multiple browsers, mobile emulation, screenshot capture, network interception, and comprehensive test assertions. Use for: (1) E2E testing across browsers, (2) UI automation, (3) Form testing and validation, (4) Visual regression testing, (5) API mocking and interception, (6) Mobile responsive testing"
---

# Web Application Testing with Playwright

## Overview

Playwright is a powerful framework for web testing and automation that supports all modern browsers (Chromium, Firefox, WebKit). It provides reliable, fast, and capable automation with auto-waiting, network control, and comprehensive testing capabilities.

## Core Capabilities

### Multi-Browser Testing
- **Chromium**: Chrome, Edge, Brave
- **Firefox**: Mozilla Firefox
- **WebKit**: Safari engine
- Cross-browser compatibility testing
- Parallel execution across browsers

### Element Interaction
- Click, double-click, right-click
- Type text with realistic keyboard simulation
- Select dropdowns and checkboxes
- Hover and focus interactions
- Drag and drop operations
- File uploads and downloads

### Assertions & Verification
- Element visibility and state checks
- Text content verification
- Attribute validation
- URL and navigation assertions
- Custom expect matchers
- Soft assertions for multiple checks

### Screenshot & Video Capture
- Full page screenshots
- Element-specific captures
- Video recording of test sessions
- Visual comparison testing
- Trace files for debugging

### Network Interception
- Mock API responses
- Intercept and modify requests
- Monitor network traffic
- Test offline scenarios
- Performance monitoring

### Mobile Device Emulation
- 100+ device presets
- Custom viewport configurations
- Touch event simulation
- Geolocation testing
- Orientation changes

## Installation & Setup

### Python Installation

```bash
# Install Playwright
pip install playwright pytest-playwright

# Install browsers
playwright install

# Install specific browser only
playwright install chromium
```

### JavaScript/TypeScript Installation

```bash
# Using npm
npm init playwright@latest

# Using yarn
yarn create playwright

# Install browsers
npx playwright install
```

### Project Structure

```
project/
├── tests/
│   ├── __init__.py
│   ├── test_login.py
│   ├── test_checkout.py
│   └── test_navigation.py
├── pages/
│   ├── __init__.py
│   ├── base_page.py
│   ├── login_page.py
│   └── product_page.py
├── fixtures/
│   ├── __init__.py
│   └── test_data.py
├── utils/
│   ├── __init__.py
│   └── helpers.py
├── playwright.config.ts  # For TypeScript
├── pytest.ini            # For Python
└── requirements.txt
```

## Configuration

### Python Configuration (pytest.ini)

```ini
[pytest]
testpaths = tests
python_files = test_*.py
python_classes = Test*
python_functions = test_*
markers =
    smoke: Quick smoke tests
    regression: Full regression suite
    slow: Tests that take longer
addopts =
    --headed
    --browser chromium
    --browser firefox
    --screenshot on
    --video retain-on-failure
```

### TypeScript Configuration (playwright.config.ts)

```typescript
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './tests',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: 'html',
  use: {
    baseURL: 'http://localhost:3000',
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
    video: 'retain-on-failure',
  },
  projects: [
    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'] },
    },
    {
      name: 'firefox',
      use: { ...devices['Desktop Firefox'] },
    },
    {
      name: 'webkit',
      use: { ...devices['Desktop Safari'] },
    },
    {
      name: 'Mobile Chrome',
      use: { ...devices['Pixel 5'] },
    },
    {
      name: 'Mobile Safari',
      use: { ...devices['iPhone 12'] },
    },
  ],
  webServer: {
    command: 'npm run start',
    url: 'http://localhost:3000',
    reuseExistingServer: !process.env.CI,
  },
});
```

## Detailed Workflows

### 1. Basic Test Setup (Python)

```python
import pytest
from playwright.sync_api import Page, expect

def test_homepage_loads(page: Page):
    """Test that homepage loads successfully."""
    page.goto("https://example.com")
    expect(page).to_have_title("Example Domain")
    expect(page.locator("h1")).to_contain_text("Example Domain")

def test_navigation(page: Page):
    """Test navigation between pages."""
    page.goto("https://example.com")

    # Click navigation link
    page.click("text=More information")

    # Verify new URL
    expect(page).to_have_url("https://www.iana.org/domains/reserved")
```

### 2. Login Flow Testing

```python
import pytest
from playwright.sync_api import Page, expect

class TestLogin:
    """Test suite for login functionality."""

    @pytest.fixture(autouse=True)
    def setup(self, page: Page):
        """Navigate to login page before each test."""
        page.goto("https://example.com/login")
        yield
        # Cleanup if needed

    def test_successful_login(self, page: Page):
        """Test successful login with valid credentials."""
        # Fill login form
        page.fill("#username", "testuser@example.com")
        page.fill("#password", "SecurePassword123")

        # Submit form
        page.click("button[type='submit']")

        # Verify successful login
        expect(page).to_have_url("https://example.com/dashboard")
        expect(page.locator(".welcome-message")).to_contain_text("Welcome, Test User")

    def test_invalid_credentials(self, page: Page):
        """Test login fails with invalid credentials."""
        page.fill("#username", "wrong@example.com")
        page.fill("#password", "WrongPassword")
        page.click("button[type='submit']")

        # Verify error message appears
        error = page.locator(".error-message")
        expect(error).to_be_visible()
        expect(error).to_contain_text("Invalid credentials")

        # Verify still on login page
        expect(page).to_have_url("https://example.com/login")

    def test_empty_fields_validation(self, page: Page):
        """Test validation for empty form fields."""
        page.click("button[type='submit']")

        # Check for HTML5 validation or custom error messages
        username_input = page.locator("#username")
        expect(username_input).to_have_attribute("required")

    def test_remember_me_functionality(self, page: Page):
        """Test remember me checkbox persistence."""
        page.fill("#username", "testuser@example.com")
        page.fill("#password", "SecurePassword123")
        page.check("#remember-me")
        page.click("button[type='submit']")

        # Wait for navigation
        page.wait_for_url("https://example.com/dashboard")

        # Check cookie was set
        cookies = page.context.cookies()
        remember_cookie = next((c for c in cookies if c['name'] == 'remember_token'), None)
        assert remember_cookie is not None
        assert remember_cookie['expires'] > 0  # Long-lived cookie
```

### 3. Form Testing and Validation

```python
from playwright.sync_api import Page, expect
import pytest

def test_contact_form_submission(page: Page):
    """Test complete contact form submission flow."""
    page.goto("https://example.com/contact")

    # Fill multi-field form
    page.fill("#name", "John Doe")
    page.fill("#email", "john@example.com")
    page.fill("#phone", "+1-555-0123")
    page.select_option("#country", "United States")
    page.check("#newsletter")
    page.fill("#message", "This is a test message for the contact form.")

    # Handle file upload
    page.set_input_files("#attachment", "path/to/test-file.pdf")

    # Submit form
    page.click("button[type='submit']")

    # Verify success message
    expect(page.locator(".success-message")).to_be_visible()
    expect(page.locator(".success-message")).to_contain_text("Thank you for contacting us")

def test_form_validation_messages(page: Page):
    """Test client-side validation messages."""
    page.goto("https://example.com/contact")

    # Try to submit with invalid email
    page.fill("#email", "invalid-email")
    page.fill("#name", "John")
    page.click("button[type='submit']")

    # Check validation message
    email_error = page.locator("#email-error")
    expect(email_error).to_be_visible()
    expect(email_error).to_contain_text("valid email")

def test_dropdown_selections(page: Page):
    """Test dropdown menu selections."""
    page.goto("https://example.com/form")

    # Select by value
    page.select_option("#category", value="electronics")

    # Select by label
    page.select_option("#country", label="Canada")

    # Select by index
    page.select_option("#priority", index=2)

    # Verify selections
    expect(page.locator("#category")).to_have_value("electronics")
```

### 4. E-commerce Testing

```python
from playwright.sync_api import Page, expect

class TestCheckoutFlow:
    """Complete e-commerce checkout testing."""

    def test_add_to_cart(self, page: Page):
        """Test adding products to cart."""
        page.goto("https://example.com/products")

        # Add first product
        page.click(".product-card:nth-child(1) .add-to-cart")

        # Verify cart badge updates
        cart_badge = page.locator(".cart-badge")
        expect(cart_badge).to_contain_text("1")

        # Add second product
        page.click(".product-card:nth-child(2) .add-to-cart")
        expect(cart_badge).to_contain_text("2")

    def test_cart_quantity_update(self, page: Page):
        """Test updating item quantities in cart."""
        page.goto("https://example.com/cart")

        # Increase quantity
        page.click(".quantity-increase")
        expect(page.locator(".item-quantity")).to_have_value("2")

        # Verify total updates
        total = page.locator(".cart-total")
        expect(total).to_contain_text("$")

    def test_complete_checkout(self, page: Page):
        """Test full checkout process."""
        # Add product to cart
        page.goto("https://example.com/products/123")
        page.click(".add-to-cart")

        # Go to cart
        page.click(".cart-icon")
        expect(page).to_have_url("https://example.com/cart")

        # Proceed to checkout
        page.click("button:text('Proceed to Checkout')")

        # Fill shipping information
        page.fill("#first-name", "John")
        page.fill("#last-name", "Doe")
        page.fill("#address", "123 Main St")
        page.fill("#city", "New York")
        page.fill("#zip", "10001")
        page.select_option("#country", "US")

        # Continue to payment
        page.click("button:text('Continue to Payment')")

        # Fill payment details (test mode)
        page.fill("#card-number", "4242424242424242")
        page.fill("#expiry", "12/25")
        page.fill("#cvv", "123")

        # Place order
        page.click("button:text('Place Order')")

        # Verify order confirmation
        expect(page).to_have_url("https://example.com/order/confirmation")
        expect(page.locator(".order-success")).to_be_visible()
        expect(page.locator(".order-number")).to_contain_text("#")

    def test_coupon_code_application(self, page: Page):
        """Test applying discount coupons."""
        page.goto("https://example.com/cart")

        # Get original total
        original_total = page.locator(".total-amount").inner_text()

        # Apply coupon
        page.fill("#coupon-code", "SAVE10")
        page.click("button:text('Apply')")

        # Verify discount applied
        expect(page.locator(".discount-applied")).to_be_visible()
        expect(page.locator(".discount-amount")).to_contain_text("-")

        # Verify new total is less
        new_total = page.locator(".total-amount").inner_text()
        assert float(new_total.replace("$", "")) < float(original_total.replace("$", ""))
```

### 5. API Mocking and Network Interception

```python
from playwright.sync_api import Page, Route
import json

def test_mock_api_response(page: Page):
    """Mock API responses for testing."""

    # Define mock data
    mock_user_data = {
        "id": 123,
        "name": "Test User",
        "email": "test@example.com",
        "premium": True
    }

    # Intercept API call and return mock data
    def handle_route(route: Route):
        if "api/user/profile" in route.request.url:
            route.fulfill(
                status=200,
                content_type="application/json",
                body=json.dumps(mock_user_data)
            )
        else:
            route.continue_()

    page.route("**/api/**", handle_route)

    # Navigate and verify mocked data appears
    page.goto("https://example.com/profile")
    expect(page.locator(".user-name")).to_contain_text("Test User")
    expect(page.locator(".premium-badge")).to_be_visible()

def test_network_failure_handling(page: Page):
    """Test how app handles network failures."""

    # Fail all API requests
    page.route("**/api/**", lambda route: route.abort())

    page.goto("https://example.com/dashboard")

    # Verify error message is shown
    expect(page.locator(".error-message")).to_be_visible()
    expect(page.locator(".retry-button")).to_be_visible()

def test_slow_network_conditions(page: Page):
    """Test app behavior under slow network."""

    def handle_slow_route(route: Route):
        # Delay response by 3 seconds
        import time
        time.sleep(3)
        route.continue_()

    page.route("**/api/**", handle_slow_route)

    page.goto("https://example.com/products")

    # Verify loading state appears
    expect(page.locator(".loading-spinner")).to_be_visible()

    # Wait for content to load
    page.wait_for_selector(".product-card", timeout=5000)

def test_monitor_network_requests(page: Page):
    """Monitor and validate network requests."""
    requests = []

    # Capture all requests
    page.on("request", lambda request: requests.append(request))

    page.goto("https://example.com")
    page.click("button:text('Load More')")

    # Verify expected requests were made
    api_requests = [r for r in requests if "api" in r.url]
    assert len(api_requests) > 0

    # Check specific request
    load_more_request = next((r for r in api_requests if "products" in r.url), None)
    assert load_more_request is not None
    assert load_more_request.method == "GET"
```

### 6. Mobile Device Emulation

```python
from playwright.sync_api import Page, expect
import pytest

@pytest.fixture
def mobile_page(playwright):
    """Create a mobile emulated browser context."""
    iphone_12 = playwright.devices['iPhone 12']
    browser = playwright.chromium.launch()
    context = browser.new_context(**iphone_12)
    page = context.new_page()
    yield page
    context.close()
    browser.close()

def test_mobile_navigation(mobile_page: Page):
    """Test mobile navigation menu."""
    mobile_page.goto("https://example.com")

    # Hamburger menu should be visible on mobile
    expect(mobile_page.locator(".hamburger-menu")).to_be_visible()

    # Desktop menu should be hidden
    expect(mobile_page.locator(".desktop-nav")).to_be_hidden()

    # Open mobile menu
    mobile_page.click(".hamburger-menu")
    expect(mobile_page.locator(".mobile-menu")).to_be_visible()

def test_touch_interactions(mobile_page: Page):
    """Test touch-specific interactions."""
    mobile_page.goto("https://example.com/gallery")

    # Swipe gesture
    mobile_page.locator(".gallery").swipe_left()

    # Tap (touch event)
    mobile_page.tap(".image-thumbnail")

    # Long press
    mobile_page.locator(".context-menu-trigger").tap(timeout=1000)

def test_responsive_images(mobile_page: Page):
    """Test responsive image loading."""
    mobile_page.goto("https://example.com")

    # Check that mobile-optimized images are loaded
    hero_image = mobile_page.locator(".hero-image")
    src = hero_image.get_attribute("src")

    assert "mobile" in src or "small" in src

def test_viewport_orientation(playwright):
    """Test landscape vs portrait orientation."""
    iphone = playwright.devices['iPhone 12']

    # Portrait mode
    browser = playwright.chromium.launch()
    context = browser.new_context(**iphone)
    page = context.new_page()

    page.goto("https://example.com")
    expect(page.locator(".portrait-layout")).to_be_visible()

    # Switch to landscape
    page.set_viewport_size({"width": 844, "height": 390})
    expect(page.locator(".landscape-layout")).to_be_visible()

    context.close()
    browser.close()
```

### 7. Visual Regression Testing

```python
from playwright.sync_api import Page, expect

def test_homepage_screenshot(page: Page):
    """Capture and compare homepage screenshot."""
    page.goto("https://example.com")

    # Take full page screenshot
    page.screenshot(path="screenshots/homepage.png", full_page=True)

    # Compare with baseline (requires pytest-playwright-visual)
    # page.screenshot().compare("baseline/homepage.png")

def test_element_screenshot(page: Page):
    """Capture specific element screenshot."""
    page.goto("https://example.com")

    # Screenshot specific element
    header = page.locator("header")
    header.screenshot(path="screenshots/header.png")

def test_visual_regression_with_mask(page: Page):
    """Take screenshot with dynamic content masked."""
    page.goto("https://example.com/dashboard")

    # Mask dynamic elements (timestamps, random IDs, etc.)
    page.screenshot(
        path="screenshots/dashboard.png",
        mask=[
            page.locator(".timestamp"),
            page.locator(".session-id")
        ],
        full_page=True
    )

def test_hover_state_screenshot(page: Page):
    """Capture screenshot of hover states."""
    page.goto("https://example.com")

    # Hover over element
    page.hover(".nav-item:first-child")

    # Take screenshot with dropdown visible
    page.screenshot(path="screenshots/nav-hover.png")
```

### 8. Authentication & Session Testing

```python
from playwright.sync_api import Page, expect
import pytest

@pytest.fixture
def authenticated_page(page: Page):
    """Fixture for authenticated user session."""
    # Login
    page.goto("https://example.com/login")
    page.fill("#username", "testuser@example.com")
    page.fill("#password", "SecurePassword123")
    page.click("button[type='submit']")

    # Wait for login to complete
    page.wait_for_url("https://example.com/dashboard")

    yield page

def test_session_persistence(authenticated_page: Page):
    """Test session persists across page navigation."""
    # Navigate to different pages
    authenticated_page.goto("https://example.com/profile")
    expect(authenticated_page.locator(".user-avatar")).to_be_visible()

    authenticated_page.goto("https://example.com/settings")
    expect(authenticated_page.locator(".user-avatar")).to_be_visible()

def test_logout_functionality(authenticated_page: Page):
    """Test logout clears session."""
    authenticated_page.click(".logout-button")

    # Should redirect to login
    expect(authenticated_page).to_have_url("https://example.com/login")

    # Try to access protected page
    authenticated_page.goto("https://example.com/dashboard")

    # Should redirect back to login
    expect(authenticated_page).to_have_url("https://example.com/login")

def test_session_timeout(page: Page):
    """Test session timeout handling."""
    # Set short-lived session cookie
    page.context.add_cookies([{
        "name": "session",
        "value": "test-session-token",
        "domain": "example.com",
        "path": "/",
        "expires": int(time.time()) + 5  # 5 seconds
    }])

    page.goto("https://example.com/dashboard")

    # Wait for session to expire
    time.sleep(6)

    # Try to perform action
    page.click(".refresh-button")

    # Should show session expired message or redirect
    expect(page.locator(".session-expired-message")).to_be_visible()
```

### 9. File Upload and Download Testing

```python
from playwright.sync_api import Page, expect
import os

def test_file_upload(page: Page):
    """Test file upload functionality."""
    page.goto("https://example.com/upload")

    # Upload single file
    page.set_input_files("#file-input", "path/to/test-document.pdf")

    # Verify file name appears
    expect(page.locator(".uploaded-file-name")).to_contain_text("test-document.pdf")

    # Submit upload
    page.click("button:text('Upload')")

    # Verify success
    expect(page.locator(".upload-success")).to_be_visible()

def test_multiple_file_upload(page: Page):
    """Test uploading multiple files."""
    page.goto("https://example.com/upload")

    # Upload multiple files
    page.set_input_files("#file-input", [
        "path/to/file1.jpg",
        "path/to/file2.jpg",
        "path/to/file3.jpg"
    ])

    # Verify all files listed
    file_list = page.locator(".file-list-item")
    expect(file_list).to_have_count(3)

def test_file_download(page: Page):
    """Test file download functionality."""
    page.goto("https://example.com/downloads")

    # Start waiting for download before clicking
    with page.expect_download() as download_info:
        page.click("a:text('Download Report')")

    download = download_info.value

    # Verify download
    assert download.suggested_filename == "report.pdf"

    # Save to specific path
    download.save_as("downloads/report.pdf")

    # Verify file was saved
    assert os.path.exists("downloads/report.pdf")

def test_drag_drop_file_upload(page: Page):
    """Test drag-and-drop file upload."""
    page.goto("https://example.com/upload")

    # Read file as buffer
    with open("path/to/file.jpg", "rb") as f:
        file_content = f.read()

    # Create file input in JS and trigger upload
    page.evaluate("""([content, name]) => {
        const dt = new DataTransfer();
        const file = new File([new Uint8Array(content)], name, {type: 'image/jpeg'});
        dt.items.add(file);
        document.querySelector('.drop-zone').files = dt.files;
        document.querySelector('.drop-zone').dispatchEvent(new Event('change', { bubbles: true }));
    }""", [list(file_content), "test-image.jpg"])

    # Verify file was added
    expect(page.locator(".file-name")).to_contain_text("test-image.jpg")
```

### 10. Page Object Model Pattern

```python
# pages/base_page.py
from playwright.sync_api import Page, expect

class BasePage:
    """Base page class with common functionality."""

    def __init__(self, page: Page):
        self.page = page

    def navigate_to(self, url: str):
        """Navigate to URL."""
        self.page.goto(url)

    def click_element(self, selector: str):
        """Click element with auto-wait."""
        self.page.click(selector)

    def fill_input(self, selector: str, text: str):
        """Fill input field."""
        self.page.fill(selector, text)

    def get_text(self, selector: str) -> str:
        """Get element text content."""
        return self.page.locator(selector).inner_text()

    def is_visible(self, selector: str) -> bool:
        """Check if element is visible."""
        return self.page.locator(selector).is_visible()

# pages/login_page.py
class LoginPage(BasePage):
    """Login page object."""

    # Selectors
    USERNAME_INPUT = "#username"
    PASSWORD_INPUT = "#password"
    SUBMIT_BUTTON = "button[type='submit']"
    ERROR_MESSAGE = ".error-message"
    REMEMBER_ME_CHECKBOX = "#remember-me"

    def login(self, username: str, password: str, remember_me: bool = False):
        """Perform login action."""
        self.fill_input(self.USERNAME_INPUT, username)
        self.fill_input(self.PASSWORD_INPUT, password)

        if remember_me:
            self.page.check(self.REMEMBER_ME_CHECKBOX)

        self.click_element(self.SUBMIT_BUTTON)

    def get_error_message(self) -> str:
        """Get login error message."""
        return self.get_text(self.ERROR_MESSAGE)

    def is_error_visible(self) -> bool:
        """Check if error message is visible."""
        return self.is_visible(self.ERROR_MESSAGE)

# pages/product_page.py
class ProductPage(BasePage):
    """Product page object."""

    ADD_TO_CART_BUTTON = ".add-to-cart"
    PRODUCT_TITLE = ".product-title"
    PRODUCT_PRICE = ".product-price"
    QUANTITY_INPUT = "#quantity"
    SIZE_SELECT = "#size"

    def add_to_cart(self, quantity: int = 1, size: str = None):
        """Add product to cart."""
        if quantity > 1:
            self.fill_input(self.QUANTITY_INPUT, str(quantity))

        if size:
            self.page.select_option(self.SIZE_SELECT, size)

        self.click_element(self.ADD_TO_CART_BUTTON)

    def get_product_title(self) -> str:
        """Get product title."""
        return self.get_text(self.PRODUCT_TITLE)

    def get_price(self) -> str:
        """Get product price."""
        return self.get_text(self.PRODUCT_PRICE)

# Test using Page Objects
def test_login_with_page_object(page: Page):
    """Test login using page object pattern."""
    login_page = LoginPage(page)
    login_page.navigate_to("https://example.com/login")
    login_page.login("testuser@example.com", "SecurePassword123")

    # Verify successful login
    expect(page).to_have_url("https://example.com/dashboard")

def test_add_product_to_cart(page: Page):
    """Test adding product using page object."""
    product_page = ProductPage(page)
    product_page.navigate_to("https://example.com/product/123")

    # Verify product details
    assert "Product Name" in product_page.get_product_title()

    # Add to cart
    product_page.add_to_cart(quantity=2, size="Large")

    # Verify cart updated
    expect(page.locator(".cart-badge")).to_contain_text("2")
```

## Best Practices

### 1. Selector Strategies

```python
# ✅ GOOD: Use test IDs (most reliable)
page.click("[data-testid='submit-button']")

# ✅ GOOD: Use semantic selectors
page.click("button:text('Submit')")
page.click("role=button[name='Submit']")

# ⚠️ OK: Use CSS selectors with meaningful classes
page.click(".submit-button")

# ❌ BAD: Fragile selectors tied to structure
page.click("div > div > button:nth-child(3)")

# ❌ BAD: Position-based selectors
page.click(".button:nth-of-type(5)")
```

### 2. Wait Strategies

```python
# ✅ GOOD: Playwright auto-waits (preferred)
page.click("button")  # Automatically waits for element

# ✅ GOOD: Wait for specific state
page.wait_for_selector(".results", state="visible")

# ✅ GOOD: Wait for network idle
page.wait_for_load_state("networkidle")

# ✅ GOOD: Wait for specific URL
page.wait_for_url("https://example.com/dashboard")

# ⚠️ USE SPARINGLY: Fixed timeouts
import time
time.sleep(2)  # Only when absolutely necessary

# ✅ GOOD: Custom wait conditions
page.wait_for_function("() => document.querySelectorAll('.item').length > 10")
```

### 3. Test Isolation

```python
import pytest
from playwright.sync_api import Page

@pytest.fixture
def isolated_page(context):
    """Create isolated page with clean state."""
    page = context.new_page()
    yield page
    page.close()

def test_with_clean_state(isolated_page: Page):
    """Each test gets fresh page instance."""
    isolated_page.goto("https://example.com")
    # Test runs in isolation

# Clear cookies/storage between tests
@pytest.fixture(autouse=True)
def clear_state(page: Page):
    """Clear cookies and storage before each test."""
    yield
    page.context.clear_cookies()
    page.evaluate("localStorage.clear()")
    page.evaluate("sessionStorage.clear()")
```

### 4. Flaky Test Prevention

```python
# ✅ GOOD: Use Playwright's auto-waiting
page.click("button")  # Waits for actionable state

# ✅ GOOD: Use strict mode to catch multiple elements
page.locator("button").click()  # Fails if multiple buttons

# ✅ GOOD: Wait for specific conditions
expect(page.locator(".result")).to_have_count(5)

# ✅ GOOD: Use soft assertions for multiple checks
expect.soft(page.locator(".title")).to_be_visible()
expect.soft(page.locator(".description")).to_be_visible()
expect.soft(page.locator(".price")).to_contain_text("$")

# ❌ BAD: Race conditions
page.click("button")
result = page.locator(".result").inner_text()  # May not be ready

# ✅ GOOD: Ensure element exists first
page.click("button")
page.wait_for_selector(".result")
result = page.locator(".result").inner_text()
```

### 5. Performance Testing Considerations

```python
from playwright.sync_api import Page
import time

def test_page_load_performance(page: Page):
    """Test page load performance metrics."""
    start_time = time.time()
    page.goto("https://example.com")
    load_time = time.time() - start_time

    # Assert page loads within acceptable time
    assert load_time < 3.0, f"Page load took {load_time}s, expected < 3s"

    # Get performance metrics
    metrics = page.evaluate("""() => {
        const timing = performance.timing;
        return {
            domContentLoaded: timing.domContentLoadedEventEnd - timing.navigationStart,
            loadComplete: timing.loadEventEnd - timing.navigationStart,
            firstPaint: performance.getEntriesByType('paint')[0]?.startTime || 0
        };
    }""")

    assert metrics['domContentLoaded'] < 2000  # 2 seconds
    assert metrics['firstPaint'] < 1000  # 1 second
```

### 6. Accessibility Testing

```python
from playwright.sync_api import Page, expect

def test_keyboard_navigation(page: Page):
    """Test keyboard accessibility."""
    page.goto("https://example.com")

    # Tab through interactive elements
    page.keyboard.press("Tab")
    focused = page.evaluate("document.activeElement.tagName")
    assert focused in ["BUTTON", "A", "INPUT"]

    # Press Enter to activate
    page.keyboard.press("Enter")

def test_aria_labels(page: Page):
    """Test ARIA labels are present."""
    page.goto("https://example.com")

    # Check for aria-label
    button = page.locator("button[aria-label='Submit form']")
    expect(button).to_be_visible()

    # Check for role attributes
    navigation = page.locator("nav[role='navigation']")
    expect(navigation).to_be_visible()

def test_screen_reader_content(page: Page):
    """Test screen reader accessible content."""
    page.goto("https://example.com")

    # Check for alt text on images
    images = page.locator("img")
    count = images.count()
    for i in range(count):
        alt = images.nth(i).get_attribute("alt")
        assert alt is not None, f"Image {i} missing alt text"
```

### 7. Parallel Test Execution

```python
# pytest configuration for parallel tests
# pytest.ini
"""
[pytest]
addopts = -n auto  # Use all CPU cores
"""

# Run tests in parallel
# pytest -n 4  # Use 4 workers

# Mark tests that can't run in parallel
import pytest

@pytest.mark.serial
def test_database_migration(page):
    """This test modifies shared state."""
    pass

# Worker-specific fixtures
@pytest.fixture(scope="session")
def worker_id(worker_id):
    """Get unique worker ID for parallel tests."""
    return worker_id

def test_with_worker_isolation(page, worker_id):
    """Use worker ID for unique test data."""
    username = f"testuser_{worker_id}"
    page.goto("https://example.com/signup")
    page.fill("#username", username)
```

### 8. Debugging Failed Tests

```python
# Enable headed mode for debugging
# pytest --headed

# Slow down execution
# pytest --slowmo=1000  # 1 second delay

# Enable debug mode with Playwright Inspector
# PWDEBUG=1 pytest tests/test_login.py

# Trace on failure
def test_with_trace(page: Page, context):
    """Enable tracing for debugging."""
    context.tracing.start(screenshots=True, snapshots=True)

    try:
        page.goto("https://example.com")
        # Test code here
    finally:
        context.tracing.stop(path="trace.zip")

# Video recording on failure
# Set in pytest.ini or playwright.config.ts
# video: "retain-on-failure"

# Screenshot on failure (automatic with pytest-playwright)
def test_with_auto_screenshot(page: Page):
    """Screenshots automatically saved on failure."""
    page.goto("https://example.com")
    assert False  # Screenshot will be saved
```

## Common Pitfalls

### 1. Race Conditions

```python
# ❌ BAD: Not waiting for dynamic content
page.click("button")
result = page.locator(".result").inner_text()  # May fail

# ✅ GOOD: Wait for element to appear
page.click("button")
page.wait_for_selector(".result")
result = page.locator(".result").inner_text()

# ✅ BETTER: Use Playwright's auto-waiting
page.click("button")
result = page.locator(".result").text_content()  # Auto-waits
```

### 2. Selector Brittleness

```python
# ❌ BAD: Structure-dependent selectors
page.click("div > div > span > button")

# ✅ GOOD: Content-based selectors
page.click("button:text('Submit')")

# ✅ GOOD: Test IDs
page.click("[data-testid='submit-btn']")

# ✅ GOOD: Semantic selectors
page.click("role=button[name='Submit']")
```

### 3. Test Interdependencies

```python
# ❌ BAD: Tests depend on each other
def test_create_user(page):
    # Creates user "testuser"
    pass

def test_login_user(page):
    # Depends on test_create_user running first
    page.fill("#username", "testuser")  # May fail if run alone

# ✅ GOOD: Each test is independent
@pytest.fixture
def created_user(page):
    """Create user for test."""
    # Setup code
    yield user_data
    # Cleanup code

def test_login_user(page, created_user):
    """Test has its own user."""
    page.fill("#username", created_user['username'])
```

### 4. Resource Cleanup

```python
# ✅ GOOD: Always clean up resources
@pytest.fixture
def upload_file(page):
    """Upload file and clean up after test."""
    file_path = "test-upload.txt"

    # Create test file
    with open(file_path, "w") as f:
        f.write("test content")

    yield file_path

    # Cleanup
    if os.path.exists(file_path):
        os.remove(file_path)

# ✅ GOOD: Clean up browser contexts
def test_with_context_cleanup(browser):
    """Properly manage browser contexts."""
    context = browser.new_context()
    page = context.new_page()

    try:
        page.goto("https://example.com")
        # Test code
    finally:
        context.close()
```

### 5. Cookie and Session Management

```python
# ✅ GOOD: Manage session state explicitly
@pytest.fixture
def authenticated_context(browser):
    """Create context with authentication."""
    context = browser.new_context()
    page = context.new_page()

    # Login and save state
    page.goto("https://example.com/login")
    page.fill("#username", "test@example.com")
    page.fill("#password", "password")
    page.click("button[type='submit']")
    page.wait_for_url("https://example.com/dashboard")

    # Save authentication state
    context.storage_state(path="auth.json")

    yield context
    context.close()

def test_with_saved_auth(browser):
    """Use saved authentication state."""
    context = browser.new_context(storage_state="auth.json")
    page = context.new_page()
    page.goto("https://example.com/dashboard")
    # Already authenticated
    context.close()
```

## CI/CD Integration

### GitHub Actions Example

```yaml
name: Playwright Tests

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    timeout-minutes: 60
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v3

    - name: Set up Python
      uses: actions/setup-python@v4
      with:
        python-version: '3.11'

    - name: Install dependencies
      run: |
        python -m pip install --upgrade pip
        pip install -r requirements.txt

    - name: Install Playwright Browsers
      run: playwright install --with-deps

    - name: Run Playwright tests
      run: pytest tests/ --browser chromium --browser firefox

    - name: Upload test results
      if: always()
      uses: actions/upload-artifact@v3
      with:
        name: playwright-report
        path: playwright-report/
        retention-days: 30

    - name: Upload screenshots
      if: failure()
      uses: actions/upload-artifact@v3
      with:
        name: screenshots
        path: test-results/
        retention-days: 7
```

### Docker Example

```dockerfile
FROM mcr.microsoft.com/playwright/python:v1.40.0-jammy

WORKDIR /app

COPY requirements.txt .
RUN pip install -r requirements.txt

COPY . .

CMD ["pytest", "tests/", "--browser", "chromium"]
```

## Quick Reference

### Common Commands

```bash
# Install Playwright
pip install playwright pytest-playwright

# Install browsers
playwright install

# Run tests
pytest tests/

# Run specific browser
pytest --browser chromium
pytest --browser firefox
pytest --browser webkit

# Run in headed mode
pytest --headed

# Run with slowmo
pytest --slowmo=1000

# Generate test code
playwright codegen https://example.com

# Debug mode
PWDEBUG=1 pytest tests/test_login.py

# Parallel execution
pytest -n auto

# Show browser on failure only
pytest --headed --screenshot on --video retain-on-failure
```

### Essential Selectors

```python
# Text content
page.click("text=Submit")
page.click("button:text('Submit')")
page.click("button:has-text('Submit')")

# CSS selectors
page.click("#submit-button")
page.click(".submit-button")
page.click("button[type='submit']")

# XPath
page.click("xpath=//button[@type='submit']")

# Role-based (accessibility)
page.click("role=button[name='Submit']")

# Data attributes
page.click("[data-testid='submit-btn']")

# Chaining
page.locator(".form").locator("button:text('Submit')").click()
```

## Conclusion

Playwright provides a comprehensive, modern framework for web application testing. Its auto-waiting, multi-browser support, and powerful API make it ideal for:

- End-to-end testing
- UI automation
- Visual regression testing
- API testing and mocking
- Mobile responsiveness validation
- Accessibility testing

By following the patterns and best practices in this guide, you can build robust, maintainable test suites that catch bugs early and ensure your web applications work flawlessly across all browsers and devices.

## Additional Resources

- Official Documentation: https://playwright.dev
- Python API: https://playwright.dev/python/docs/intro
- TypeScript API: https://playwright.dev/docs/intro
- Best Practices: https://playwright.dev/docs/best-practices
- Community Discord: https://aka.ms/playwright/discord
