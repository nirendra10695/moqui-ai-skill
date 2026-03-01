---
name: moqui-framework
description: "Use this skill for ANY development task involving the Moqui Framework ecosystem including moqui-framework, moqui-runtime, mantle-udm (Universal Data Model), mantle-usl (Universal Service Library), SimpleScreens, and MarbleERP. Triggers include: creating or editing XML entity definitions, XML service definitions, XML screen files, XML form definitions, writing Groovy scripts for Moqui, creating data load files, designing REST APIs, configuring moqui-conf XML, building e-commerce or ERP features, working with Mantle business logic (orders, invoices, payments, shipments, products, parties, accounting), creating or modifying screen transitions, working with Entity ECA or Service ECA rules, extending entities, creating view-entities, writing XML Actions, configuring ProductStore, building admin screens with SimpleScreens patterns, extending or customizing MarbleERP modules (Customer, Supplier, Internal, Accounting, Project, Configure), and any mention of 'Moqui', 'Mantle', 'MarbleERP', 'Marble ERP', 'mantle-udm', 'mantle-usl', 'SimpleScreens', 'XML Screen', 'XML Form', 'entity-find', 'service-call', 'ExecutionContext', 'qapps', or 'Quasar UI'. Also trigger when user references ERP, order management, inventory, warehouse, fulfillment, procure-to-pay, order-to-cash, or accounting in a Moqui context. Do NOT use for unrelated Java/Groovy projects or other frameworks."
---

# Moqui Framework Development Skill

## Overview

Moqui Framework is an all-in-one enterprise application framework based on Java and Groovy. It provides tools for databases, services, screens/forms, security, localization, caching, and integration. The ecosystem consists of:

- **moqui-framework** — Core framework (Entity Facade, Service Facade, Screen Facade, etc.)
- **moqui-runtime** — Default runtime directory with configuration, components, webroot screens
- **mantle-udm** — Universal Data Model (Party, Product, Order, Shipment, Invoice, Payment, Accounting, etc.)
- **mantle-usl** — Universal Service Library (business logic services for all Mantle UDM entities)
- **SimpleScreens** — Admin/back-office UI screens built on Mantle
- **MarbleERP** — Full ERP application (Customer, Supplier, Internal, Accounting, Project modules)

> **Before writing any code**, read the relevant reference file(s) in this skill's `references/` directory:
> - `references/ENTITIES.md` — Entity definitions, view-entities, extend-entity, field types, relationships, ECA rules
> - `references/SERVICES.md` — Service definitions, XML Actions, service types, SECA rules, REST API
> - `references/SCREENS.md` — XML Screens, XML Forms, transitions, widgets, subscreens, menus
> - `references/MANTLE.md` — Mantle UDM/USL entity packages, key entities, service naming, status flows, business processes
> - `references/MARBLE_ERP.md` — MarbleERP application modules, screen hierarchy, extending patterns, customization recipes

---

## Architecture Quick Reference

### Project Structure

```
moqui/                          # Root (executable WAR)
├── framework/                  # moqui-framework core
│   ├── src/main/               # Java/Groovy source
│   ├── entity/                 # Framework entities (moqui.* packages)
│   ├── service/                # Framework services
│   └── screen/                 # Framework screens (Tools app)
├── runtime/                    # moqui-runtime
│   ├── conf/                   # MoquiProductionConf.xml, etc.
│   ├── component/              # All add-on components live here
│   │   ├── mantle-udm/         # Data model definitions
│   │   │   ├── entity/         # Entity XML files (*.xml)
│   │   │   └── data/           # Seed/demo data files
│   │   ├── mantle-usl/         # Service library
│   │   │   ├── service/        # Service XML files
│   │   │   └── data/           # Service-related data
│   │   ├── SimpleScreens/      # Admin UI screens
│   │   │   ├── screen/         # Screen XML hierarchy
│   │   │   └── template/       # FTL templates
│   │   └── YourComponent/      # Custom component
│   │       ├── entity/         # Entity definitions (.xml, .eecas.xml)
│   │       ├── service/        # Service definitions (.xml, .secas.xml)
│   │       ├── screen/         # Screen hierarchy
│   │       ├── data/           # Data load files
│   │       ├── lib/            # JARs (added to classpath)
│   │       └── script/         # Groovy/other scripts
│   ├── db/                     # Database files (H2 default)
│   └── log/                    # Log files
```

### Component Directory Convention

All files in these directories are auto-discovered:

| Directory | Contents | Auto-loaded | Structure |
|-----------|----------|-------------|-----------|
| `entity/` | Entity definitions (*.xml), Entity ECA rules (*.eecas.xml) | Yes | **Flat** — no subfolders |
| `service/` | Service definitions (*.xml), Service ECA rules (*.secas.xml) | Yes | Hierarchical — subdirs map to package |
| `data/` | Entity XML data files (seed, seed-initial, install, demo) | Via `-load` | Flat |
| `screen/` | XML Screens (referenced by component:// URL) | By reference | Hierarchical (URL path) |
| `script/` | Groovy/XML Action scripts (by component:// URL) | By reference | By reference |
| `lib/` | JAR files added to classpath | Yes | Flat |

> **Important**: Entity files go directly in `entity/` with NO package subfolders. The entity package is defined via the XML `package` attribute, not the directory structure. Service files DO use subdirectories that map to the Java package name (e.g., `service/mycomp/myapp/MyServices.xml` → service name `mycomp.myapp.MyServices.verb#Noun`).

### Execution Context (ec)

The central API object, available in all XML Actions and Groovy scripts:

```groovy
ec.entity      // EntityFacade - CRUD, find, queries
ec.service     // ServiceFacade - call services
ec.user        // UserFacade - current user, login, preferences
ec.message     // MessageFacade - messages and errors
ec.resource    // ResourceFacade - files, scripts, templates
ec.logger      // LoggerFacade - logging
ec.cache       // CacheFacade - caching
ec.transaction // TransactionFacade - JTA transactions
ec.screen      // ScreenFacade - screen rendering
ec.l10n        // L10nFacade - localization, formatting
ec.web         // WebFacade - servlet request/response (web context only)
```

---

## Entity Development (mantle-udm patterns)

### Entity Definition XML

```xml
<entities xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:noNamespaceSchemaLocation="http://moqui.org/xsd/entity-definition-3.xsd">

    <!-- Standard entity -->
    <entity entity-name="MyEntity" package="mycomp.myapp" short-alias="myEntities">
        <field name="myEntityId" type="id" is-pk="true"/>
        <field name="description" type="text-medium"/>
        <field name="statusId" type="id"/>
        <field name="amount" type="currency-amount"/>
        <field name="quantity" type="number-decimal"/>
        <field name="fromDate" type="date-time"/>
        <field name="partyId" type="id"/>
        <relationship type="one" related="mantle.party.Party" short-alias="party"/>
        <relationship type="one" title="MyEntity" related="moqui.basic.StatusItem" short-alias="status"/>
        <relationship type="many" related="mycomp.myapp.MyEntityItem" short-alias="items">
            <key-map field-name="myEntityId"/></relationship>
        <seed-data>
            <moqui.basic.StatusType statusTypeId="MyEntity" description="My Entity"/>
            <moqui.basic.StatusItem statusId="MyActive" statusTypeId="MyEntity" description="Active"/>
            <moqui.basic.StatusItem statusId="MyClosed" statusTypeId="MyEntity" description="Closed"/>
            <moqui.basic.StatusFlowTransition statusFlowId="Default"
                statusId="MyActive" toStatusId="MyClosed" transitionName="Close"/>
        </seed-data>
    </entity>

    <!-- Extend existing entity (add fields/relationships) -->
    <extend-entity entity-name="OrderHeader" package="mantle.order">
        <field name="myCustomField" type="text-medium"/>
        <relationship type="one" related="mycomp.myapp.MyEntity"/>
    </extend-entity>

    <!-- View entity (join/aggregate) -->
    <view-entity entity-name="OrderItemDetail" package="mycomp.myapp">
        <member-entity entity-alias="OI" entity-name="mantle.order.OrderItem"/>
        <member-entity entity-alias="OH" entity-name="mantle.order.OrderHeader" join-from-alias="OI">
            <key-map field-name="orderId"/></member-entity>
        <member-entity entity-alias="PROD" entity-name="mantle.product.Product" join-from-alias="OI" join-optional="true">
            <key-map field-name="productId"/></member-entity>
        <alias entity-alias="OI" name="orderId"/>
        <alias entity-alias="OI" name="orderItemSeqId"/>
        <alias entity-alias="OI" name="unitAmount"/>
        <alias entity-alias="OI" name="quantity"/>
        <alias entity-alias="OH" name="placedDate"/>
        <alias entity-alias="OH" name="statusId"/>
        <alias entity-alias="PROD" name="productName"/>
    </view-entity>
</entities>
```

### Field Types

| Type | Java Type | SQL Default | Notes |
|------|-----------|-------------|-------|
| `id` | String | VARCHAR(40) | PKs, FKs |
| `id-long` | String | VARCHAR(255) | Longer IDs |
| `text-short` | String | VARCHAR(63) | Short text |
| `text-medium` | String | VARCHAR(255) | Medium text |
| `text-long` | String | VARCHAR(4095) | Long text |
| `text-very-long` | String | TEXT/CLOB | Very long text |
| `number-integer` | Long | NUMERIC(20,0) | Integers |
| `number-decimal` | BigDecimal | NUMERIC(26,6) | Decimals |
| `number-float` | Double | DOUBLE | Floating point |
| `currency-amount` | BigDecimal | NUMERIC(22,2) | Money |
| `currency-precise` | BigDecimal | NUMERIC(23,3) | Precise money |
| `date-time` | Timestamp | TIMESTAMP | Date + time |
| `date` | Date | DATE | Date only |
| `time` | Time | TIME | Time only |
| `binary-very-long` | byte[] | BLOB | Binary data |

### Entity ECA Rules (*.eecas.xml)

```xml
<eecas xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:noNamespaceSchemaLocation="http://moqui.org/xsd/eeca-definition-3.xsd">
    <eeca entity="mantle.order.OrderHeader" on-create="true" on-update="true" run-on-error="false">
        <condition><expression>statusId == 'OrderPlaced' &amp;&amp; statusId_old != 'OrderPlaced'</expression></condition>
        <actions>
            <service-call name="mycomp.myapp.MyServices.handle#OrderPlaced"
                in-map="[orderId: orderId]"/>
        </actions>
    </eeca>
</eecas>
```

---

## Service Development (mantle-usl patterns)

### Service Definition XML

```xml
<services xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:noNamespaceSchemaLocation="http://moqui.org/xsd/service-definition-3.xsd">

    <service verb="create" noun="MyEntity" type="inline">
        <description>Create a new MyEntity record</description>
        <in-parameters>
            <parameter name="description" required="true"/>
            <parameter name="statusId" default-value="MyActive"/>
            <parameter name="partyId" required="true"/>
            <auto-parameters entity-name="mycomp.myapp.MyEntity" include="nonpk"/>
        </in-parameters>
        <out-parameters>
            <parameter name="myEntityId"/>
        </out-parameters>
        <actions>
            <service-call name="create#mycomp.myapp.MyEntity" in-map="context" out-map="context"/>
        </actions>
    </service>

    <service verb="update" noun="MyEntity">
        <in-parameters>
            <parameter name="myEntityId" required="true"/>
            <auto-parameters entity-name="mycomp.myapp.MyEntity" include="nonpk"/>
        </in-parameters>
        <actions>
            <service-call name="update#mycomp.myapp.MyEntity" in-map="context"/>
        </actions>
    </service>

    <service verb="get" noun="MyEntityDisplayInfo">
        <in-parameters>
            <parameter name="myEntityId" required="true"/>
        </in-parameters>
        <out-parameters>
            <parameter name="myEntity" type="Map"/>
            <parameter name="itemList" type="List"/>
            <parameter name="partyDetail" type="Map"/>
        </out-parameters>
        <actions>
            <entity-find-one entity-name="mycomp.myapp.MyEntity" value-field="myEntity"/>
            <entity-find entity-name="mycomp.myapp.MyEntityItem" list="itemList">
                <econdition field-name="myEntityId"/>
                <order-by field-name="sequenceNum"/>
            </entity-find>
            <service-call name="mantle.party.PartyServices.get#PartyDetail"
                in-map="[partyId: myEntity.partyId]" out-map="partyDetail"/>
        </actions>
    </service>
</services>
```

### Service Naming Convention (Mantle Pattern)

Services follow verb#Noun naming within a file path that maps to the service name:

```
File: service/mycomp/myapp/MyServices.xml
Service name: mycomp.myapp.MyServices.create#MyEntity
```

Common Mantle service file patterns:
```
mantle.order.OrderServices           # Order CRUD, place, approve, cancel
mantle.order.OrderInfoServices       # Order display/query info
mantle.product.ProductServices       # Product CRUD
mantle.product.AssetServices         # Inventory/asset management
mantle.product.PriceServices         # Price calculation
mantle.shipment.ShipmentServices     # Shipment lifecycle
mantle.account.InvoiceServices       # Invoice creation/processing
mantle.account.PaymentServices       # Payment processing
mantle.party.PartyServices           # Party/person/org management
mantle.party.ContactServices         # Addresses, phone, email
```

### CRUD Shorthand Services

Moqui auto-generates CRUD services for any entity:
```xml
<!-- These don't need XML definitions — they exist automatically -->
<service-call name="create#mantle.order.OrderHeader" in-map="context" out-map="context"/>
<service-call name="update#mantle.order.OrderHeader" in-map="context"/>
<service-call name="delete#mantle.order.OrderHeader" in-map="[orderId: orderId]"/>
<service-call name="store#mantle.order.OrderHeader" in-map="context" out-map="context"/>
```

### XML Actions Reference

```xml
<actions>
    <!-- Variables -->
    <set field="myVar" value="literal"/>
    <set field="myVar" from="someExpression"/>
    <set field="myVar" from="ec.user.nowTimestamp"/>

    <!-- Entity Operations -->
    <entity-find-one entity-name="mantle.order.OrderHeader" value-field="orderHeader"/>
    <entity-find-one entity-name="mantle.order.OrderHeader" value-field="orderHeader" for-update="true"/>

    <entity-find entity-name="mantle.order.OrderItem" list="itemList">
        <econdition field-name="orderId"/>
        <econdition field-name="statusId" value="OrderApproved"/>
        <econdition field-name="quantity" operator="greater" from="0"/>
        <date-filter/>  <!-- Checks fromDate/thruDate against now -->
        <select-field field-name="orderId"/>
        <select-field field-name="unitAmount"/>
        <order-by field-name="-unitAmount"/>  <!-- Prefix - for DESC -->
        <limit count="100" offset="0"/>
    </entity-find>

    <entity-find-count entity-name="mantle.order.OrderItem" count-field="itemCount">
        <econdition field-name="orderId"/>
    </entity-find-count>

    <!-- Service calls -->
    <service-call name="mantle.order.OrderServices.place#Order"
        in-map="[orderId: orderId]" out-map="placeResult"/>
    <service-call name="mantle.order.OrderServices.place#Order"
        in-map="context" out-map="context" transaction="force-new"/>

    <!-- Control flow -->
    <if condition="orderHeader.statusId == 'OrderApproved'">
        <then><!-- actions --></then>
        <else-if condition="orderHeader.statusId == 'OrderPlaced'"><!-- actions --></else-if>
        <else><!-- actions --></else>
    </if>

    <iterate list="itemList" entry="item">
        <set field="item.statusId" value="ItemApproved"/>
        <entity-update value-field="item"/>
    </iterate>

    <!-- Return messages -->
    <return message="Order ${orderId} processed successfully"/>
    <return error="true" message="Cannot process order in status ${orderHeader.statusId}"/>
    <if condition="ec.message.hasError()"><return/></if>

    <!-- Script (Groovy) -->
    <script>
        import java.math.RoundingMode
        def total = items.sum { it.unitAmount * it.quantity }
        total = total.setScale(2, RoundingMode.HALF_UP)
    </script>

    <!-- Logging -->
    <log level="info" message="Processing order ${orderId} with ${itemList.size()} items"/>
</actions>
```

### Service ECA Rules (*.secas.xml)

```xml
<secas xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:noNamespaceSchemaLocation="http://moqui.org/xsd/service-eca-definition-3.xsd">
    <seca service="mantle.order.OrderServices.place#Order" when="post-service" run-on-error="false">
        <condition><expression>orderId</expression></condition>
        <actions>
            <service-call name="mycomp.myapp.MyServices.after#OrderPlaced"
                in-map="[orderId: orderId]"/>
        </actions>
    </seca>
</secas>
```

### REST API

Services are auto-exposed via REST if configured. The standard pattern:

```xml
<!-- In service definition, add authenticate and method attributes -->
<service verb="get" noun="ProductInfo" authenticate="anonymous-view" method="get">
    ...
</service>
```

REST endpoints follow: `POST /rest/s1/mycomp/myapp/MyServices/getProductInfo`

For the store API pattern (popstore/PopRestStore):
```
GET    /rest/s1/store/products/{productId}
POST   /rest/s1/store/cart/add
PUT    /rest/s1/store/cart/updateItem
```

---

## Screen Development (SimpleScreens patterns)

### XML Screen Structure

```xml
<?xml version="1.0" encoding="UTF-8"?>
<screen xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:noNamespaceSchemaLocation="http://moqui.org/xsd/xml-screen-3.xsd"
    default-menu-title="My Entities" default-menu-index="5">

    <!-- Pre-actions: run before all screens in path -->
    <pre-actions><set field="html_title" value="My Entities"/></pre-actions>

    <!-- Transition: handles form submissions and navigation -->
    <transition name="createMyEntity">
        <service-call name="mycomp.myapp.MyServices.create#MyEntity"/>
        <default-response url="."/>
    </transition>
    <transition name="editMyEntity">
        <default-response url="../EditMyEntity"/>
    </transition>

    <!-- Data preparation -->
    <actions>
        <entity-find entity-name="mycomp.myapp.MyEntity" list="myEntityList">
            <search-form-inputs default-order-by="-lastUpdatedStamp"/>
        </entity-find>
    </actions>

    <widgets>
        <!-- Search/filter form -->
        <form-list name="MyEntityList" list="myEntityList" transition="editMyEntity"
                header-dialog="true" select-columns="true" saved-finds="true">
            <row-actions>
                <entity-find-one entity-name="mantle.party.PartyDetail" value-field="partyDetail">
                    <field-map field-name="partyId"/></entity-find-one>
            </row-actions>

            <field name="myEntityId"><header-field show-order-by="true"/>
                <default-field><link url="editMyEntity" text="${myEntityId}" link-type="anchor"/></default-field></field>
            <field name="description"><header-field show-order-by="true"/>
                <default-field><display/></default-field></field>
            <field name="statusId"><header-field title="Status" show-order-by="true">
                    <widget-template-include><set field="efqEnumTypeId" value="MyEntityStatus"/></widget-template-include>
                </header-field>
                <default-field><display-entity entity-name="moqui.basic.StatusItem"/></default-field></field>
            <field name="partyId"><header-field title="Party"/>
                <default-field><display text="${partyDetail?.organizationName ?: ''} ${partyDetail?.firstName ?: ''} ${partyDetail?.lastName ?: ''}"/></default-field></field>
            <field name="submitButton"><header-field title="Find"><submit/></header-field></field>
        </form-list>

        <!-- Create dialog -->
        <container-dialog id="CreateMyEntityDialog" button-text="New My Entity">
            <form-single name="CreateMyEntity" transition="createMyEntity">
                <field name="description"><default-field><text-line size="60"/></default-field></field>
                <field name="partyId"><default-field title="Party">
                    <drop-down allow-empty="true">
                        <dynamic-options transition="getPartyList" server-search="true" min-length="2"/>
                    </drop-down></default-field></field>
                <field name="submitButton"><default-field title="Create"><submit/></default-field></field>
            </form-single>
        </container-dialog>
    </widgets>
</screen>
```

### Screen Hierarchy & Subscreens

Screens form a hierarchy. The directory structure mirrors the URL path:

```
screen/
└── MyApp.xml                    → /myapp
    └── MyApp/
        ├── FindMyEntity.xml     → /myapp/FindMyEntity
        ├── EditMyEntity.xml     → /myapp/EditMyEntity
        └── dashboard.xml        → /myapp/dashboard
```

Mount in MoquiConf.xml (two options):

```xml
<!-- Option A: Standalone app on home App Listing screen (like MarbleERP, System, Tools) -->
<screen-facade>
    <screen location="component://webroot/screen/webroot/apps.xml">
        <subscreens-item name="myapp" location="component://MyComponent/screen/MyApp.xml"
            menu-title="My App" menu-index="20"/>
    </screen>
</screen-facade>

<!-- Option B: Tab inside MarbleERP -->
<screen-facade>
    <screen location="component://MarbleERP/screen/marble.xml">
        <subscreens-item name="MyModule" location="component://MyComponent/screen/marble/MyModule.xml"
            menu-title="My Module" menu-index="15"/>
    </screen>
</screen-facade>
```

> **Note**: Option A requires artifact authorization seed data. See [Artifact Authorization](#artifact-authorization-required-for-app-visibility).

### Common Screen Widgets

```xml
<!-- Display widgets -->
<display/>                                    <!-- Read-only text -->
<display-entity entity-name="..." text="..."/> <!-- Display from related entity -->
<link url="..." text="..." link-type="anchor"/> <!-- Hyperlink -->

<!-- Input widgets -->
<text-line size="30"/>                        <!-- Text input -->
<text-area rows="5" cols="60"/>               <!-- Textarea -->
<text-find size="25" default-operator="begins"/> <!-- Search input -->
<drop-down allow-empty="true">                <!-- Select dropdown -->
    <entity-options text="${description}" key="${enumId}">
        <entity-find entity-name="moqui.basic.Enumeration">
            <econdition field-name="enumTypeId" value="MyType"/>
            <order-by field-name="description"/>
        </entity-find>
    </entity-options>
</drop-down>
<date-time type="date-time"/>                 <!-- Date/time picker -->
<date-find type="date"/>                      <!-- Date range filter -->
<check><entity-options.../></check>           <!-- Checkboxes -->
<radio><entity-options.../></radio>           <!-- Radio buttons -->

<!-- Layout widgets -->
<container>...</container>                    <!-- Div wrapper -->
<container-row><row-col lg="6">...</row-col></container-row>  <!-- Bootstrap grid -->
<container-panel id="...">                    <!-- Panel layout -->
    <panel-header>...</panel-header>
    <panel-left size="3">...</panel-left>
    <panel-center>...</panel-center>
    <panel-right size="3">...</panel-right>
    <panel-footer>...</panel-footer>
</container-panel>
<container-dialog id="..." button-text="..."> <!-- Modal dialog -->
    ...
</container-dialog>

<!-- Conditional rendering -->
<section name="MySection">
    <condition><expression>someCondition</expression></condition>
    <widgets><!-- shown when true --></widgets>
    <fail-widgets><!-- shown when false --></fail-widgets>
</section>
<section-iterate name="..." list="myList" entry="item">
    <widgets>...</widgets>
</section-iterate>
```

---

## Mantle UDM Key Entity Packages

### Core Packages

| Package | Key Entities | Purpose |
|---------|-------------|---------|
| `mantle.party` | Party, Person, Organization, PartyRole, PartyRelationship, RoleType | People and organizations |
| `mantle.party.contact` | ContactMech, PostalAddress, TelecomNumber, PartyContactMech | Contact info |
| `mantle.party.agreement` | Agreement, AgreementItem, AgreementTerm | Agreements |
| `mantle.product` | Product, ProductAssoc, ProductCategory, ProductFeature, ProductPrice, ProductStore | Products and catalog |
| `mantle.product.asset` | Asset, AssetDetail, AssetReservation | Inventory |
| `mantle.product.issuance` | AssetIssuance, AssetReservation | Inventory reservations/issuance |
| `mantle.product.receipt` | AssetReceipt | Inventory receiving |
| `mantle.order` | OrderHeader, OrderPart, OrderItem, OrderItemBilling | Orders |
| `mantle.order.return` | ReturnHeader, ReturnItem | Returns |
| `mantle.shipment` | Shipment, ShipmentItem, ShipmentPackage, ShipmentRouteSegment | Shipping |
| `mantle.account.invoice` | Invoice, InvoiceItem, SettlementTerm | Invoicing |
| `mantle.account.payment` | Payment, PaymentApplication | Payments |
| `mantle.account.method` | PaymentMethod, CreditCard, BankAccount | Payment methods |
| `mantle.account.financial` | FinancialAccount, FinancialAccountTrans | Financial accounts |
| `mantle.ledger` | AcctgTrans, AcctgTransEntry, GlAccount | General ledger |
| `mantle.facility` | Facility, FacilityLocation, FacilityContactMech | Warehouses/locations |
| `mantle.work.effort` | WorkEffort, WorkEffortParty, TimeEntry | Projects, tasks, time |

### Key Status Flows

**Order**: OrderOpen → OrderProposed → OrderPlaced → OrderApproved → OrderSent → OrderCompleted / OrderCancelled / OrderHeld

**Invoice**: InvoiceIncoming/InvoiceInProcess → InvoiceFinalized → InvoiceApproved → InvoiceSent → InvoicePmtRecvd (receivable) or InvoicePaid (payable) / InvoiceCancelled / InvoiceWriteOff

**Shipment**: ShipInput → ShipScheduled → ShipPicked → ShipPacked → ShipShipped → ShipDelivered / ShipCancelled

**Payment**: PmntProposed → PmntPromised → PmntAuthorized → PmntConfirmed → PmntDelivered / PmntDeclined / PmntCancelled / PmntRefunded

**Asset**: AstAvailable, AstOnHold, AstIncoming, AstInStorage

### Order Processing Flow

1. **Create Order** — `OrderServices.create#Order` → OrderOpen status
2. **Add Items** — `OrderServices.create#OrderItem` with productId, quantity, unitAmount
3. **Set Payment** — `OrderServices.add#OrderPartPayment`
4. **Set Shipping** — Update OrderPart with shipmentMethodEnumId, postalContactMechId
5. **Place Order** — `OrderServices.place#Order` → OrderPlaced status (triggers inventory reservation)
6. **Approve** — `OrderServices.approve#Order` → OrderApproved (may be automatic)
7. **Ship** — Create Shipment, pack items, ship → triggers Invoice creation
8. **Invoice** — Auto-created on shipment pack, or manually via `InvoiceServices`
9. **Payment** — Process payment, apply to invoice

---

## Data Loading

### Data File Format

```xml
<entity-facade-xml type="seed">
    <!-- type: seed (always loaded), seed-initial (first load only),
         install (initial setup), demo (demo/test data) -->
    <moqui.basic.Enumeration enumId="MyEnumId" enumTypeId="MyEnumType" description="My Value"/>
    <mantle.party.RoleType roleTypeId="MyRole" description="My Custom Role"/>
    <mycomp.myapp.MyEntity myEntityId="TEST1" description="Test Record" statusId="MyActive"/>
</entity-facade-xml>
```

Load command: `java -jar moqui.war load types=seed,seed-initial,install`

---

## Configuration (moqui-conf.xml)

### Key Configuration Sections

```xml
<moqui-conf xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:noNamespaceSchemaLocation="http://moqui.org/xsd/moqui-conf-3.xsd">

    <!-- Database configuration -->
    <entity-facade>
        <datasource group-name="transactional" database-conf-name="postgres" schema-name="public">
            <inline-jdbc jdbc-uri="jdbc:postgresql://localhost:5432/moqui"
                jdbc-username="moqui" jdbc-password="moqui" pool-maxsize="50"/>
        </datasource>
    </entity-facade>

    <!-- Component loading -->
    <component-list>
        <component name="mantle-udm" location="component/mantle-udm"/>
        <component name="mantle-usl" location="component/mantle-usl"/>
        <component name="SimpleScreens" location="component/SimpleScreens"/>
        <component name="MyComponent" location="component/MyComponent"/>
    </component-list>

    <!-- Screen mounting: Standalone app on home App Listing screen -->
    <screen-facade>
        <screen location="component://webroot/screen/webroot/apps.xml">
            <subscreens-item name="myapp" location="component://MyComponent/screen/MyApp.xml"
                menu-title="My Application" menu-index="10"/>
        </screen>
    </screen-facade>

    <!-- Screen mounting: Tab inside MarbleERP (not on home screen) -->
    <screen-facade>
        <screen location="component://MarbleERP/screen/marble.xml">
            <subscreens-item name="MyModule"
                location="component://MyComponent/screen/marble/MyModule.xml"
                menu-title="My Module" menu-index="15"/>
        </screen>
    </screen-facade>

    <!-- Webapp configuration -->
    <webapp-list>
        <webapp name="webroot">
            <root-screen host=".*" location="component://webroot/screen/webroot.xml"/>
        </webapp>
    </webapp-list>
</moqui-conf>
```

> **Critical**: Mounting under `apps.xml` makes your app appear on the home App Listing screen (alongside MarbleERP, System, Tools). Mounting under `marble.xml` adds it as a tab inside MarbleERP. Choose based on whether your app is an independent application or a MarbleERP module extension.

### Artifact Authorization (Required for App Visibility)

**Every app mounted on the home screen MUST have artifact authorization seed data**, or it will be invisible. The home screen (`AppList.xml`) checks `isPermitted()` for each app — without `ArtifactGroup` and `ArtifactAuthz` records, the framework denies access and hides the app tile.

Create a seed data file in `data/`:

```xml
<!-- YourComponent/data/YourComponentSetupData.xml -->
<?xml version="1.0" encoding="UTF-8"?>
<entity-facade-xml type="seed">
    <!-- Screen access: grants access to root screen and all subscreens (inheritAuthz=Y) -->
    <artifactGroups artifactGroupId="MY_APP" description="My App (via root screen)">
        <artifacts artifactName="component://MyComponent/screen/MyApp.xml"
            artifactTypeEnumId="AT_XML_SCREEN" inheritAuthz="Y"/>
        <!-- Grant access to ADMIN user group (required at minimum) -->
        <authz artifactAuthzId="MY_APP_ADMIN" userGroupId="ADMIN"
            authzTypeEnumId="AUTHZT_ALWAYS" authzActionEnumId="AUTHZA_ALL"/>
    </artifactGroups>

    <!-- REST API access (if your component exposes REST endpoints) -->
    <artifactGroups artifactGroupId="MY_API" description="My App REST API">
        <artifacts artifactTypeEnumId="AT_REST_PATH" inheritAuthz="Y" artifactName="/myapp"/>
        <authz artifactAuthzId="MY_API_ADMIN" userGroupId="ADMIN"
            authzTypeEnumId="AUTHZT_ALWAYS" authzActionEnumId="AUTHZA_ALL"/>
    </artifactGroups>
</entity-facade-xml>
```

After creating this file, reload data: `java -jar moqui.war load types=seed`

**Key rules for artifact authorization:**
- `artifactGroupId` must be globally unique (use a descriptive prefix)
- `artifactAuthzId` must be globally unique
- `inheritAuthz="Y"` on artifacts means all subscreens/sub-paths inherit authorization
- `userGroupId="ADMIN"` grants access to the admin group — add more `<authz>` elements for other user groups
- Data type MUST be `seed` (loaded every time) — not `seed-initial` or `demo`

---

## Common Development Patterns

### 1. Creating a New Component

```bash
mkdir -p runtime/component/MyComponent/{entity,service,screen,data,lib,script}
```

Create `component.xml`:
```xml
<component name="MyComponent" version="1.0.0">
    <depends-on name="mantle-udm"/>
    <depends-on name="mantle-usl"/>
</component>
```

### 2. Extending Mantle Entities

```xml
<!-- Add custom field to OrderHeader -->
<extend-entity entity-name="OrderHeader" package="mantle.order">
    <field name="myCustomField" type="text-medium"/>
    <field name="myEnumId" type="id"/>
    <relationship type="one" title="MyCustomType" related="moqui.basic.Enumeration">
        <key-map field-name="myEnumId"/></relationship>
</extend-entity>
```

### 3. Overriding/Extending Mantle Services (SECA)

Don't modify mantle-usl; use SECA rules to hook into existing services:

```xml
<!-- service/mycomp/MyOrderSeca.secas.xml -->
<secas>
    <seca service="mantle.order.OrderServices.place#Order" when="post-service">
        <actions>
            <service-call name="mycomp.MyOrderServices.after#PlaceOrder" in-map="context"/>
        </actions>
    </seca>
</secas>
```

### 4. ProductStore Configuration

ProductStore is central to e-commerce configuration:
```xml
<mantle.product.store.ProductStore productStoreId="MYSTORE"
    storeName="My Store" organizationPartyId="ORG_MYCOMP"
    defaultCurrencyUomId="USD" defaultLocale="en"
    requireCustomerRole="true" requireInventory="true"/>
```

### 5. Registering a Standalone App on the Home Screen

To create an independent app that appears on the home App Listing screen (alongside MarbleERP, System, Tools), you need **three things**: (1) a root app screen, (2) MoquiConf.xml mounting under `apps.xml`, and (3) artifact authorization seed data.

**Step 1: Root App Screen** (`screen/MyApp.xml`):
```xml
<?xml version="1.0" encoding="UTF-8"?>
<screen xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:noNamespaceSchemaLocation="http://moqui.org/xsd/xml-screen-3.xsd"
    default-menu-title="My Application" menu-image="fa fa-cogs" menu-image-type="icon"
    server-static="vuet,qvt">
    <always-actions>
        <service-call name="mantle.party.PartyServices.setup#UserOrganizationInfo" out-map="context"/>
    </always-actions>
    <subscreens default-item="FindMyEntity" always-use-full-path="true"/>
    <widgets>
        <subscreens-panel id="MyAppPanel" type="popup" title="My Application"/>
    </widgets>
</screen>
```

Key attributes:
- `server-static="vuet,qvt"` — Required for Quasar/Vue UI compatibility
- `subscreens-panel type="popup"` — Creates left-drawer navigation menu (standard for standalone apps)
- `always-use-full-path="true"` — Ensures URL paths include the subscreen name
- `menu-image` — Icon shown on the home screen app tile
- **No `<parameter>` elements** — The root screen must NOT have required parameters (or it won't show on home screen)

**Step 2: MoquiConf.xml** — Mount under `apps.xml`:
```xml
<moqui-conf xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:noNamespaceSchemaLocation="http://moqui.org/xsd/moqui-conf-3.xsd">
    <screen-facade>
        <screen location="component://webroot/screen/webroot/apps.xml">
            <subscreens-item name="myapp" menu-title="My Application" menu-index="20"
                location="component://MyComponent/screen/MyApp.xml"/>
        </screen>
    </screen-facade>
</moqui-conf>
```

**Step 3: Artifact Authorization** — Create `data/MyComponentSetupData.xml` (see [Artifact Authorization](#artifact-authorization-required-for-app-visibility) section above).

Your app will be accessible at `/qapps/myapp` and appear on the home screen at `/qapps`.

### 6. REST API for External Integration

Expose services as REST endpoints:
```xml
<resource name="myapi" require-authentication="true">
    <method type="get"><service name="mycomp.MyServices.get#Data"/></method>
    <method type="post"><service name="mycomp.MyServices.create#Data"/></method>
    <id name="dataId">
        <method type="get"><service name="mycomp.MyServices.get#DataDetail"/></method>
        <method type="put"><service name="mycomp.MyServices.update#Data"/></method>
    </id>
</resource>
```

---

## MarbleERP Application

MarbleERP is the flagship Moqui ERP application, combining goods and services management for B2C and B2B businesses. It is a **screens-only** component — no custom entities or services — relying entirely on Mantle UDM/USL and SimpleScreens.

### Access URL
- **Quasar UI (default)**: `http://localhost:8080/qapps/marble`
- Dynamic Bootstrap UI: `http://localhost:8080/vapps/marble`
- Legacy Standard UI: `http://localhost:8080/apps/marble`

### Module Structure

| Module | URL Path | Business Process |
|--------|----------|-----------------|
| **Customer** | `/marble/Customer` | Order-to-Cash (quotes, sales orders, returns, customer mgmt) |
| **Supplier** | `/marble/Supplier` | Procure-to-Pay (purchase orders, vendor returns, supplier mgmt) |
| **Internal** | `/marble/Internal` | Operations (shipments, inventory, production, facilities) |
| **Accounting** | `/marble/Accounting` | Finance (invoices, payments, GL, reports, fiscal periods) |
| **Project** | `/marble/Project` | PM (projects, tasks, time tracking — from HiveMind) |
| **Configure** | `/marble/Configure` | Setup (ProductStore, facilities, products, categories, GL) |

### Extending MarbleERP

**Always use your own component** — never modify MarbleERP source:

```xml
<!-- YourComponent/MoquiConf.xml — Add a new module tab -->
<screen-facade>
    <screen location="component://MarbleERP/screen/marble.xml">
        <subscreens-item name="MyModule"
            location="component://YourComponent/screen/marble/MyModule.xml"
            menu-title="My Module" menu-index="15"/>
    </screen>
</screen-facade>

<!-- Add screen under existing module -->
<screen-facade>
    <screen location="component://MarbleERP/screen/marble/Customer.xml">
        <subscreens-item name="MyScreen"
            location="component://YourComponent/screen/marble/Customer/MyScreen.xml"
            menu-title="My Screen" menu-index="20"/>
    </screen>
</screen-facade>
```

> For full MarbleERP module details, screen hierarchy, SimpleScreens reuse patterns, and customization recipes, read `references/MARBLE_ERP.md`.

---

## Critical Rules & Best Practices

1. **Never modify framework or Mantle source** — Use extend-entity, SECA/EECA, and custom components
2. **Use service-call for all business logic** — Don't put logic directly in screens
3. **Follow Mantle naming conventions** — verb#Noun for services, package.Entity for entities
4. **Status changes should go through services** — Never directly update statusId fields
5. **Use entity relationships** — Define relationships for proper FK management and navigation
6. **Seed data for types/statuses** — Use `<seed-data>` in entity files or data/ directory
7. **Transaction boundaries** — Services have automatic transaction handling; use `transaction="force-new"` sparingly
8. **Use `in-map="context"`** — Pass context automatically to sub-services instead of building maps manually
9. **Use `auto-parameters`** — Let service definitions auto-discover parameters from entity definitions
10. **Prefer entity-find with search-form-inputs** — For list screens, this auto-wires filter forms to queries
11. **Use `store#Entity`** — Instead of separate create/update logic, `store#` creates if PK missing, updates if exists
12. **Cache wisely** — Set `cache="true"` on entity-find-one for configuration/type entities, not transactional data
13. **Entity files are flat** — Place all entity XML files directly in `entity/` with NO package subfolders; the package is defined via the XML `package` attribute
14. **Service files use subdirectories** — Service XML files go in subdirectories that map to the package name (e.g., `service/mycomp/myapp/MyServices.xml`)
15. **Always create artifact authorization for new apps** — Any app mounted on the home screen needs `ArtifactGroup` + `ArtifactAuthz` seed data or it will be invisible
16. **Root app screens must have no required parameters** — The home screen checks `isPermitted()` and skips screens with required parameters
