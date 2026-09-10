# Eyewear Platform — Domain Model

## 1. Identity & Access

### User
Represents a customer, staff member, or administrator.

Key responsibilities:
- Authentication identity
- Customer profile
- Role assignment
- Account status

Relationships:
- User → Roles
- User → Addresses
- User → Cart
- User → Wishlist
- User → Orders
- User → Prescriptions
- User → Reviews
- User → Appointments
- User → Audit Logs

### Role
Represents an authorization role.

Examples:
- CUSTOMER
- STAFF
- ADMIN
- SUPER_ADMIN

### Permission
Represents an individual authorization capability.

Examples:
- product.read
- product.create
- product.update
- product.delete
- order.read
- order.update
- inventory.manage
- customer.read

### UserRole
Join entity between User and Role.

### RolePermission
Join entity between Role and Permission.

---

## 2. Customer

### Address
Stores customer delivery or billing addresses.

A user may have multiple addresses.

Important fields:
- recipient name
- phone
- address lines
- city
- state
- postal code
- country
- address type
- default status

---

## 3. Product Catalog

### Brand
Represents an eyewear brand.

### Category
Represents a product category.

Categories may support hierarchical relationships.

### Color
Represents a reusable frame color.

### Product
Catalog-level eyewear product.

A Product represents the conceptual frame/product family.

A Product does NOT represent a single sellable SKU.

### ProductVariant
Represents a specific sellable frame variant.

Examples:
- different frame color
- different size
- different SKU

Each sellable variant has its own:
- SKU
- barcode
- measurements
- price
- inventory relationship
- status

### ProductImage
Stores product/variant imagery and metadata.

Images are stored externally; the database stores references and metadata.

---

## 4. Inventory

### Inventory
Represents current inventory for a sellable ProductVariant.

Tracks:
- available quantity
- reserved quantity
- sold quantity
- low-stock threshold

### InventoryTransaction
Immutable inventory history.

Examples:
- STOCK_IN
- STOCK_OUT
- RESERVATION
- RELEASE
- SALE
- ADJUSTMENT
- RETURN

Inventory-critical operations must use database transactions.

---

## 5. Lens Catalog

### LensType
Defines the optical lens category.

Examples:
- Single Vision
- Progressive
- Bifocal
- Reading
- Computer
- Office
- Plano

### LensVariant
Defines the lens material/index/variant.

Examples may include:
- lens material
- refractive index
- thickness class
- processing variant

### LensCoating
Defines an available coating.

Examples:
- Regular
- Blue Cut
- Anti-Glare
- UV
- Scratch Resistant
- Photochromic
- Night Drive
- Premium

### LensCompatibility
Defines which lens combinations are valid for a frame variant.

Compatibility is backend-authoritative.

The compatibility model must support:

ProductVariant
→ LensType
→ LensVariant
→ LensCoating

Invalid combinations must be rejected server-side.

---

## 6. Product Configuration

### ProductConfiguration
Represents a customer's configured eyewear before it becomes a cart item.

Configuration lifecycle:

DRAFT
→ FRAME_SELECTED
→ LENS_SELECTED
→ COATING_SELECTED
→ PRESCRIPTION_PENDING
→ PRESCRIPTION_CONFIRMED
→ READY_FOR_CART

The configuration references:
- Product
- ProductVariant
- LensType
- LensVariant
- LensCoating
- Prescription

It also stores pricing information captured for the configuration.

The backend must recalculate authoritative pricing.

---

## 7. Prescription

### Prescription
Represents an optical prescription.

Supported sources:
- MANUAL
- UPLOAD
- OPTICAL_STORE
- APPOINTMENT

A prescription may contain separate right-eye and left-eye values.

### PrescriptionEye
Stores eye-specific measurements.

Possible measurements:
- SPH
- CYL
- AXIS
- ADD
- PD
- MONOCULAR_PD
- PRISM
- BASE

Prescription validation depends on the selected lens configuration.

### PrescriptionFile
Stores references to uploaded prescription files.

Accepted formats:
- JPG
- PNG
- PDF

Binary files are stored externally.

---

## 8. Shopping

### Cart
Represents a customer's active shopping cart.

### CartItem
Represents one configured eyewear item.

A CartItem references ProductConfiguration.

Two configurations of the same frame must remain separate cart items when their lens/prescription/configuration differs.

### Wishlist
Represents a customer's saved products.

### WishlistItem
Join entity between Wishlist and Product/ProductVariant.

---

## 9. Promotions

### Promotion
Represents a promotional campaign.

### Coupon
Represents a redeemable coupon.

### CouponUsage
Records coupon redemption by users/orders.

Promotional calculations are performed server-side.

---

## 10. Stores

### Store
Represents a physical eyewear store.

Stores contain:
- name
- address
- phone
- opening hours
- location
- status

### StoreService
Represents a service available at a store.

Examples:
- Eye Examination
- Frame Fitting
- Consultation

### AppointmentSlot
Represents an available appointment time.

### Appointment
Represents a customer's appointment.

Appointment booking must prevent double booking server-side.

---

## 11. Orders

### Order
Represents a historical customer purchase.

Orders must preserve historical snapshots so future catalog changes do not modify historical order information.

### OrderItem
Represents an individual purchased item.

Historical information should include:
- product name
- brand
- SKU
- frame details
- lens details
- coating
- configuration
- unit price
- discount
- tax
- quantity

### Payment
Represents payment attempts/results for an order.

Payment processing must use a provider abstraction.

### Refund
Represents a refund associated with a payment/order.

---

## 12. Reviews

### Review
Represents customer feedback about purchased products.

Reviews may reference:
- User
- Product
- OrderItem

---

## 13. Content

### Banner
Represents promotional homepage/banner content.

### HomepageSection
Represents configurable homepage content sections.

Homepage content must be database-driven where appropriate.

---

## 14. Administration

### AuditLog
Records important administrative/system actions.

Examples:
- product created
- product updated
- inventory adjusted
- order status changed
- user role changed

Audit records should preserve actor, action, entity, entity identifier, timestamp, and relevant metadata.

---

## 15. Core Relationship Summary

User
→ Roles
→ Addresses
→ Cart
→ Wishlist
→ Orders
→ Prescriptions
→ Reviews
→ Appointments
→ AuditLogs

Product
→ Brand
→ Category
→ ProductVariants
→ ProductImages

ProductVariant
→ Color
→ Inventory
→ InventoryTransactions
→ LensCompatibility
→ ProductConfigurations

ProductConfiguration
→ LensType
→ LensVariant
→ LensCoating
→ Prescription
→ CartItems

Order
→ OrderItems
→ Payments
→ Refunds

Store
→ StoreServices
→ AppointmentSlots
→ Appointments

Promotion
→ Coupons
→ CouponUsage
